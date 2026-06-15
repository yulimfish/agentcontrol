# AgentControl 发布指南

这个工作区已经整理为一个总仓，或一个总仓加三个副仓的发布结构。

## 总仓

仓库名建议：

```text
AgentControl
```

上传这些路径：

- `agentcontrol-fabric/`
- `agentcontrol-mcp/`
- `agentcontrol-docs/`
- `README.md`
- `LICENSE`
- `.gitignore`
- `AGENTS.md`

## 副仓

### AgentControl-Fabric

使用这个目录的内容作为仓库根目录：

```text
agentcontrol-fabric/
```

用途：Minecraft Fabric 客户端模组，负责暴露本地状态和动作接口。

### AgentControl-MCP

使用这个目录的内容作为仓库根目录：

```text
agentcontrol-mcp/
```

用途：MCP 服务，负责让 AI 工具访问 AgentControl Fabric。

### AgentControl-Docs

使用这个目录的内容作为仓库根目录：

```text
agentcontrol-docs/
```

用途：文档、架构说明、安装步骤、安全边界和示例。

## 不要上传

- `node_modules/`
- `.gradle/`
- `build/`
- `bin/`
- `moling/`
- `SESSION_HANDOFF.md`
- 本地 OpenCode 配置文件
- Minecraft 世界、日志、启动器配置、令牌或任何密钥

## 发布检查清单

- 使用 `cd agentcontrol-fabric && gradle clean build` 构建 Fabric。
- 面向最终用户发布 `agentcontrol-fabric/build/libs/agentcontrol-fabric-<version>.jar`。
- 不要把 `*-sources.jar` 当成用户安装用 jar 发布。
- 使用 `cd agentcontrol-mcp && npm install && node --check src/server.js` 检查 MCP。
- 确认文档中的版本号、端点 URL 和工具名称一致。
