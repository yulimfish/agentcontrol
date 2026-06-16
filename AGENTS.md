# AgentControl 总仓 Agent 指南

这个根仓库协调三个可以单独发布的项目：

- `agentcontrol-fabric/`：Fabric 客户端模组，在 Minecraft 内运行并暴露本地状态/动作端点。
- `agentcontrol-mcp/`：MCP 服务，向 AI 工具暴露能力并调用 Fabric 模组端点。
- `agentcontrol-docs/`：文档与跨项目架构说明。

## 行为准则

**所有面向用户的文档、README、说明性文字、界面提示和注释必须使用中文撰写。** 仓库名、目录名、代码标识符、命令、工具名、环境变量和 URL 保持原样不翻译。MIT 许可证文本保持英文原文。

### 知识沉淀准则（关键）

**每次完成工作后，必须确保下一个 Agent 可以在不读任何文档、仅通过运行 MCP 工具的情况下，就能理解所有沉淀下来的内容，并以同样的最佳状态完成任务。**

这意味着：
- 所有关键知识、最佳实践、常见错误、经验教训必须直接沉淀在 MCP 工具的描述（description）中
- 工具描述必须自解释、自包含——不需要额外的 README 或 AGENTS.md 就能正确使用
- 工具描述中必须包含：使用场景、关键参数说明、最佳实践、常见错误、验证步骤
- 如果某个工作流程需要多个步骤，每个步骤对应的工具描述中都必须重复相关的上下文和注意事项
- 示例：如果发现了"必须检查 properties.color 才能区分潜影盒颜色"这个教训，这个信息必须出现在 `mod_break_crosshair_block` 和 `mod_look_at` 的描述中，而不是只在文档里写一次

**原因**：下一个 Agent 的上下文是空的，它不会读取任何文件。它唯一能获取知识的方式就是 MCP 工具注册时的描述字段。工具描述就是 Agent 的"内存"。

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

- 工具：`get_client_state`（支持 `scan_radius` 和 `filter` 参数）、`mod_move_player`、`mod_move_multi`、`mod_move_to`、`mod_look`、`mod_look_at`、`mod_look_facing`、`mod_attack`、`mod_use_item`、`mod_break_crosshair_block`、`mod_break_block`、`mod_place_crosshair_block`、`mod_close_screen`、`mod_release_mouse`、`mod_select_slot`、`mod_drop`、`mod_swap_hands`。
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
- 测试破坏方块：`mod_break_crosshair_block` — 需要准星对准方块，且必须对准方块中心（非完整方块碰撞箱在方块内部，边缘对准会错过）
- 测试攻击：`mod_attack` — 正常
- 测试使用物品：`mod_use_item` — 正常
- 测试主副手交换：`mod_swap_hands` — 正常
- 测试丢弃物品：`mod_drop` — 正常，物品从栏位消失
- 发现 `crosshairTarget` 为 null 时无法破坏方块 — 需要添加准星对准机制，且必须对准方块中心而非方块坐标
- **第四组测试（准星对准与灯笼破坏）**
  - `look_at` 对准方块坐标（x=834, y=72, z=544）→ crosshairTarget 为 null，射线从方块边缘穿过，错过 lantern 碰撞箱
  - `look_at` 对准方块中心（x=834.5, y=72.5, z=544.5）→ crosshairTarget 命中 lantern，成功破坏
  - 结论：`look_at` 必须对准方块中心，而非整数方块坐标

### 已修复的问题

1. **MCP 自动关闭屏幕**：`server.js` 新增 `ensureScreenClosed()`，所有动作工具在执行前自动关闭界面，无需 AI 手动检查
2. **ESC 菜单关闭**：`close_screen` 现在先调用 `screen.close()` 再 `setScreen(null)`，能正确关闭 ESC 菜单
3. **GitHub Actions 构建**：修复 YAML 语法错误，使用纯 `echo` 方式生成 release notes，避免 `**` 字符在 YAML 中解析出错
4. **Gradle 版本兼容性**：移除硬编码的 `gradle-version: 8.8`，让 `setup-gradle` 自动检测项目所需版本（Fabric Loom 1.17.11 需要 Gradle 9.5.0）
5. **准星对准精度**：`look_at` 现在对准方块坐标时，如果传入的是整数，自动加 0.5 偏移对准方块中心。以前对准方块坐标（西南下角），射线从方块边缘穿过，100% 错过非完整方块（如灯笼、告示牌）的碰撞箱。
6. **视线高度计算**：`look_at` 使用 `getEyePos()` 而不是 `getY()`，确保从眼睛位置而不是脚部计算角度。

### 新增功能（v0.1.3）

**Fabric 模组**：
- `crosshairTarget` 返回方块属性：命中方块时返回 `properties` 字段（如 `color: purple`、`facing: east`），允许区分同一方块 ID 的不同变体
- `nearbyBlocks` 支持 `filter` 参数：传入方块 ID 子串（如 `shulker_box`）即可只返回匹配的方块，减少响应数据量

**MCP 服务**：
- `get_client_state` 新增 `filter` 参数：高效搜索特定方块
- `crosshairTarget` 验证增强：工具描述强调检查 `properties.color` 等属性来区分变体
- 移动策略优化：工具描述强调一次性长距离移动（3-5 秒）而不是反复短距离移动+检查

**文档站**：
- API 参考文档更新：新增 `filter` 参数和 `properties` 字段说明
- 版本更新记录：新增 v0.1.3 条目

### 新增功能（v0.1.4）

**Fabric 模组**：
- `break_block` action：传入方块坐标，自动对准方块中心并执行 `attackBlock`。比手动 `look_at` + 验证 + `break_crosshair` 高效得多。在生存模式下有效距离约 4.5 格。
- `move_to` action：传入目标 X/Z 坐标，自动计算方向、估算移动时间（~4.3 格/秒）、检测前方 1 格障碍并自动跳跃。只处理水平移动。

**MCP 服务**：
- `mod_break_block`：坐标破坏工具，一步完成对准+破坏
- `mod_move_to`：坐标移动工具，自动方向+跳跃

### 本次任务经验教训（关键 — 下一个 Agent 必读）

1. **移动精度问题**：`mod_move_player` 基于帧率，精度约 ±2 格。移动 10 格可能 overshoot 到 12 格或 undershoot 到 8 格。**正确做法**：优先使用 `mod_move_to` 到达目标坐标，移动后立即用 `get_client_state` 验证位置。
2. **Z 轴方向混淆**：z 增加=南，z 减少=北。不要想当然地移动，先确认坐标差异再行动。
3. **方块遮挡处理**：目标方块被其他方块覆盖时（如草方块覆盖床），必须先破坏覆盖方块。覆盖方块通常在 y+1 位置。
4. **破坏被覆盖方块的推荐流程**：
   ```text
   1. get_client_state(filter="目标方块") → 获取坐标
   2. 分析 y+1 位置是否有覆盖方块
   3. 如果有 → mod_break_block(x, y+1, z) 先破坏覆盖方块
   4. mod_break_block(x, y, z) 再破坏目标方块
   ```
5. **移动时间估算**：正常行走速度约 4.3 格/秒。`mod_move_to` 自动估算。
6. **跳跃时机**：前方 1 格高度有方块（如潜影盒、台阶）时需要跳跃。`mod_move_to` 自动检测并跳跃。
7. **高效工具选择**：
   - 破坏已知坐标的方块 → `mod_break_block`（一步到位，不要用 look_at + break_crosshair 的两步流程）
   - 移动到目标坐标 → `mod_move_to`（自动方向+跳跃，不要用 look_facing + mod_move_player 的两步流程）
   - 搜索特定方块 → `get_client_state(filter="方块ID")`（不要不加 filter 扫描所有方块）
8. **水中移动**：`mod_move_to` 自动检测水中状态，切换为游泳模式（前进+跳跃同时按，速度约 2 格/秒）。返回 `mode: "swim"` 供 Agent 判断

### 已知问题与待办

1. **GitHub Actions 缓存**：GitHub 缓存服务偶尔返回 400 错误，不影响构建，但会拖慢后续构建速度

2. **Node.js 20 弃用**：`actions/checkout@v4` 等 Action 基于 Node.js 20，GitHub 将在 2026-09-16 后移除支持。需要关注 Action 更新版本。

3. **移动精度**：当前移动通过 `KeyBinding.setPressed(true)` 实现，在后台线程延迟释放。移动距离取决于帧率，不够精确。正常行走速度约 4-5 格/秒。如需精确移动，可能需要考虑其他方案（如使用 Minecraft 客户端的 `ClientPlayerEntity` 直接修改坐标，但这可能触发反作弊）。

### 最佳实践（高效搜索与破坏）

#### 1. 使用 `filter` 高效搜索方块
不要不加过滤地扫描所有方块。使用 `filter` 参数只搜索目标方块：
```text
get_client_state(filter="shulker_box", scan_radius=8) → 只返回潜影盒
```
这比返回 2000+ 个 stone/dirt 的列表高效得多，也减少状态查询次数。

#### 2. 分析相对位置，一次性移动到位
从初始位置获取状态后，分析目标方块相对于玩家的坐标，判断从哪个角度不会被阻挡：
- 如果目标在东边（x > player.x），且中间可能有阻挡 → 先向东移动绕过，再向北/南接近
- 如果目标在南边（z > player.z）→ 面朝南（yaw=0）移动
- 不要反复短距离移动+检查。估算距离（约 4-5 格/秒），一次移动 3-5 秒，再验证位置
- 移动方向：x 增加=东，x 减少=西；z 增加=南，z 减少=北

#### 3. 使用 `mod_look_at` 对准后验证 `properties`
```text
1. get_client_state(filter="shulker_box") → 获取目标坐标
2. mod_look_at(x, y, z) → 模组自动对准方块中心
3. get_client_state → 检查 crosshairTarget.properties.color
4. 如果 color 匹配 → mod_break_crosshair_block
5. 如果 color 不匹配 → 说明命中了其他潜影盒，调整位置重新对准
```
**关键**：不要只检查 `crosshairTarget.block`，必须检查 `crosshairTarget.properties.color` 来区分颜色。

#### 4. 避免无意义的重复状态查询
- 一次移动 3-5 秒后再检查状态，不要每 0.5-1 秒检查一次
- 使用 `filter` 减少 nearbyBlocks 数据量
- 如果 crosshairTarget 为 null，说明当前角度被挡住，应该换个方向移动，而不是反复对准同一位置

#### 5. 推荐的破坏流程（以彩色潜影盒为例）
```text
1. get_client_state(filter="shulker_box", scan_radius=8) → 获取所有潜影盒坐标
2. 分析位置：从玩家位置看，哪些潜影盒会阻挡视线？
3. 选择一个能直接看到目标的角度（如从东侧接近）
4. mod_look_facing + mod_move_player → 移动到该角度
5. mod_look_at(x, y, z) → 对准目标
6. get_client_state → 验证 crosshairTarget.properties.color == "purple"
7. 如果确认 → mod_break_crosshair_block
8. 如果命中的是红色 → 向东/西移动一格，重复步骤 5-7
```

#### 6. 常见错误
- ❌ 不加 filter 扫描所有方块 → 返回 2000+ 行数据，浪费时间和上下文
- ❌ 反复短距离移动+检查 → 应该估算距离，一次移动到位
- ❌ 不检查 properties.color → 可能误破坏红色潜影盒
- ❌ 被挡住时反复对准同一位置 → 应该换个角度移动
- ❌ 混淆 Z 轴方向 → z 增加是向南，减少是向北

## 发布说明

根目录可以作为 `AgentControl` 总仓发布。每个子目录也有自己的 `README.md`、`.gitignore` 和 `AGENTS.md`，因此也可以单独作为副仓发布。