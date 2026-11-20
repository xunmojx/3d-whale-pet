# 给 AI 的协作规则

这个文件是给接手这个项目的 AI 看的，不是给用户看的。用户看不懂代码，也不想听排查过程。

## 最重要的一条：改完不要直接发版

**流程是：改本地 → 用户实测 → 他说没问题 → 才打包 + 推两个仓库。**

以前是改完就发版，结果版本号从 0.3 一路涨到 0.6，中间一堆没验证就被覆盖掉的实验版本。现在反过来：本地随便改、反复改，发出去的每个版本都是他确认过的。

- 改完只动 `~/.dsh/plugins/3d-whale-pet` 里的文件
- `README.md` 和 `pet/renderer/help.html` 照旧随手同步（那是本地文件，不算发布）
- **不要**升版本号、**不要** `pnpm pack`、**不要**跑发布脚本
- 告诉用户改了什么、怎么试，然后就等着

## 用户实测的两种生效方式

- 改 `pet/` 里的东西（菜单、动画、气泡、窗口）→ **重启桌宠**就生效
- 改 `lib/` 里的东西（插件加载逻辑）→ **重启 DSH**

## 发版的时候（他说可以了才做）

1. `package.json` 版本号 +1
2. `pnpm pack` → 重命名为 `3d-whale-pet.tgz`（固定名）
3. 推 Gitee（主发布源，国内唯一稳定通道）
4. **同时**推 GitHub（市场收录 + 版本号对齐）
5. 两边版本号必须一致，否则市场会显示旧版本

脚本都在 `C:\Users\hp\Documents\deepseek-harness\default-workspace\`：
- `publish-gitee.cjs`（需要 `GITEE_TOKEN`）
- `publish-github.cjs`（需要 `GITHUB_TOKEN`，默认不提新 PR，换条目才用 `SYNC_PR=1`）

## 用户网络环境（很关键，别搞错）

```
gitee.com                ✓ 稳
api.github.com           ✓ 稳（推送用它）
github.com               ✗ 时通时断
raw.githubusercontent    ✗ 完全不通
www.npmjs.com            ✗ 403（Cloudflare）
```

所以：**推送**走 GitHub API 没问题；**市场页面**在他那边可能加载不出来；npm 发布这条路彻底不通。

## 沟通方式

- 只要结果、结论、下一步怎么做；不要展开代码层面的排查过程
- 口语，别端着，别分点报告腔
- 有明确判断，别模棱两可
- 他看不懂代码，解释要短

## 踩过的坑（别再踩）

- `display.workArea` **已经是 DIP**，不要再除 `scaleFactor`
- 透明无边框窗口构造时给的 x/y 会被系统忽略，要等 `did-finish-load` 之后再 `setBounds`
- 帮助窗口必须 `screen-saver` 层级，否则被桌宠压住（表现为"点不开"）
- 两个桥对象：`petWindow`（窗口/游戏/菜单/取名/清数据）和 `petBridge`（情绪/泳姿/回复/皮肤）。**接错是静默失效**，不报错——help 页里要把两个都合并
- `join is not defined` 犯过两次：`pet/main.js` 里只有 `const path = require('node:path')`，没有解构 `join`
- pnpm 10+ 会拦 electron 的构建脚本，装完必须补跑 `node node_modules/electron/install.js`
- 删 172MB 的 runtime 要交给独立 node 子进程（detached 的 `cmd + timeout` 不行，没控制台 timeout 直接失败）
