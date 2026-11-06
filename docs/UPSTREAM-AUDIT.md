# Official upstream audit / 官方源码核实

Bridge **v0.2.2-rc1**, reviewed **2026-11-07 (Asia/Hong_Kong)**. [Earlier audit](UPSTREAM-AUDIT-v0.2.md).

The official [DSH 0.2.2-rc.1 release](https://github.com/deepseek-ai/deepseek-harness/releases/tag/dsh-v0.2.2-rc.1) points to `66491b05e3f05ad35bcdd4f6cd7ed26041d9e3db`. The installed macOS application reports the same version. Targeted ASAR manifests confirm all six guarded DSH core packages at **0.2.2-rc.1**, Cordis **4.1.0** and Schemastery **3.19.0**. The upstream [MIT license](https://github.com/deepseek-ai/deepseek-harness/blob/66491b05e3f05ad35bcdd4f6cd7ed26041d9e3db/LICENSE) is unchanged.

## Desktop Host and authentication

The [official Desktop reference](https://github.com/deepseek-ai/deepseek-harness/blob/66491b05e3f05ad35bcdd4f6cd7ed26041d9e3db/apps/desktop/README.md) describes an Electron-owned Host, the Desktop profile, authenticated HTTP/WebSocket transport and the official plugin installation surface. It does not document an external MCP attach interface that reuses the running Desktop. This bridge continues to use a Cordis plugin inside that Host and its own private authenticated Unix socket. It neither starts an independent DSH CLI/Web runtime nor reads Desktop authentication material.

## Adapter API review

- **Metadata:** the [profile metadata service](https://github.com/deepseek-ai/deepseek-harness/blob/66491b05e3f05ad35bcdd4f6cd7ed26041d9e3db/packages/boot/app-boot/src/profile-resolution/service.ts) retains `pluginPackages.packageOf`. The adapter requires one complete tested tuple; unresolved metadata and mixed core versions fail closed.
- **Permissions:** [ToolsService](https://github.com/deepseek-ai/deepseek-harness/blob/66491b05e3f05ad35bcdd4f6cd7ed26041d9e3db/packages/core/tools/src/index.ts) retains scoped `tools.guard`. The [shared execution pipeline](https://github.com/deepseek-ai/deepseek-harness/blob/66491b05e3f05ad35bcdd4f6cd7ed26041d9e3db/packages/core/tools/src/pipeline.ts) checks guards after extensible pre-execute policy. A listener returning `allow` cannot override a guard denial. The bridge installs the guard before mounting each agent preset, retaining its additional pre-execute/execute checks.
- **Tool execution:** native and PTC execution are now separate plugins. The adapter preserves the Host/preset's executor selection. The new-version test environment mounts the official native executor; older test environments retain their original integrated executor. The removed upstream `defineContentToolFixture` helper is replaced by an explicit test tool output contract. Production permission policy is unchanged; `run_code` remains unsupported.
- **Models and sessions:** `agentDefaultModel.currentSelection()`, `agentPresets.resolve/mount`, `agents.create`, `followup`, `whenIdle`, cancellation and owned-handle disposal remain available. The new model-selection timing does not require the adapter to intercept routing. Results still come from durable `assistant/message` and `turn/end` events via `snapshotEvents`.
- **Dependencies:** development pins use the new official release. Each older matrix entry resolves its own exact Cordis dependency tree, without forced or legacy peer resolution. Node **>=24.9** matches the new upstream minimum.

## Supported exact combinations

| DSH core   | Cordis | Schemastery |
| ---------- | ------ | ----------- |
| 0.2.0-rc.2 | 4.0.4  | 3.18.4      |
| 0.2.1-rc.1 | 4.0.5  | 3.18.5      |
| 0.2.1-rc.2 | 4.0.5  | 3.18.5      |
| 0.2.1-rc.3 | 4.0.5  | 3.18.5      |
| 0.2.1-rc.4 | 4.0.5  | 3.18.5      |
| 0.2.1-rc.5 | 4.0.5  | 3.18.5      |
| 0.2.2-rc.1 | 4.1.0  | 3.19.0      |

Tests and live Desktop acceptance are recorded separately in [VALIDATION.md](VALIDATION.md).

中文：新版仍使用现有 Desktop Host 和官方插件服务。工具执行器拆分后，测试通过独立官方 Native 插件验证权限与取消流程；适配器不改变用户的执行器、模型配置或工具白名单。
