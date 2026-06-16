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

## 阶段总结

### 已完成的测试轮次

**第一组测试（基础状态与移动）**
- 验证 `get_client_state` 正常工作，返回完整状态
- 验证 `mod_move_player` 能执行移动，位置变化正确
- 发现 `mod_close_screen` 对 ESC 菜单不生效的问题

**第二组测试（界面管理）**
- 验证 `mod_release_mouse` 能打开透明界面释放鼠标
- 验证在透明界面下仍可执行移动命令
- 确认 Minecraft 界面会拦截动作输入

**第三组测试（全面功能验证）**
- 测试移动：前进、左转、跳跃 — 全部正常
- 测试视角：yaw/pitch 调整 — 正常
- 测试快捷栏切换：`mod_select_slot` — 正常
- 测试破坏方块：`mod_break_crosshair_block` — 需要准星对准方块
- 测试攻击：`mod_attack` — 正常
- 测试使用物品：`mod_use_item` — 正常
- 测试主副手交换：`mod_swap_hands` — 正常
- 测试丢弃物品：`mod_drop` — 正常，物品从栏位消失
- 发现 `crosshairTarget` 为 null 时无法破坏方块 — 需要添加准星对准机制

### 已修复的问题

1. **MCP 自动关闭屏幕**：`server.js` 新增 `ensureScreenClosed()`，所有动作工具在执行前自动关闭界面，无需 AI 手动检查
2. **ESC 菜单关闭**：`close_screen` 现在先调用 `screen.close()` 再 `setScreen(null)`，能正确关闭 ESC 菜单
3. **GitHub Actions 构建**：修复 YAML 语法错误，使用纯 `echo` 方式生成 release notes，避免 `**` 字符在 YAML 中解析出错
4. **Gradle 版本兼容性**：移除硬编码的 `gradle-version: 8.8`，让 `setup-gradle` 自动检测项目所需版本（Fabric Loom 1.17.11 需要 Gradle 9.5.0）

### 新增功能（v0.1.1）

**Fabric 模组**：
- `look_at` action：传入 x, y, z 坐标，自动计算 yaw/pitch 对准目标
- `look_facing` action：支持 north/south/east/west/up/down 六个方位

**MCP 服务**：
- `mod_look_at`：对准指定世界坐标（用于破坏方块前瞄准）
- `mod_look_facing`：面朝指定方向（用于移动前转向）
- 所有动作工具自动关闭屏幕（对 AI 透明）

**文档站**：
- API 参考文档更新：新增 `look_at` 和 `look_facing` 说明
- 前置条件说明更新：从"手动检查"改为"自动处理"
- 版本更新记录：新增 v0.1.1 条目

### 已知问题与待办

1. **准星对准机制**：需要确保在执行 `break_crosshair` 或 `place_crosshair` 前，准星确实对准了目标方块。推荐流程：
   ```text
   1. get_client_state → 获取 nearbyBlocks
   2. 分析 nearbyBlocks，选择目标方块
   3. mod_look_at(x, y, z) → 对准方块中心
   4. mod_break_crosshair_block → 破坏方块
   ```

2. **GitHub Actions 缓存**：GitHub 缓存服务偶尔返回 400 错误，不影响构建，但会拖慢后续构建速度

3. **Node.js 20 弃用**：`actions/checkout@v4` 等 Action 基于 Node.js 20，GitHub 将在 2026-09-16 后移除支持。需要关注 Action 更新版本。

4. **移动精度**：当前移动通过 `KeyBinding.setPressed(true)` 实现，在后台线程延迟释放。移动距离取决于帧率，不够精确。如需精确移动，可能需要考虑其他方案（如使用 Minecraft 客户端的 `ClientPlayerEntity` 直接修改坐标，但这可能触发反作弊）。

## 发布说明

根目录可以作为 `AgentControl` 总仓发布。每个子目录也有自己的 `README.md`、`.gitignore` 和 `AGENTS.md`，因此也可以单独作为副仓发布。