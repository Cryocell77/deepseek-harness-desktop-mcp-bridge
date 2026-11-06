# Validation / 验证记录

Bridge **v0.2.2-rc1**, macOS arm64, **2026-11-07 (Asia/Hong_Kong)**. Earlier results: [v0.2.0](VALIDATION-v0.2.md).

## Compatibility matrix

| Desktop/core | Cordis | Schemastery | Typecheck | Automated suite | Live Desktop                 |
| ------------ | ------ | ----------- | --------- | --------------- | ---------------------------- |
| 0.2.2-rc.1   | 4.1.0  | 3.19.0      | Passed    | 27 passed       | Passed, local package        |
| 0.2.0-rc.2   | 4.0.4  | 3.18.4      | Passed    | 27 passed       | Not tested                   |
| 0.2.1-rc.4   | 4.0.5  | 3.18.5      | Passed    | 27 passed       | —                            |
| 0.2.1-rc.5   | 4.0.5  | 3.18.5      | Passed    | 27 passed       | Passed, local package        |
| 0.2.1-rc.3   | 4.0.5  | 3.18.5      | Passed    | 27 passed       | Passed, local package        |
| 0.2.1-rc.2   | 4.0.5  | 3.18.5      | Passed    | 27 passed       | Passed, npm installation     |
| 0.2.1-rc.1   | 4.0.5  | 3.18.5      | Passed    | 27 passed       | Development build, see below |

Run `npm run test:compatibility` to install each release's dependencies in a separate temporary directory and run typechecking and the full suite. Select one target with `-- --desktop 0.2.0-rc.2`. The tests use official Cordis, Loader, AgentLoop, Session, Preset and Default Model services with a scripted model provider; no API key is required.

## 0.2.2-rc.1 assessment

Official tag `dsh-v0.2.2-rc.1` resolves to `66491b05e3f05ad35bcdd4f6cd7ed26041d9e3db`. The installed official macOS application and its guarded core manifests report **0.2.2-rc.1**, Cordis **4.1.0** and Schemastery **3.19.0**. See the [source audit](UPSTREAM-AUDIT.md) for the Host, permissions and executor changes.

Tests use the new official native tool executor and an explicit tool-owned output contract, with a scripted model provider. They cover allowed/denied tools, monotonic guard enforcement after a short-circuit policy, separation from ordinary Desktop agents, real AgentLoop execution, in-flight model/tool cancellation, deadlines and built-plugin activation/unload. All seven exact dependency trees passed typechecking and 27 tests each under Node 24.21.0. The fresh local package installation, portable bundle verification and production dependency audit also passed.

## 0.2.2-rc.1 live Desktop run

On **2026-11-07 (Asia/Hong_Kong)**, after the user saved work and fully quit Desktop, bridge **0.2.2-rc1** was installed from the local archive through the official Desktop-owned installer. Configuration, token, tool allowlist and plugin composition were unchanged. Desktop was reopened and reported `runtime: existing-desktop`, `desktopVersion: 0.2.2-rc.1`, `adapter: dsh-v0.2.2-rc.1` and Cordis `4.1.0`.

A fresh anonymous installation of the local archive exposed all 10 MCP tools and connected to that Host. An isolated session completed a real model task with the exact requested response. Duplicate follow-up submission returned the same task; cancellation settled as `cancelled` with a terminal result. Verification tasks were forgotten, and the session closed and disappeared from the list. Its workspace was retained.

Live prompts requested no tools; permission enforcement and in-flight tool cancellation were verified by the automated suite. Allowed file edits and approval UI routing remain outside this live run. No separate model credentials or DSH CLI/Web runtime were used for task execution.

中文：本地 **0.2.2-rc1** 适配器已在官方 Desktop **0.2.2-rc.1** 通过连接、真实模型任务、后续请求去重、取消、结果读取与会话关闭验证。配置、token 和工具白名单保持不变；实机任务未调用工具，工具权限及工具取消由自动化测试覆盖。

## rc.4 and rc.5 assessment

Official source tags: `dsh-v0.2.1-rc.4` (`504a01a9578abc2c6f1fb896b1dc322c9fe24f66`) and `dsh-v0.2.1-rc.5` (`b830ec950faba0a46dda564be3e2d854afed94f1`). Compared with rc.3, Agent, AgentLoop, Session, Tools, LLM, Default Model, Preset Registry and App Boot changed only their package versions. Adapter API calls and permission guards require no implementation changes.

The installed official macOS Desktop reports **0.2.1-rc.5**, with all inspected core packages matching rc.5, Cordis **4.0.5** and Schemastery **3.18.5**. An isolated rc.5 dependency tree passed typechecking and all 27 automated tests.

## rc.5 live Desktop run

On **2026-11-02 (Asia/Hong_Kong)**, bridge **0.2.1-rc5** was installed from the local package through the official Desktop-owned plugin installer while Desktop was quit. Configuration, authentication token, tool allowlist and plugin composition were preserved. No stale bridge lock or socket was present before Desktop reopened.

A fresh consumer installation of the local package passed CLI/MCP initialization checks and connected to `runtime: existing-desktop`, `desktopVersion: 0.2.1-rc.5`, `adapter: dsh-v0.2.1-rc.5` and Cordis `4.0.5`. An isolated session completed a real model request with the expected response. Duplicate follow-up submission returned the same task; cancellation reached `cancelled`, with a terminal result available. Verification tasks were forgotten and the session closed, retaining its workspace.

The live prompts requested no tools. Automated permission enforcement passed across all six supported combinations; allowed file edits and permission approval UI remain untested on rc.5.

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

## npm publication check (bridge 0.2.1-rc3)

On **2026-10-30 (Asia/Hong_Kong)**, the published npm package **0.2.1-rc3** was verified against release commit `8d5eda4a56dfa67409c75e2f7f8edfe17886c633`. The `next` tag points to rc3; `latest` remains rc2.

- Registry metadata, SHA-512 integrity and SHA-1 matched the release archive; the public download was byte-identical.
- A fresh anonymous installation exposed the CLI and all 10 MCP tools. Explicit initialization and reconnect passed; a missing Desktop returned `DESKTOP_UNAVAILABLE`.
- The registry-installed MCP client connected to the existing Desktop **0.2.1-rc.3** adapter. This publication check made no model calls and did not reinstall the Desktop adapter; the earlier live task and cancellation run is recorded above.

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

中文：六个精确版本组合均通过类型检查和自动化测试。从公共 npm 安装的桥接与适配器已在 rc.2 实机通过连接、真实模型任务、重复请求去重、取消和会话关闭验证；独立 npx 安装也通过 setup/doctor 检查。原有配置、权限与 token 保持不变。文件编辑与权限审批界面仍待实机验证；rc.1 的较早记录见上表。

rc.3 实机验证：本地 0.2.1-rc3 安装包已在官方 Desktop 0.2.1-rc.3 通过连接、真实模型任务、重复请求去重、取消及会话关闭验证。首次启动遇到旧进程遗留锁；确认原进程不存在且 socket 无占用后，保留备份并重载桥接组件，连接恢复正常。

rc.5 实机验证：已在官方 Desktop 0.2.1-rc.5 通过真实任务、请求去重、结果读取、取消及会话关闭检查。六个版本组合均通过类型检查和各 27 项自动化测试。本次实机检查使用本地安装包。
