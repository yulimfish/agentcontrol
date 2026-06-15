# AgentControl

AgentControl 是一个本地 Fabric + MCP 桥接项目，用于让 AI 助手观察并控制用户自己的 Minecraft Java 客户端，不需要服务端插件，也不需要额外的机器人账号。

这个仓库是协调总仓，包含三个可以单独发布的子项目。

## 仓库组成

- `agentcontrol-fabric/`：Fabric 客户端模组。它在 Minecraft 客户端内运行，并在 `127.0.0.1` 暴露本地状态和动作接口。
- `agentcontrol-mcp/`：MCP 服务。它向 AI/MCP 客户端暴露工具，并调用 Fabric 模组的本地接口。
- `agentcontrol-docs/`：文档仓。它记录架构、安全边界、安装说明和跨仓库协作信息。

## 仓库关系

AgentControl Fabric 是 Minecraft 内的运行时桥接层。AgentControl MCP 是面向 AI 的工具服务。AgentControl Docs 负责解释两者如何安装、连接和安全扩展。

正常调用链路是：

```text
AI 助手 / MCP 客户端 -> agentcontrol-mcp -> http://127.0.0.1:17777 -> agentcontrol-fabric -> Minecraft 客户端 API
```

`agentcontrol-mcp` 可以在 Minecraft 未启动时启动，但 Fabric 相关工具只有在 `agentcontrol-fabric` 已安装且 Minecraft 客户端正在运行时才可用。`agentcontrol-fabric` 也可以独立加载，但它的主要用途是作为 `agentcontrol-mcp` 消费的本地端点。

## 当前能力

- 读取本地客户端状态：坐标、朝向、血量、饥饿值、物品栏摘要、准星目标、附近方块和附近实体。
- 通过正常 Minecraft 客户端 API 执行移动、视角调整、攻击、使用、放置和破坏。
- 关闭 Minecraft 界面，或打开透明不暂停界面来释放系统鼠标。
- 安装 Mod Menu 时，可以通过 Mod Menu 配置鼠标捕获行为。

## 安全边界

AgentControl 只控制用户自己的本地 Minecraft 客户端。它不会提供服务端访问权限，不会授予权限，不会绕过反作弊、区域保护、冷却时间或游戏规则。

## 构建快速开始

构建 Fabric 模组：

```sh
cd agentcontrol-fabric
gradle build
```

检查 MCP 服务：

```sh
cd agentcontrol-mcp
npm install
node --check src/server.js
```

## 发布策略

推荐的 GitHub 初始布局：

- 创建总仓 `AgentControl`，包含三个子目录。
- 如需模块级 issue、release 或权限管理，再创建 `AgentControl-Fabric`、`AgentControl-MCP`、`AgentControl-Docs` 三个副仓。

每个子目录都已经包含自己的 `README.md`、`.gitignore` 和 `AGENTS.md`，可以直接作为独立仓库上传。
