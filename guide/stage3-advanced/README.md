# 阶段三：高级开发（3-4周）

## 🎯 学习目标

在这个阶段，你将：
- 掌握前端React开发的高级技巧
- 深入理解后端FastAPI架构和开发
- 学习LLM集成和优化技术
- 掌握性能调优和监控技术
- 能够开发完整的功能模块

## 📋 学习清单

### 第1-5天：前端开发深入
- [ ] 掌握React 19的新特性和最佳实践
- [ ] 深入理解Redux Toolkit状态管理
- [ ] 学习TanStack Query数据获取和缓存
- [ ] 掌握Socket.IO实时通信
- [ ] 理解Monaco Editor集成和定制

### 第6-10天：后端API开发
- [ ] 深入学习FastAPI框架和异步编程
- [ ] 理解依赖注入和中间件机制
- [ ] 掌握WebSocket和SSE实现
- [ ] 学习数据验证和序列化
- [ ] 实现RESTful API设计

### 第11-15天：LLM集成优化
- [ ] 深入理解LiteLLM多模型支持
- [ ] 学习提示工程和优化技巧
- [ ] 掌握函数调用和工具使用
- [ ] 理解流式响应和异步处理
- [ ] 实现自定义LLM集成

### 第16-20天：性能调优
- [ ] 分析系统性能瓶颈
- [ ] 优化数据库查询和缓存
- [ ] 实现异步处理和并发优化
- [ ] 学习内存管理和垃圾回收
- [ ] 掌握负载测试和性能监控

### 第21-28天：综合项目开发
- [ ] 设计和实现完整功能模块
- [ ] 集成前后端开发
- [ ] 实现测试和文档
- [ ] 性能优化和部署准备
- [ ] 准备进入下一阶段

## 📚 核心技术栈深入

### 1. 前端技术栈
```
React 19
├── 新特性
│   ├── Server Components
│   ├── Concurrent Features
│   ├── Automatic Batching
│   └── Suspense Improvements
├── 状态管理
│   ├── Redux Toolkit
│   ├── RTK Query
│   └── Context API
├── 数据获取
│   ├── TanStack Query
│   ├── SWR模式
│   └── 缓存策略
└── UI组件
    ├── HeroUI
    ├── Tailwind CSS
    └── Framer Motion
```

### 2. 后端技术栈
```
FastAPI
├── 核心特性
│   ├── 异步支持
│   ├── 自动文档生成
│   ├── 类型验证
│   └── 依赖注入
├── 数据处理
│   ├── Pydantic模型
│   ├── 序列化/反序列化
│   └── 数据验证
├── 实时通信
│   ├── WebSocket
│   ├── Server-Sent Events
│   └── Socket.IO
└── 中间件
    ├── CORS处理
    ├── 认证授权
    └── 错误处理
```

### 3. LLM集成技术
```
LiteLLM
├── 多模型支持
│   ├── OpenAI
│   ├── Anthropic
│   ├── Google
│   └── 本地模型
├── 功能特性
│   ├── 函数调用
│   ├── 流式响应
│   ├── 提示缓存
│   └── 错误重试
├── 优化技术
│   ├── 批处理
│   ├── 并发控制
│   ├── 负载均衡
│   └── 成本优化
└── 自定义集成
    ├── 自定义提供商
    ├── 模型适配器
    └── 中间件扩展
```

## 🛠️ 实践项目

### 项目1：高级前端组件开发
创建一个智能代码编辑器组件：

```typescript
// SmartCodeEditor.tsx
import React, { useCallback, useEffect, useRef } from 'react';
import { Editor } from '@monaco-editor/react';
import { useQuery, useMutation } from '@tanstack/react-query';
import { useSocket } from '../hooks/useSocket';

interface SmartCodeEditorProps {
  language: string;
  initialValue?: string;
  onSave?: (content: string) => void;
  enableAIAssist?: boolean;
}

export const SmartCodeEditor: React.FC<SmartCodeEditorProps> = ({
  language,
  initialValue = '',
  onSave,
  enableAIAssist = true
}) => {
  const editorRef = useRef<any>(null);
  const socket = useSocket();

  // 获取语言配置
  const { data: languageConfig } = useQuery({
    queryKey: ['language-config', language],
    queryFn: () => fetchLanguageConfig(language)
  });

  // AI代码建议
  const { mutate: getAISuggestions } = useMutation({
    mutationFn: (context: string) => getCodeSuggestions(context),
    onSuccess: (suggestions) => {
      // 显示AI建议
      showSuggestions(suggestions);
    }
  });

  // 编辑器挂载
  const handleEditorDidMount = useCallback((editor: any, monaco: any) => {
    editorRef.current = editor;

    // 配置编辑器
    setupEditor(editor, monaco, languageConfig);

    // 注册快捷键
    editor.addCommand(monaco.KeyMod.CtrlCmd | monaco.KeyCode.KeyS, () => {
      const content = editor.getValue();
      onSave?.(content);
    });

    // AI辅助
    if (enableAIAssist) {
      setupAIAssist(editor, monaco);
    }
  }, [languageConfig, onSave, enableAIAssist]);

  // 内容变化处理
  const handleEditorChange = useCallback((value: string | undefined) => {
    if (enableAIAssist && value) {
      // 延迟获取AI建议
      debounceGetSuggestions(value);
    }
  }, [enableAIAssist]);

  // 实时协作
  useEffect(() => {
    if (socket) {
      socket.on('code-change', (change: any) => {
        // 应用远程更改
        applyRemoteChange(change);
      });

      return () => {
        socket.off('code-change');
      };
    }
  }, [socket]);

  return (
    <div className="smart-code-editor">
      <Editor
        height="400px"
        language={language}
        value={initialValue}
        onMount={handleEditorDidMount}
        onChange={handleEditorChange}
        options={{
          theme: 'vs-dark',
          fontSize: 14,
          minimap: { enabled: false },
          scrollBeyondLastLine: false,
          automaticLayout: true,
          suggestOnTriggerCharacters: true,
          quickSuggestions: true,
          wordBasedSuggestions: true
        }}
      />
    </div>
  );
};

// 辅助函数
const setupEditor = (editor: any, monaco: any, config: any) => {
  // 配置语言特性
  if (config) {
    monaco.languages.setLanguageConfiguration(config.language, config.configuration);
    monaco.languages.setMonarchTokensProvider(config.language, config.tokenizer);
  }
};

const setupAIAssist = (editor: any, monaco: any) => {
  // 注册AI建议提供者
  monaco.languages.registerCompletionItemProvider('typescript', {
    provideCompletionItems: async (model: any, position: any) => {
      const context = model.getValueInRange({
        startLineNumber: Math.max(1, position.lineNumber - 10),
        startColumn: 1,
        endLineNumber: position.lineNumber,
        endColumn: position.column
      });

      const suggestions = await getCodeSuggestions(context);
      return { suggestions };
    }
  });
};
```

### 项目2：高级后端API开发
创建一个智能任务处理API：

```python
# smart_task_api.py
from fastapi import FastAPI, Depends, HTTPException, BackgroundTasks
from fastapi.websockets import WebSocket, WebSocketDisconnect
from pydantic import BaseModel, Field
from typing import List, Optional, Dict, Any
import asyncio
import json
from datetime import datetime

app = FastAPI(title="Smart Task API", version="1.0.0")

class TaskRequest(BaseModel):
    task_type: str = Field(..., description="任务类型")
    content: str = Field(..., description="任务内容")
    priority: int = Field(default=1, ge=1, le=5, description="优先级")
    metadata: Optional[Dict[str, Any]] = Field(default=None, description="元数据")

class TaskResponse(BaseModel):
    task_id: str
    status: str
    result: Optional[Dict[str, Any]] = None
    created_at: datetime
    updated_at: datetime

class TaskManager:
    def __init__(self):
        self.tasks: Dict[str, Dict] = {}
        self.task_queue = asyncio.Queue()
        self.workers = []
        self.websocket_connections: List[WebSocket] = []

    async def start_workers(self, num_workers: int = 3):
        """启动任务处理工作者"""
        for i in range(num_workers):
            worker = asyncio.create_task(self._worker(f"worker-{i}"))
            self.workers.append(worker)

    async def _worker(self, worker_id: str):
        """任务处理工作者"""
        while True:
            try:
                task = await self.task_queue.get()
                await self._process_task(task, worker_id)
                self.task_queue.task_done()
            except Exception as e:
                print(f"Worker {worker_id} error: {e}")

    async def _process_task(self, task: Dict, worker_id: str):
        """处理具体任务"""
        task_id = task['id']
        task_type = task['type']

        # 更新任务状态
        self.tasks[task_id]['status'] = 'processing'
        self.tasks[task_id]['worker'] = worker_id
        await self._broadcast_task_update(task_id)

        try:
            # 根据任务类型处理
            if task_type == 'code_analysis':
                result = await self._analyze_code(task['content'])
            elif task_type == 'text_generation':
                result = await self._generate_text(task['content'])
            elif task_type == 'data_processing':
                result = await self._process_data(task['content'])
            else:
                raise ValueError(f"Unknown task type: {task_type}")

            # 更新任务结果
            self.tasks[task_id]['status'] = 'completed'
            self.tasks[task_id]['result'] = result
            self.tasks[task_id]['updated_at'] = datetime.now()

        except Exception as e:
            self.tasks[task_id]['status'] = 'failed'
            self.tasks[task_id]['error'] = str(e)
            self.tasks[task_id]['updated_at'] = datetime.now()

        await self._broadcast_task_update(task_id)

    async def _broadcast_task_update(self, task_id: str):
        """广播任务更新"""
        task_data = self.tasks[task_id]
        message = {
            'type': 'task_update',
            'task_id': task_id,
            'status': task_data['status'],
            'updated_at': task_data['updated_at'].isoformat()
        }

        # 发送给所有连接的WebSocket客户端
        for websocket in self.websocket_connections[:]:
            try:
                await websocket.send_text(json.dumps(message))
            except:
                self.websocket_connections.remove(websocket)

    async def _analyze_code(self, code: str) -> Dict[str, Any]:
        """代码分析任务"""
        # 模拟异步处理
        await asyncio.sleep(2)

        # 这里可以集成真实的代码分析工具
        return {
            'lines': len(code.split('\n')),
            'complexity': 'medium',
            'suggestions': ['Add type hints', 'Improve error handling']
        }

    async def _generate_text(self, prompt: str) -> Dict[str, Any]:
        """文本生成任务"""
        # 模拟LLM调用
        await asyncio.sleep(3)

        return {
            'generated_text': f"Generated response for: {prompt}",
            'tokens_used': 150,
            'model': 'gpt-4'
        }

    async def _process_data(self, data: str) -> Dict[str, Any]:
        """数据处理任务"""
        await asyncio.sleep(1)

        return {
            'processed_items': 100,
            'processing_time': '1.2s',
            'status': 'success'
        }

# 全局任务管理器
task_manager = TaskManager()

@app.on_event("startup")
async def startup_event():
    """应用启动事件"""
    await task_manager.start_workers(3)

@app.post("/tasks", response_model=TaskResponse)
async def create_task(
    task_request: TaskRequest,
    background_tasks: BackgroundTasks
):
    """创建新任务"""
    import uuid

    task_id = str(uuid.uuid4())
    now = datetime.now()

    # 创建任务记录
    task_data = {
        'id': task_id,
        'type': task_request.task_type,
        'content': task_request.content,
        'priority': task_request.priority,
        'metadata': task_request.metadata or {},
        'status': 'pending',
        'created_at': now,
        'updated_at': now
    }

    task_manager.tasks[task_id] = task_data

    # 添加到任务队列
    await task_manager.task_queue.put(task_data)

    return TaskResponse(
        task_id=task_id,
        status='pending',
        created_at=now,
        updated_at=now
    )

@app.get("/tasks/{task_id}", response_model=TaskResponse)
async def get_task(task_id: str):
    """获取任务状态"""
    if task_id not in task_manager.tasks:
        raise HTTPException(status_code=404, detail="Task not found")

    task_data = task_manager.tasks[task_id]
    return TaskResponse(
        task_id=task_id,
        status=task_data['status'],
        result=task_data.get('result'),
        created_at=task_data['created_at'],
        updated_at=task_data['updated_at']
    )

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    """WebSocket连接端点"""
    await websocket.accept()
    task_manager.websocket_connections.append(websocket)

    try:
        while True:
            # 保持连接活跃
            await websocket.receive_text()
    except WebSocketDisconnect:
        task_manager.websocket_connections.remove(websocket)

@app.get("/health")
async def health_check():
    """健康检查"""
    return {
        "status": "healthy",
        "timestamp": datetime.now().isoformat(),
        "active_tasks": len([t for t in task_manager.tasks.values() if t['status'] == 'processing']),
        "pending_tasks": task_manager.task_queue.qsize()
    }
```

### 项目3：LLM集成优化
创建一个智能LLM管理器：

```python
# smart_llm_manager.py
import asyncio
import time
from typing import Dict, List, Optional, AsyncGenerator
from dataclasses import dataclass
from enum import Enum
import litellm
from litellm import completion, acompletion
import json

class ModelProvider(Enum):
    OPENAI = "openai"
    ANTHROPIC = "anthropic"
    GOOGLE = "google"
    LOCAL = "local"

@dataclass
class ModelConfig:
    name: str
    provider: ModelProvider
    max_tokens: int
    cost_per_token: float
    supports_functions: bool
    supports_streaming: bool

class SmartLLMManager:
    def __init__(self):
        self.models: Dict[str, ModelConfig] = {}
        self.usage_stats: Dict[str, Dict] = {}
        self.rate_limits: Dict[str, Dict] = {}
        self.load_balancer = LoadBalancer()

        # 初始化模型配置
        self._initialize_models()

    def _initialize_models(self):
        """初始化模型配置"""
        self.models.update({
            "gpt-4": ModelConfig(
                name="gpt-4",
                provider=ModelProvider.OPENAI,
                max_tokens=8192,
                cost_per_token=0.00003,
                supports_functions=True,
                supports_streaming=True
            ),
            "claude-3-sonnet": ModelConfig(
                name="claude-3-sonnet-20240229",
                provider=ModelProvider.ANTHROPIC,
                max_tokens=4096,
                cost_per_token=0.000015,
                supports_functions=True,
                supports_streaming=True
            ),
            "gemini-pro": ModelConfig(
                name="gemini-pro",
                provider=ModelProvider.GOOGLE,
                max_tokens=2048,
                cost_per_token=0.00001,
                supports_functions=False,
                supports_streaming=True
            )
        })

    async def smart_completion(
        self,
        messages: List[Dict],
        task_type: str = "general",
        requirements: Optional[Dict] = None
    ) -> Dict:
        """智能模型选择和完成"""

        # 选择最适合的模型
        model_name = await self._select_best_model(
            messages, task_type, requirements
        )

        # 优化提示
        optimized_messages = await self._optimize_prompt(messages, model_name)

        # 执行完成
        try:
            response = await self._execute_completion(
                model_name, optimized_messages, requirements
            )

            # 记录使用统计
            await self._record_usage(model_name, response)

            return response

        except Exception as e:
            # 故障转移
            return await self._fallback_completion(
                messages, task_type, requirements, str(e)
            )

    async def _select_best_model(
        self,
        messages: List[Dict],
        task_type: str,
        requirements: Optional[Dict]
    ) -> str:
        """选择最佳模型"""

        # 分析任务特征
        task_features = await self._analyze_task(messages, task_type)

        # 评分模型
        model_scores = {}
        for model_name, config in self.models.items():
            score = await self._score_model(config, task_features, requirements)
            model_scores[model_name] = score

        # 考虑负载均衡
        balanced_scores = await self.load_balancer.adjust_scores(model_scores)

        # 选择最高分模型
        best_model = max(balanced_scores.items(), key=lambda x: x[1])[0]
        return best_model

    async def _analyze_task(self, messages: List[Dict], task_type: str) -> Dict:
        """分析任务特征"""
        total_tokens = sum(len(msg.get('content', '').split()) for msg in messages)

        features = {
            'token_count': total_tokens,
            'complexity': self._estimate_complexity(messages),
            'requires_functions': 'function_call' in str(messages),
            'requires_streaming': task_type in ['code_generation', 'long_text'],
            'task_type': task_type
        }

        return features

    async def _score_model(
        self,
        config: ModelConfig,
        features: Dict,
        requirements: Optional[Dict]
    ) -> float:
        """为模型评分"""
        score = 100.0

        # 基于成本的评分
        cost_factor = 1.0 / (config.cost_per_token * 1000000)  # 转换为每百万token
        score *= cost_factor

        # 基于能力的评分
        if features['requires_functions'] and not config.supports_functions:
            score *= 0.1  # 严重降分

        if features['requires_streaming'] and not config.supports_streaming:
            score *= 0.8

        # 基于负载的评分
        current_load = await self._get_model_load(config.name)
        load_factor = max(0.1, 1.0 - current_load)
        score *= load_factor

        # 基于历史性能的评分
        performance_factor = await self._get_performance_factor(config.name)
        score *= performance_factor

        return score

    async def _optimize_prompt(
        self,
        messages: List[Dict],
        model_name: str
    ) -> List[Dict]:
        """优化提示"""
        config = self.models[model_name]

        # 根据模型特点优化
        if config.provider == ModelProvider.ANTHROPIC:
            # Claude喜欢更结构化的提示
            return self._optimize_for_claude(messages)
        elif config.provider == ModelProvider.OPENAI:
            # GPT-4优化
            return self._optimize_for_gpt(messages)
        else:
            return messages

    async def _execute_completion(
        self,
        model_name: str,
        messages: List[Dict],
        requirements: Optional[Dict]
    ) -> Dict:
        """执行模型完成"""
        config = self.models[model_name]

        # 构建请求参数
        params = {
            "model": model_name,
            "messages": messages,
            "max_tokens": min(requirements.get('max_tokens', 1000), config.max_tokens),
            "temperature": requirements.get('temperature', 0.7)
        }

        # 添加函数调用支持
        if requirements and requirements.get('functions') and config.supports_functions:
            params['functions'] = requirements['functions']
            params['function_call'] = requirements.get('function_call', 'auto')

        # 执行请求
        if requirements and requirements.get('stream', False) and config.supports_streaming:
            return await self._stream_completion(params)
        else:
            response = await acompletion(**params)
            return response

    async def _stream_completion(self, params: Dict) -> AsyncGenerator[Dict, None]:
        """流式完成"""
        params['stream'] = True

        async for chunk in await acompletion(**params):
            yield chunk

    async def _record_usage(self, model_name: str, response: Dict):
        """记录使用统计"""
        if model_name not in self.usage_stats:
            self.usage_stats[model_name] = {
                'total_requests': 0,
                'total_tokens': 0,
                'total_cost': 0.0,
                'avg_response_time': 0.0,
                'error_rate': 0.0
            }

        stats = self.usage_stats[model_name]
        stats['total_requests'] += 1

        if hasattr(response, 'usage'):
            tokens = response.usage.total_tokens
            stats['total_tokens'] += tokens

            config = self.models[model_name]
            cost = tokens * config.cost_per_token
            stats['total_cost'] += cost

class LoadBalancer:
    def __init__(self):
        self.model_loads: Dict[str, float] = {}
        self.request_counts: Dict[str, int] = {}

    async def adjust_scores(self, scores: Dict[str, float]) -> Dict[str, float]:
        """根据负载调整分数"""
        adjusted_scores = {}

        for model_name, score in scores.items():
            load = self.model_loads.get(model_name, 0.0)
            load_factor = max(0.1, 1.0 - load)
            adjusted_scores[model_name] = score * load_factor

        return adjusted_scores

    async def update_load(self, model_name: str, load: float):
        """更新模型负载"""
        self.model_loads[model_name] = load

    async def increment_requests(self, model_name: str):
        """增加请求计数"""
        self.request_counts[model_name] = self.request_counts.get(model_name, 0) + 1

# 使用示例
async def main():
    llm_manager = SmartLLMManager()

    messages = [
        {"role": "user", "content": "请帮我分析这段Python代码的性能问题"}
    ]

    response = await llm_manager.smart_completion(
        messages=messages,
        task_type="code_analysis",
        requirements={
            "max_tokens": 1000,
            "temperature": 0.3,
            "functions": [
                {
                    "name": "analyze_code",
                    "description": "分析代码性能",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "issues": {"type": "array", "items": {"type": "string"}},
                            "suggestions": {"type": "array", "items": {"type": "string"}}
                        }
                    }
                }
            ]
        }
    )

    print(response)

if __name__ == "__main__":
    asyncio.run(main())
```

## 📊 学习进度跟踪

| 技术领域 | 理论学习 | 实践开发 | 项目完成 | 掌握程度 |
|---------|----------|----------|----------|----------|
| React高级开发 | ⏳ | ⏳ | ⏳ | 0% |
| FastAPI后端 | ⏳ | ⏳ | ⏳ | 0% |
| LLM集成优化 | ⏳ | ⏳ | ⏳ | 0% |
| 性能调优 | ⏳ | ⏳ | ⏳ | 0% |
| 综合项目 | ⏳ | ⏳ | ⏳ | 0% |

## 🎯 核心技能目标

### 前端开发技能
- [ ] 熟练使用React 19新特性
- [ ] 掌握复杂状态管理
- [ ] 实现高性能组件
- [ ] 集成实时通信功能

### 后端开发技能
- [ ] 设计RESTful API
- [ ] 实现异步处理
- [ ] 集成WebSocket通信
- [ ] 优化数据库操作

### LLM集成技能
- [ ] 多模型管理
- [ ] 智能模型选择
- [ ] 提示工程优化
- [ ] 成本控制策略

### 性能优化技能
- [ ] 性能瓶颈分析
- [ ] 缓存策略设计
- [ ] 并发处理优化
- [ ] 监控和告警

## 🔧 开发工具和技巧

### 前端开发工具
- **React DevTools**: 组件调试和性能分析
- **Redux DevTools**: 状态管理调试
- **Lighthouse**: 性能和可访问性测试
- **Storybook**: 组件开发和测试

### 后端开发工具
- **FastAPI自动文档**: 交互式API文档
- **Uvicorn**: 高性能ASGI服务器
- **Pydantic**: 数据验证和序列化
- **SQLAlchemy**: ORM和数据库操作

### 性能分析工具
- **Chrome DevTools**: 前端性能分析
- **cProfile**: Python性能分析
- **Memory Profiler**: 内存使用分析
- **Load Testing**: 压力测试工具

## ➡️ 下一阶段

完成本阶段学习后，你应该：
- 具备全栈开发能力
- 能够优化系统性能
- 掌握LLM集成技术
- 为扩展开发做好准备

准备好了吗？让我们进入 [阶段四：扩展开发](../stage4-extension/) 吧！
