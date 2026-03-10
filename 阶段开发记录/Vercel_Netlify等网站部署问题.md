# 企业项目系统部署 FAQ 文档
> 本文档整理了 Next.js 项目在 Vercel、Netlify 平台部署过程中的常见问题与解决方案，方便后续查阅与排查。

---

## 一、Vercel 部署相关问题

### Q1：Vercel 显示部署成功，但访问 URL 无法打开？
**A**：
1.  **优先排查网络限制**：`vercel.app` 域名在国内网络环境下通常无法直接访问，表现为超时或连接失败。可通过海外网络/代理验证，若能正常访问，则为网络限制问题。
2.  **检查构建配置**：
    -   确认 `Build Command` 拼写正确，如 `pnpm build` 或 `npm run build`，避免使用 `npx build`、`pnx build` 等无效命令。
    -   确认 `Output Directory` 为 Next.js 默认的 `.next`。
    -   检查包管理器一致性，如 `Install Command` 为 `pnpm install`，则 `Build Command` 也应使用 `pnpm build`。
3.  **查看部署日志**：在 Vercel 部署详情页，检查构建和运行时日志，排查依赖安装失败、TypeScript 报错、环境变量缺失等问题。
4.  **解决方案**：
    -   长期方案：绑定自定义域名，绕过 `vercel.app` 限制。
    -   临时方案：使用海外网络或代理访问。

### Q2：Vercel 构建命令 `npx build` 或 `pnx build` 导致部署异常？
**A**：
-   **问题原因**：`npx build` 不是有效的 Next.js 构建命令，`pnx build` 为拼写错误，会导致项目未真正构建，无可用产物。
-   **正确配置**：
    | 包管理器 | 正确 Build Command |
    |----------|--------------------|
    | npm      | `npm run build`    |
    | pnpm     | `pnpm build`       |
    | yarn     | `yarn build`       |
-   **修复步骤**：
    1.  进入 Vercel 项目设置 → Build & Development Settings。
    2.  修正 Build Command 为对应包管理器的正确命令。
    3.  清除缓存并重新部署。

### Q3：如何验证 Vercel 部署是否真正成功？
**A**：
1.  **检查部署状态**：状态为 `Ready Latest` 仅表示流程完成，不代表应用可正常运行。
2.  **查看构建日志**：确认依赖安装、编译、静态页面生成等步骤无错误。
3.  **海外访问测试**：使用海外网络或代理访问 URL，若能正常加载页面，则部署成功。
4.  **本地验证**：本地执行 `pnpm build && pnpm start`，确认项目可正常运行。
5.  **DNS 检测工具验证**：
    -   使用工具：[DNS Checker](https://www.dnschecker.org/)
    -   操作：输入 Vercel 域名，选择 A 记录，点击 Search。
    -   结果解读：海外 DNS 节点（如美国 OpenDNS、Google、Quad9）可成功解析到 IP（如 `64.29.17.195`、`216.198.79.67`），说明域名在全球互联网可访问；国内节点解析失败/超时，确认是国内网络限制。

---

## 二、Netlify 部署与积分管控问题

### Q1：Netlify 积分（构建分钟数）何时恢复？
**A**：
-   积分并非每月 1 日重置，而是按**账单周期**重置，即团队/账号创建日的每月对应日期。
-   月度配额积分不结转，当月未用完将清零；额外购买的积分可结转至下一周期。
-   查看方式：Netlify 控制台 → Team Billing → 查看 Billing period。

### Q2：如何设置 Netlify 手动触发部署，避免自动消耗积分？
**A**：
1.  **完全关闭自动构建（推荐）**：
    -   进入站点设置 → Build & deploy → Continuous deployment。
    -   找到 Build settings → Configure，将 Build status 切换为 `Stopped builds`。
    -   效果：Git 推送不再触发构建，仅手动操作时消耗积分。
2.  **停止自动发布（保留自动构建）**：
    -   在 Deploys 页面点击 `Lock to stop auto publishing`。
    -   效果：Git 推送仍会自动构建（消耗积分），但需手动点击 `Publish deploy` 才上线。

### Q3：手动触发 Netlify 部署的具体步骤？
**A**：
-   **方式一：网页端触发构建（消耗积分）**
    1.  进入站点 Deploys 页面。
    2.  点击右上角 `Trigger deploy`，选择分支（如 `main`）。
    3.  点击 `Deploy site`，完成后手动发布（若关闭了自动发布）。
-   **方式二：本地构建后拖拽上传（零积分消耗）**
    1.  本地执行 `pnpm build`，生成构建产物（如 `.next`）。
    2.  进入 Netlify Deploys 页面，将构建文件夹拖拽至指定区域上传。
    3.  上传完成即部署，不消耗构建分钟数。

### Q4：Netlify 中 `Lock to stop auto publishing` 的作用是什么？
**A**：
-   该选项用于控制“自动发布”，而非“自动构建”。
-   开启后，Git 推送仍会自动构建（消耗积分），但构建产物不会自动上线，需手动审核后发布。
-   适合需要预览后再上线的场景，但无法节省构建积分。
-   **界面位置**：Netlify 站点 Deploys 页面，显示为“Lock to stop auto publishing”按钮。

---

## 三、网络访问与代理问题

### Q1：国内网络无法访问 Vercel/Netlify 网站，如何临时解决？
**A**：
1.  **使用 VPN**：推荐 Proton VPN（免费无限流量）、Windscribe（每月 10GB 免费流量），连接海外节点后访问。
2.  **浏览器代理扩展**：使用 Proxy SwitchyOmega 配置代理规则，仅对受限域名启用代理。
3.  **注意事项**：仅用于访问合法开发平台，遵守公司规定与国家法律法规。

### Q2：公司内网限制访问 Proton VPN 官网，如何使用代理？
**A**：
1.  安装 Proxy SwitchyOmega 浏览器扩展。
2.  新建代理情景模式，配置代理服务器地址、端口和协议。
3.  设置自动切换规则，仅对 `vercel.app`、`netlify.app` 等域名启用代理。
4.  确保内网地址（如 `192.168.*.*`）不经过代理，避免影响内网资源访问。

### Q3：如何使用 DNS Checker 验证域名可访问性？
**A**：
1.  打开 [https://www.dnschecker.org/](https://www.dnschecker.org/)。
2.  输入待检测域名（如 `enterprise-project-system.vercel.app`）。
3.  选择记录类型（如 A 记录），点击 Search。
4.  查看各地区 DNS 节点解析结果：
    -   海外节点（如美国）显示绿色对勾，解析到有效 IP → 域名全球可访问。
    -   国内节点显示超时/失败 → 确认国内网络限制。
5.  **实际案例**：检测 `enterprise-project-system-6utkbjjb3-wastonxhs-projects.vercel.app` 时，美国旧金山、山景城等节点成功解析到 `64.29.17.195` 等 IP，确认部署成功，国内访问失败为网络限制。

---

## 四、核心配置参考

| 平台 | 关键配置项 | 推荐值 |
|------|------------|--------|
| Vercel | Framework Preset | Next.js |
| Vercel | Build Command | `pnpm build`（或 `npm run build`） |
| Vercel | Output Directory | `.next` |
| Vercel | Install Command | `pnpm install`（或 `npm install`） |
| Netlify | Build status | `Stopped builds`（管控积分） |
| Netlify | Build Command | `pnpm build` |
| Netlify | Publish directory | `.next` |
