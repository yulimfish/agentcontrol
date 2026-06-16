# AgentControl 总仓 Agent 指南

这个根仓库协调三个可以单独发布的项目：

- `agentcontrol-fabric/`：Fabric 客户端模组，在 Minecraft 内运行并暴露本地状态/动作端点。
- `agentcontrol-mcp/`：MCP 服务，向 AI 工具暴露能力并调用 Fabric 模组端点。
- `agentcontrol-docs/`：文档与跨项目架构说明。

## 行为准则

**所有面向用户的文档、README、说明性文字、界面提示和注释必须使用中文撰写。** 仓库名、目录名、代码标识符、命令、工具名、环境变量和 URL 保持原样不翻译。MIT 许可证文本保持英文原文。

## 操作前置条件（所有 Agent 必须遵守）

在任何动作操作（移动、攻击、使用、破坏/放置等）之前，必须先检查当前状态。如果检测到 `screen` 字段不为 `null`（表示有界面打开），必须**先执行 `close_screen` 关闭界面**，然后再执行目标动作。如果 `close_screen` 后界面仍然存在（如 Minecraft 的 ESC 暂停菜单），则必须**再执行 `release_mouse` 释放鼠标并关闭界面**，然后再执行动作。

**原因**：Minecraft 在界面打开时会拦截所有移动和交互输入，导致动作无效。

**正确流程**：
```text
1. 读取状态 -> 检查 screen 是否为 null
2. 如果 screen 不为 null -> 执行 close_screen
3. 如果界面仍未关闭 -> 执行 release_mouse
4. 界面关闭后 -> 执行目标动作
5. 动作完成后 -> 再次读取状态确认效果
```

## 默认架构

默认使用 Fabric 驱动的当前玩家控制：

```text
AI/MCP 客户端 -> agentcontrol-mcp -> AgentControl Fabric 本地端点 -> Minecraft 客户端 API
```

除非用户明确要求，不要重新引入服务端控制、RCON、命令方块或机器人账号方案。

## 项目命名

- 项目统一显示名：`AgentControl`
- Fabric mod id：`agentcontrol`
- Java 包名：`dev.agentcontrol.minecraft.client`
- 产物名：`agentcontrol-fabric-*.jar`
- MCP 服务名：`agentcontrol-mcp`
- 配置文件：`config/agentcontrol.properties`
- 翻译资源路径：`assets/agentcontrol/lang/`

## 已实现功能

### Fabric 侧（agentcontrol-fabric）

- 本地 `/state` 端点：读取玩家坐标、朝向、血量、饥饿值、物品栏、装备、准星目标、附近方块、附近实体、环境信息（生物群系、光照、天气、时间）、氧气、经验、朝向方位、地面状态、世界缓存信息。
- 本地 `/action` 端点：执行移动、视角调整、攻击、使用、破坏/放置方块、关闭界面、释放鼠标、快捷栏切换、丢弃物品、主副手交换。
- 区域方块扫描：通过 `scanRadius` 参数（1-16）调整附近方块扫描范围。
- 世界缓存：按维度隔离，自动保存已探索区域方块到 `.minecraft/agentcontrol-cache/`。
- 通过 Minecraft 客户端 API 执行，不依赖 macOS 系统键盘鼠标控制。
- 可选 Mod Menu 集成，配置项："Capture mouse on release action"。
- 英文和简体中文本地化。

### MCP 侧（agentcontrol-mcp）

- 工具：`get_client_state`（支持 `scan_radius` 参数）、`mod_move_player`、`mod_look`、`mod_attack`、`mod_use_item`、`mod_break_crosshair_block`、`mod_place_crosshair_block`、`mod_close_screen`、`mod_release_mouse`、`mod_select_slot`、`mod_drop`、`mod_swap_hands`。
- 旧的 macOS System Events 工具默认关闭，仅当设置 `MINECRAFT_MCP_ENABLE_SYSTEM_INPUT=1` 时注册。
- 默认走 Fabric 模组通道，不依赖系统辅助功能。

## 安全边界

AgentControl 不绕过服务器权限、反作弊、区域保护、冷却时间或游戏规则。它只通过正常客户端 API 控制用户自己的本地 Minecraft 客户端。

## 本地路径

- MCP 服务：`agentcontrol-mcp/src/server.js`
- Fabric 模组：`agentcontrol-fabric/src/main/java/dev/agentcontrol/minecraft/client/AgentControlClientMod.java`
- 文档站：`agentcontrol-docs/docs/` (VitePress 构建)
- 构建产物：`agentcontrol-fabric/build/libs/agentcontrol-fabric-0.1.0.jar`
- 文档站点：`https://yulimfish.github.io/agentcontrol-docs/`

## 验证命令

检查 MCP 语法：

```sh
cd agentcontrol-mcp && node --check src/server.js
```

构建 Fabric 模组：

```sh
cd agentcontrol-fabric && gradle build
```

## GitHub 发布状态

已上传四个仓库：

- 总仓：`https://github.com/yulimfish/agentcontrol`
- Fabric 副仓：`https://github.com/yulimfish/agentcontrol-fabric`
- MCP 副仓：`https://github.com/yulimfish/agentcontrol-mcp`
- Docs 副仓：`https://github.com/yulimfish/agentcontrol-docs`

总仓通过 Git 子模块引用三个副仓。子仓更新后，在总仓执行 `git submodule update --remote` 并推送，可同步引用 hash。

## 文档站

- 使用 VitePress 构建，位于 `agentcontrol-docs/docs/`
- GitHub Actions 自动构建并部署到 GitHub Pages
- 文档站地址：`https://yulimfish.github.io/agentcontrol-docs/`
- 配置项：`docs/.vitepress/config.mjs`，`base` 必须设为 `/agentcontrol-docs/` 以适配 GitHub Pages 路径

## 许可证

所有子仓库均使用 MIT 许可证。MIT 许可证文本保持英文原文。

## OpenCode 配置注意事项

MCP 入口名已从 `minecraft-player` 改为 `agentcontrol`，命令路径已更新为 `agentcontrol-mcp/src/server.js`。修改 OpenCode 配置后必须退出并重启才能生效。

## 发布说明

根目录可以作为 `AgentControl` 总仓发布。每个子目录也有自己的 `README.md`、`.gitignore` 和 `AGENTS.md`，因此也可以单独作为副仓发布。