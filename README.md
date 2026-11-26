# SAD-homework05-202611 ｜ 实践课2 & 实践课3

湖南师范大学软件工程 2026 年「开源软件项目设计与分析」课程作业。

- 姓名：陈智杰　GitHub：[@czj-daniel-knox](https://github.com/czj-daniel-knox)
- 本仓库：<https://github.com/czj-daniel-knox/SAD-homework05-202611>
- 小组组织仓库（本次分析与编译的源码）：<https://github.com/SAD-OpenStudy/pi-hunnu-2026>
- 实际 clone 地址：<https://github.com/SAD-OpenStudy/pi-hunnu-2026.git>
- 源码版本：`c3cc91632d0a1950fb122605b190233f2aa0006e`（`main`）

## 报告

| 实践 | 主题 | 报告 | 提交 Issue |
|---|---|---|---|
| 实践课2 | 组织建仓与 Pi Agent 源码分析 | [`practice-02/report.md`](practice-02/report.md) | [#250](https://github.com/hunnuOpenSourceSoftware2026/assignments/issues/250) `实践课2：组织建仓与源码分析 chen zhijie` |
| 实践课3 | Pi Agent 源码获取、依赖安装与本地编译 | [`practice-03/report.md`](practice-03/report.md) | [#251](https://github.com/hunnuOpenSourceSoftware2026/assignments/issues/251) `实践课3：Pi Agent源码安装与编译 chen zhijie` |
| 实践课4 | Pi Agent 的模型调用与运行流程 | [`practice-04/practice-04.md`](practice-04/practice-04.md) | [#283](https://github.com/hunnuOpenSourceSoftware2026/assignments/issues/283) `实践课4：Pi Agent模型调用与运行流程 chen zhijie` |
| 实践课5 | Pi Agent 工具调用与会话恢复 | [`practice-05/practive-05.md`](practice-05/practive-05.md) | [#348](https://github.com/hunnuOpenSourceSoftware2026/assignments/issues/348) `实践课5：Pi Agent工具调用与会话恢复 chen zhijie` |

## 实践课2 要点

- 小组 fork 已就绪：`SAD-OpenStudy/pi-hunnu-2026`，父仓库 `hunnuOpenSourceSoftware2026/pi-hunnu-2026`；本人 clone 到全英文路径并核对远端、分支与提交号。
- 11 个包的**功能 / 构建行为 / 内部依赖 / 依据文件**逐行分析（`telemetry`、`schema`、`chord`、`tui`、`codemode`、`mcp`、`ai`、`durable`、`env`、`agent`、`coding-agent`）。
- 额外发现：`packages/evals`（第 12 个 workspace）与 `apps/p2`、`apps/p2-mac`、`apps/p3` 不在根构建序列中，原因写在报告 §2.3。
- 模块层级图：`practice-02/images/dependency-graph.png`，可编辑图源 `practice-02/diagrams/dependency-graph.mmd`（Mermaid）。
- 三个问题（构建配置来源、构建顺序与依赖关系、三模块依赖路径）分别作答，并用“未构建上游时单独编译 `agent` 报 TS2307”的实验证明构建顺序的因果。

## 实践课3 要点

- 环境：Node.js `v24.18.0`、npm `12.1.0`、git `2.55.0.windows.2`。
- 依赖：`npm install --ignore-scripts` → 326 packages、0 vulnerabilities、退出码 0（两条弃用警告不等于失败）。
- 构建：`npm run build` → 11 个包、53 秒、退出码 0；`ai` 联网生成模型目录，`coding-agent` 用 esbuild 打包出 `84 files / 9.4 MiB`。
- 产物：`packages/coding-agent/dist/bundle/cli.js` 存在，`--version` 为 `1.5.2`；用 `.\pi-test.bat` 运行源码版并完成一次真实模型问答。
- 困难记录：PowerShell 5.1 把 npm stderr 当错误、Windows Terminal 大量输出后停刷、`fd`/`ripgrep` 因证书校验下载失败、全局 `pi update` 提示的处理。

## 实践课4 要点

- 在 `packages/agent/src/agent-loop.ts` 加两行 stderr 日志：模型调用前（现第 413 行）、工具结果写入上下文后（现第 285 行），重新 `npm run build` 并启动 `dist/cli.js`。
- 会话 A（直接问答）：0 次工具调用、1 条“即将请求模型”日志、1 轮模型调用。
- 会话 B（`read` 读 `practice4.txt`）：1 次 `read` 工具，日志顺序为“即将请求模型 → 工具结果已加入上下文 → 即将请求模型”，2 轮模型调用，读到的代号 `P4-20261123-CZJ-KX7Q`。
- 结论：文件是本地 `read` 工具读的，模型在第二轮才拿到 `toolResult`；下一轮请求多带一条工具结果消息。附运行流程简图。

## 实践课5 要点

- `handoff.txt` 用随机标识 `quartz-68707`；A 阶段一次 `read` 工具调用、两轮模型调用，原样给出待办与启动命令。
- `-c` 恢复会话后提问：0 次工具、1 次模型调用，回复含完整命令（A 的 4 条历史消息 + 新提问一起发给模型）。
- `/new` 新建会话、相同提问：0 次工具、1 次模型调用，回复说明没有交接单内容。
- 图1 为五条生命线的代码级时序图，图2 为恢复/新建两条分支的对照图，均附 Mermaid 图源；read 工具实际走流式读取（`selectLinesFromStream` + `fs` 分块）。

## 仓库结构

```text
SAD-homework05-202611/
├─ README.md                       本文件
├─ practice-02/
│  ├─ report.md                    实践课2 报告
│  ├─ diagrams/dependency-graph.mmd  依赖关系图源（Mermaid）
│  └─ images/                      S1~S3 截图、依赖图、tsc 顺序实验
└─ practice-03/
   ├─ report.md                    实践课3 报告
   └─ images/                      S1~S7 截图
```

```text
practice-04/
├─ practice-04.md                  实践课4 报告
├─ diagrams/flow.mmd               运行流程简图源（Mermaid）
└─ images/                        S1~S6 截图 + 简图
```

```text
practice-05/
├─ practive-05.md                  实践课5 报告（文件名沿用作业要求写法）
├─ diagrams/fig1-sequence.mmd      图1 时序图源（Mermaid）
├─ diagrams/fig2-sessions.mmd      图2 对照图源（Mermaid）
└─ images/                        A/B 截图 + 两张图
```

## 证据与版本

| 项 | 值 |
|---|---|
| 源码提交 | `c3cc91632d0a1950fb122605b190233f2aa0006e` |
| 源码 / 打包 / 全局 pi | 1.5.2 / 1.5.2 / 1.5.0 |
| 安装 | 326 packages，356 audited，0 vulnerabilities，EXIT=0 |
| 构建 | 11 个包，53 s，EXIT=0，bundle 84 files / 9.4 MiB |
