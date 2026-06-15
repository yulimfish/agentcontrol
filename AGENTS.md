# AgentControl 总仓 Agent 指南

这个根仓库协调三个可以单独发布的项目：

- `agentcontrol-fabric/`：Fabric 客户端模组，在 Minecraft 内运行并暴露本地状态/动作端点。
- `agentcontrol-mcp/`：MCP 服务，向 AI 工具暴露能力并调用 Fabric 模组端点。
- `agentcontrol-docs/`：文档与跨项目架构说明。

## 默认架构

默认使用 Fabric 驱动的当前玩家控制：

```text
AI/MCP 客户端 -> agentcontrol-mcp -> AgentControl Fabric 本地端点 -> Minecraft 客户端 API
```

除非用户明确要求，不要重新引入服务端控制、RCON、命令方块或机器人账号方案。

## 安全要求

AgentControl 不绕过服务器权限、反作弊、区域保护、冷却时间或游戏规则。它只通过正常客户端 API 控制用户自己的本地 Minecraft 客户端。

## 本地路径

- MCP 服务：`agentcontrol-mcp/src/server.js`
- Fabric 模组：`agentcontrol-fabric/src/main/java/dev/agentcontrol/minecraft/client/AgentControlClientMod.java`
- 文档：`agentcontrol-docs/README.md`

## 验证命令

检查 MCP 语法：

```sh
cd agentcontrol-mcp && node --check src/server.js
```

构建 Fabric 模组：

```sh
cd agentcontrol-fabric && gradle build
```

## 发布说明

根目录可以作为 `AgentControl` 总仓发布。每个子目录也有自己的 `README.md`、`.gitignore` 和 `AGENTS.md`，因此也可以单独作为副仓发布。
