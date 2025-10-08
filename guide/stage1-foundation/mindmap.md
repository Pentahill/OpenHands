# 阶段一学习脑图：基础理解

```mermaid
mindmap
  root((阶段一<br/>基础理解<br/>1-2周))
    环境搭建
      系统要求
        Python 3.12+
        Node.js 22+
        Docker
        Poetry
        Git
      安装步骤
        克隆代码
        make build
        依赖安装
        环境验证
      开发工具
        IDE配置
        调试设置
        Git配置

    核心概念
      Agent系统
        CodeActAgent
        BrowsingAgent
        ReadOnlyAgent
        VisualBrowsingAgent
      事件模式
        Action执行
        Observation反馈
        Event流处理
        状态管理
      运行时
        LocalRuntime
        DockerRuntime
        RemoteRuntime
        插件系统

    项目结构
      后端架构
        openhands/core
        openhands/agenthub
        openhands/events
        openhands/runtime
        openhands/server
      前端架构
        frontend/src/components
        frontend/src/state
        frontend/src/api
        frontend/src/routes
      配置文件
        pyproject.toml
        package.json
        config.toml

    基础实践
      运行项目
        启动后端
        启动前端
        功能验证
      简单任务
        文件操作
        命令执行
        代码编辑
      日志分析
        错误排查
        性能监控
        调试技巧

    学习资源
      官方文档
        README.md
        Development.md
        API文档
      代码示例
        测试用例
        示例项目
        最佳实践
      社区资源
        GitHub Issues
        Slack社区
        Discord频道
```

## 🎯 重点学习路径

### 1. 环境搭建路径
```
系统检查 → 依赖安装 → 项目构建 → 功能验证 → 开发配置
```

### 2. 概念理解路径
```
Agent概念 → 事件模式 → 运行时系统 → 工具集成 → 架构理解
```

### 3. 代码探索路径
```
入口文件 → 核心模块 → 配置系统 → 工具函数 → 测试代码
```

### 4. 实践学习路径
```
基础运行 → 简单任务 → 日志分析 → 问题排查 → 功能测试
```

## 📊 知识点权重分布

| 知识领域 | 重要程度 | 学习时间占比 | 实践比重 |
|---------|----------|-------------|----------|
| 环境搭建 | ⭐⭐⭐⭐⭐ | 25% | 80% |
| 核心概念 | ⭐⭐⭐⭐⭐ | 35% | 40% |
| 项目结构 | ⭐⭐⭐⭐ | 25% | 60% |
| 基础实践 | ⭐⭐⭐⭐ | 15% | 90% |

## 🔄 学习循环

```mermaid
graph LR
    A[理论学习] --> B[实践操作]
    B --> C[问题发现]
    C --> D[解决方案]
    D --> E[知识巩固]
    E --> A
```

## 📈 进度里程碑

### 第1周里程碑
- [ ] 环境搭建完成
- [ ] 项目成功运行
- [ ] 核心概念理解
- [ ] 代码结构熟悉

### 第2周里程碑
- [ ] 完成基础实践项目
- [ ] 能够独立排查常见问题
- [ ] 理解主要模块功能
- [ ] 准备进入下一阶段

## 🎨 可视化学习工具

### 架构图理解
```
用户界面 (Frontend)
    ↓
API网关 (Server)
    ↓
Agent控制器 (Controller)
    ↓
运行时环境 (Runtime)
    ↓
工具执行 (Tools)
```

### 数据流图
```
用户输入 → 消息处理 → Agent思考 → 动作执行 → 结果反馈 → 界面更新
```

## 💡 学习技巧

1. **渐进式学习**：从简单到复杂，逐步深入
2. **实践导向**：每学一个概念就动手实践
3. **问题驱动**：带着问题去学习，效果更好
4. **笔记记录**：记录重要概念和实践心得
5. **社区参与**：积极参与讨论，获取帮助

## 🔗 相关链接

- [官方文档](https://docs.all-hands.dev)
- [GitHub仓库](https://github.com/All-Hands-AI/OpenHands)
- [社区讨论](https://join.slack.com/t/openhands-ai/shared_invite/zt-3847of6xi-xuYJIPa6YIPg4ElbDWbtSA)
- [开发指南](../../Development.md)
