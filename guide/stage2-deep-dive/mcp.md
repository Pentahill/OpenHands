# OpenHands MCP (Model Context Protocol) 集成详解

## 概述

MCP (Model Context Protocol) 是 OpenHands 中用于扩展代理能力的重要协议。通过 MCP，OpenHands 可以连接外部工具和服务，为代理提供更丰富的功能。

### 什么是 MCP？

MCP (Model Context Protocol) 是一个标准化的协议，允许 AI 代理安全地访问外部工具和服务。在 OpenHands 中，MCP 用于：

- 连接外部工具和服务
- 提供标准化的工具调用接口
- 支持多种传输协议（SSE、SHTTP、Stdio）
- 实现工具的动态发现和调用

## 架构设计

### 核心组件

OpenHands 的 MCP 集成包含以下核心组件：

#### 1. MCP 客户端 (`openhands/mcp/client.py`)
- **作用**: 管理与 MCP 服务器的连接和通信
- **关键功能**:
  - 支持 SSE、SHTTP、Stdio 三种传输协议
  - 自动发现和注册服务器提供的工具
  - 提供工具调用接口

#### 2. MCP 工具 (`openhands/mcp/tool.py`)
- **作用**: 表示 MCP 服务器提供的工具
- **关键功能**:
  - 封装工具元数据（名称、描述、参数）
  - 转换为 LLM 可用的函数调用格式

#### 3. MCP 配置 (`openhands/core/config/mcp_config.py`)
- **作用**: 管理 MCP 服务器配置
- **支持的服务器类型**:
  - `MCPSSEServerConfig`: SSE 服务器配置
  - `MCPSHTTPServerConfig`: SHTTP 服务器配置
  - `MCPStdioServerConfig`: Stdio 服务器配置

#### 4. MCP 代理管理器 (`openhands/runtime/mcp/proxy/manager.py`)
- **作用**: 管理 FastMCP 代理实例
- **关键功能**:
  - 初始化 FastMCP 代理
  - 配置代理工具
  - 挂载到 FastAPI 应用

#### 5. MCP 工具集 (`openhands/mcp/utils.py`)
- **作用**: 提供 MCP 相关的工具函数
- **关键功能**:
  - 创建 MCP 客户端
  - 转换工具格式
  - 调用 MCP 工具

### 数据传输流程

```
Agent → MCPAction → Runtime → MCP Client → MCP Server → Tool Execution
     ↓
Observation ← MCPObservation ← Runtime ← MCP Client ← MCP Server
```

## 配置详解

### MCP 配置结构

MCP 配置通过 `MCPConfig` 类管理，支持三种类型的服务器：

```python
class MCPConfig(BaseModel):
    sse_servers: list[MCPSSEServerConfig] = Field(default_factory=list)
    stdio_servers: list[MCPStdioServerConfig] = Field(default_factory=list)
    shttp_servers: list[MCPSHTTPServerConfig] = Field(default_factory=list)
```

### 服务器配置示例

#### SSE 服务器配置
```python
MCPSSEServerConfig(
    url="http://localhost:8080/sse",
    api_key="your-api-key"
)
```

#### SHTTP 服务器配置
```python
MCPSHTTPServerConfig(
    url="http://localhost:8080/shttp",
    api_key="your-api-key"
)
```

#### Stdio 服务器配置
```python
MCPStdioServerConfig(
    name="tavily",
    command="npx",
    args=["-y", "tavily-mcp@0.2.1"],
    env={"TAVILY_API_KEY": "your-api-key"}
)
```

## 实现细节

### MCP 客户端实现

#### 连接管理
MCP 客户端支持三种连接方式：

1. **HTTP 连接** (SSE/SHTTP):
   ```python
   async def connect_http(
       self,
       server: MCPSSEServerConfig | MCPSHTTPServerConfig,
       conversation_id: str | None = None,
       timeout: float = 30.0
   )
   ```

2. **Stdio 连接**:
   ```python
   async def connect_stdio(self, server: MCPStdioServerConfig, timeout: float = 30.0)
   ```

#### 工具发现
连接建立后，客户端会自动发现服务器提供的工具：

```python
async def _initialize_and_list_tools(self) -> None:
    if not self.client:
        raise RuntimeError('Session not initialized.')
    
    async with self.client:
        tools = await self.client.list_tools()
    
    # 创建工具对象
    for tool in tools:
        server_tool = MCPClientTool(
            name=tool.name,
            description=tool.description,
            inputSchema=tool.inputSchema,
            session=self.client,
        )
        self.tool_map[tool.name] = server_tool
        self.tools.append(server_tool)
```

### 工具调用流程

#### 1. 创建 MCP 客户端
```python
mcp_clients = await create_mcp_clients(
    sse_servers=mcp_config.sse_servers,
    shttp_servers=mcp_config.shttp_servers,
    conversation_id=conversation_id,
    stdio_servers=mcp_config.stdio_servers,
)
```

#### 2. 查找匹配的客户端
```python
matching_client = None
for client in mcp_clients:
    if action.name in [tool.name for tool in client.tools]:
        matching_client = client
        break
```

#### 3. 调用工具
```python
response = await matching_client.call_tool(action.name, action.arguments)
```

#### 4. 返回观察结果
```python
return MCPObservation(
    content=json.dumps(response.model_dump(mode='json')),
    name=action.name,
    arguments=action.arguments,
)
```

### 错误处理

#### MCP 错误收集器
OpenHands 提供了专门的错误收集器来捕获 MCP 相关的错误：

```python
class MCPErrorCollector:
    """Thread-safe collector for MCP errors during startup."""
    
    def add_error(
        self,
        server_name: str,
        server_type: str,
        error_message: str,
        exception_details: str | None = None,
    ) -> None
```

#### 错误处理策略
- 连接失败时记录错误但继续运行
- 工具调用失败时返回错误观察结果
- 支持错误信息的收集和查询

## 运行时集成

### CLI 运行时集成

在 CLI 运行时中，MCP 工具通过以下方式集成：

```python
async def call_tool_mcp(self, action: MCPAction) -> Observation:
    """Execute an MCP tool action in CLI runtime."""
    
    # 获取 MCP 配置
    mcp_config = self.get_mcp_config()
    
    # 创建 MCP 客户端
    mcp_clients = await create_mcp_clients(
        mcp_config.sse_servers,
        mcp_config.shttp_servers,
        self.sid,
        mcp_config.stdio_servers,
    )
    
    # 调用工具
    result = await call_tool_mcp_handler(mcp_clients, action)
    return result
```

### 代理集成

MCP 工具通过以下方式集成到代理中：

```python
async def add_mcp_tools_to_agent(
    agent: 'Agent', runtime: Runtime, memory: 'Memory'
) -> MCPConfig:
    """Add MCP tools to an agent."""
    
    # 获取更新的 MCP 配置
    updated_mcp_config = runtime.get_mcp_config(extra_stdio_servers)
    
    # 获取 MCP 工具
    mcp_tools = await fetch_mcp_tools_from_config(
        updated_mcp_config, use_stdio=isinstance(runtime, CLIRuntime)
    )
    
    # 设置代理的 MCP 工具
    agent.set_mcp_tools(mcp_tools)
    
    return updated_mcp_config
```

## 服务器端实现

### MCP 路由

OpenHands 提供了内置的 MCP 工具，位于 `openhands/server/routes/mcp.py`：

#### 创建 PR 工具
```python
@mcp_server.tool()
async def create_pr(
    repo_name: str,
    source_branch: str,
    target_branch: str,
    title: str,
    body: str | None,
    draft: bool = True,
    labels: list[str] | None = None,
) -> str:
    """Open a PR in GitHub"""
```

#### 创建 MR 工具
```python
@mcp_server.tool()
async def create_mr(
    id: int | str,
    source_branch: str,
    target_branch: str,
    title: str,
    description: str | None,
    labels: list[str] | None = None,
) -> str:
    """Open a MR in GitLab"""
```

#### 创建 Bitbucket PR 工具
```python
@mcp_server.tool()
async def create_bitbucket_pr(
    repo_name: str,
    source_branch: str,
    target_branch: str,
    title: str,
    description: str | None,
) -> str:
    """Open a PR in Bitbucket"""
```

## 事件系统集成

### MCP Action

MCP 操作通过 `MCPAction` 类表示：

```python
@dataclass
class MCPAction(Action):
    name: str
    arguments: dict[str, Any] = field(default_factory=dict)
    thought: str = ''
    action: str = ActionType.MCP
    runnable: ClassVar[bool] = True
    security_risk: ActionSecurityRisk | None = None
```

### MCP Observation

MCP 观察结果通过 `MCPObservation` 类表示：

```python
@dataclass
class MCPObservation(Observation):
    """This data class represents the result of a MCP Server operation."""
    
    observation: str = ObservationType.MCP
    name: str = ''  # The name of the MCP tool that was called
    arguments: dict[str, Any] = field(default_factory=dict)
```

## 使用示例

### 基本使用流程

1. **配置 MCP 服务器**
```python
mcp_config = MCPConfig(
    sse_servers=[
        MCPSSEServerConfig(url="http://localhost:8080/sse")
    ],
    stdio_servers=[
        MCPStdioServerConfig(
            name="tavily",
            command="npx",
            args=["-y", "tavily-mcp@0.2.1"],
            env={"TAVILY_API_KEY": "your-api-key"}
        )
    ]
)
```

2. **创建 MCP 客户端**
```python
mcp_clients = await create_mcp_clients(
    mcp_config.sse_servers,
    mcp_config.shttp_servers,
    conversation_id="test-conversation",
    stdio_servers=mcp_config.stdio_servers,
)
```

3. **调用 MCP 工具**
```python
action = MCPAction(
    name="search_tool",
    arguments={"query": "OpenHands MCP integration"}
)

result = await call_tool_mcp(mcp_clients, action)
```

### 集成到代理

```python
# 在代理初始化时添加 MCP 工具
mcp_config = await add_mcp_tools_to_agent(agent, runtime, memory)

# 代理现在可以使用 MCP 工具
response = await agent.run("使用搜索工具查找 OpenHands 相关信息")
```

## 最佳实践

### 1. 配置管理
- 使用环境变量管理敏感信息（如 API keys）
- 为不同的环境配置不同的服务器
- 定期验证服务器连接状态

### 2. 错误处理
- 实现优雅的降级策略
- 记录详细的错误信息
- 提供用户友好的错误消息

### 3. 性能优化
- 合理设置连接超时时间
- 使用连接池管理 MCP 客户端
- 监控工具调用性能

### 4. 安全性
- 验证服务器证书
- 使用安全的传输协议
- 限制工具调用权限

## 限制和注意事项

### 平台限制
- **Windows**: MCP 功能在 Windows 平台上被禁用
- **其他平台**: 支持 Linux 和 macOS

### 性能考虑
- MCP 工具调用涉及网络通信，可能影响性能
- 建议对关键工具实现缓存机制
- 监控工具调用延迟

### 错误恢复
- 连接失败时自动重试
- 提供备选工具或降级方案
- 记录详细的错误日志

## 扩展开发

### 开发新的 MCP 工具

1. **定义工具函数**
```python
@mcp_server.tool()
async def my_custom_tool(
    param1: str,
    param2: int,
) -> str:
    """My custom tool description."""
    # 工具实现
    return "Tool result"
```

2. **注册到 MCP 服务器**
```python
mcp_server = FastMCP('my-server')
```

3. **配置到 OpenHands**
```python
mcp_config = MCPConfig(
    sse_servers=[
        MCPSSEServerConfig(url="http://localhost:8080/sse")
    ]
)
```

## 总结

OpenHands 的 MCP 集成提供了一个强大而灵活的框架，用于扩展代理的能力。通过标准化的协议和丰富的工具支持，开发者可以轻松地集成外部服务，为代理提供更丰富的功能。

关键优势：
- **标准化**: 基于 MCP 协议，兼容各种工具和服务
- **灵活性**: 支持多种传输协议和服务器类型
- **可扩展**: 易于添加新的工具和服务
- **安全性**: 提供完整的错误处理和安全管理

通过深入了解 MCP 的架构和实现，开发者可以更好地利用这一功能，构建更强大的 AI 代理应用。