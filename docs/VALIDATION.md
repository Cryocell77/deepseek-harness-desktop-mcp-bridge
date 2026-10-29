# Validation / 验证记录

Bridge **v0.2.1-rc3**, macOS arm64, **2026-10-29 (Asia/Hong_Kong)**. Earlier results: [v0.2.0](VALIDATION-v0.2.md).

## Compatibility matrix

| Desktop/core | Cordis | Schemastery | Typecheck | Automated suite | Live Desktop                 |
| ------------ | ------ | ----------- | --------- | --------------- | ---------------------------- |
| 0.2.0-rc.2   | 4.0.4  | 3.18.4      | Passed    | 27 passed       | Not tested                   |
| 0.2.1-rc.3   | 4.0.5  | 3.18.5      | Passed    | 27 passed       | Passed, local package        |
| 0.2.1-rc.2   | 4.0.5  | 3.18.5      | Passed    | 27 passed       | Passed, npm installation     |
| 0.2.1-rc.1   | 4.0.5  | 3.18.5      | Passed    | 27 passed       | Development build, see below |

Run `npm run test:compatibility` to install each release's dependencies in a separate temporary directory and run typechecking and the full suite. Select one target with `-- --desktop 0.2.0-rc.2`. The tests use official Cordis, Loader, AgentLoop, Session, Preset and Default Model services with a scripted model provider; no API key is required.

## rc.3 adapter assessment

Official tag `dsh-v0.2.1-rc.3` (`1f81c047fc48fa190a683e469daa699857e9afe8`) was compared with rc.2. Agent, AgentLoop, Session, Tools, Default Model, Preset Registry and App Boot changed only their package versions; their implementation files were unchanged. A fresh dependency tree of the official rc.3 packages passed typechecking and all 27 tests, including execution, scoped tool permissions, cancellation and built-adapter activation. No adapter API changes were required.

The automated assessment uses official runtime services and a scripted model in a simulated Desktop profile. Live application acceptance is recorded below.

## Automated coverage

The bridge's Zod dependency is pinned to **4.6.7**, addressing [GHSA-mrhx-c7xc-m3w5](https://github.com/advisories/GHSA-mrhx-c7xc-m3w5). Production dependency auditing (`npm audit --omit=dev`) passes. The automated matrix was rerun after this dependency update; the live Desktop run below predates it.

| Area               | Checks                                                                                                 |
| ------------------ | ------------------------------------------------------------------------------------------------------ |
| Tasks and sessions | Execution, follow-ups, busy-session rejection, request deduplication and bounded retention             |
| Permissions        | Denied and allowed tools, pre-execute short-circuit resistance, isolation from ordinary Desktop agents |
| Cancellation       | Model cancellation, in-flight tool quiescence, deadlines and terminal results                          |
| Plugin lifecycle   | Built adapter activation in a simulated Desktop profile, socket cleanup and session disposal           |
| Version checks     | Complete supported combinations accepted; mixed, missing, misnamed and unknown packages rejected       |
| Transport          | Token authentication, browser-origin rejection, protocol validation and input/output limits            |
| Workspaces         | Detached Git worktrees, preservation of main-checkout changes and retained child files                 |
| Portable bundle    | Relocation without node_modules, MCP initialization before setup and later reconnection                |

The 0.2.1-rc.2 dependency tree also passed all 10 adapter tests under the installed application's bundled Node in Electron RunAsNode mode. These use simulated contexts rather than a live Desktop Host.

## rc.3 live Desktop run

On **2026-10-29**, the local bridge **0.2.1-rc3** package was installed through the official application-owned plugin installer into macOS Desktop **0.2.1-rc.3**. The MCP client was installed separately from the local tarball in a fresh consumer directory; this was not an npm registry installation.

- Desktop Plugins displayed the rc3 adapter component as **Running**. Connection diagnostics reported `runtime: existing-desktop`, `adapter: dsh-v0.2.1-rc.3` and Cordis `4.0.5`.
- An isolated session completed a real model task with the exact expected response.
- A duplicate follow-up request returned the same task. Cancellation reached `cancelled`, and its terminal result was available.
- Test tasks were forgotten; the session closed and disappeared from the session list. Its workspace was retained.
- Bridge configuration, authentication token and tool allowlist were preserved. The live prompts requested no tools; file edits and permission approval UI remain outside this acceptance run.

An initial startup failed because a previous process had left `bridge.lock`, `bridge.sock` and `ready.json`. The recorded owner process was dead and the socket had no open owner. With the failed bridge component disabled, those three runtime entries were moved to a private backup and the component was re-enabled. No credentials, workspace files or session data were removed.

## npm installation and Desktop run (bridge 0.2.1-rc2)

Package: [`dsh-desktop-mcp-bridge@0.2.1-rc2`](https://www.npmjs.com/package/dsh-desktop-mcp-bridge/v/0.2.1-rc2), published under `next`. Environment: official macOS arm64 DSH Desktop **0.2.1-rc.2**, Cordis **4.0.5**, Schemastery **3.18.5**.

Both the MCP client and Desktop adapter were installed by package name from the public npm registry. The Desktop-owned installer pinned the exact version in its profile; bridge configuration, token and tool permissions were preserved. No additional model credentials were configured.

| Check                | Result                                                                                                             |
| -------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Published archive    | Registry metadata, SHA-512 integrity and SHA-1 matched; public download was byte-identical to the reviewed archive |
| Standalone install   | Fresh anonymous npm cache; CLI available; optional Desktop peers absent; MCP handshake exposed 10 tools            |
| Configuration        | Inspection wrote no state; explicit initialization and reconnect succeeded; missing Desktop returned an error      |
| Desktop connection   | `runtime: existing-desktop`, bridge `0.2.1-rc2`, adapter `dsh-v0.2.1-rc.2`                                         |
| Model task           | New isolated session returned the exact verification response through the Desktop provider                         |
| Follow-up and cancel | Duplicate request returned the same task; cancellation reached `cancelled` with a terminal result                  |
| Session cleanup      | Verification tasks forgotten; session closed and absent from the list; isolated workspace retained                 |
| npx installation     | Fresh anonymous cache; pinned package completed `setup` and `doctor`; existing configuration and token unchanged   |

The live tasks requested no tools; allowed file edits and permission approval UI routing remain untested. Automated tool permission coverage is listed above.

Reproduce the registry check without model calls with `npm run verify:npm`. To include a live task and cancellation against a configured, running Desktop:

```sh
npm run verify:npm -- --desktop-config /absolute/path/to/config.json --live-task
```

This submits model requests using the Desktop provider. The script closes its verification session and preserves its workspace.

## Earlier Desktop run

Environment: signed macOS DSH Desktop **0.2.1-rc.1**, pre-release bridge build, `allowedTools: []`.

| Check          | Result                                                                                        |
| -------------- | --------------------------------------------------------------------------------------------- |
| Installation   | Application-owned plugin installer completed its lockfile check; adapter activated in Desktop |
| Connection     | `runtime: existing-desktop`, adapter `dsh-v0.2.1-rc.1`                                        |
| Model response | Isolated session returned the expected response; durable events recorded a completed turn     |
| Cancellation   | running → cancelling → cancelled; durable `turn/end` reason was `aborted`                     |
| Tool denial    | Native `read` was denied; the session's `tool/result` recorded `isError: true`                |
| Cleanup        | Both test sessions closed; workspaces retained                                                |

## Remaining checks

- Live end-to-end testing on Desktop 0.2.0-rc.2.
- Allowed file edits and tool approval UI routing.
- Rendered plugin metadata and icon.
- Linux Desktop, Windows transport and other DSH versions.
- Long-running external side effects and production reliability.

中文：四个精确版本组合均通过类型检查和自动化测试。从公共 npm 安装的桥接与适配器已在 rc.2 实机通过连接、真实模型任务、重复请求去重、取消和会话关闭验证；独立 npx 安装也通过 setup/doctor 检查。原有配置、权限与 token 保持不变。文件编辑与权限审批界面仍待实机验证；rc.1 的较早记录见上表。

rc.3 实机验证：本地 0.2.1-rc3 安装包已在官方 Desktop 0.2.1-rc.3 通过连接、真实模型任务、重复请求去重、取消及会话关闭验证。首次启动遇到旧进程遗留锁；确认原进程不存在且 socket 无占用后，保留备份并重载桥接组件，连接恢复正常。
