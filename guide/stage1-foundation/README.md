# 阶段一：基础理解（1-2周）

## 🎯 学习目标

在这个阶段，你将：
- 成功搭建 OpenHands 开发环境
- 理解 OpenHands 的核心概念和架构
- 熟悉项目的代码结构
- 掌握基本的使用方法

## 📋 学习清单

### 第1-2天：环境搭建
- [ ] 克隆项目代码
- [ ] 安装系统依赖（Python 3.12+, Node.js, Docker等）
- [ ] 运行 `make build` 构建项目
- [ ] 启动项目并验证功能
- [ ] 配置开发工具（IDE、Git等）

### 第3-4天：核心概念学习
- [ ] 理解 Agent-Action-Observation 模式
- [ ] 学习事件驱动架构
- [ ] 了解运行时系统概念
- [ ] 掌握前后端通信机制

### 第5-7天：代码结构熟悉
- [ ] 浏览后端目录结构（openhands/）
- [ ] 浏览前端目录结构（frontend/）
- [ ] 理解配置文件作用
- [ ] 查看关键文件和模块

### 第8-10天：基础实践
- [ ] 运行示例任务
- [ ] 查看日志和调试信息
- [ ] 尝试修改简单配置
- [ ] 阅读官方文档

### 第11-14天：深入理解
- [ ] 分析一个完整的任务执行流程
- [ ] 理解不同 Agent 的特点
- [ ] 学习工具系统的使用
- [ ] 准备进入下一阶段

## 📚 必读文档

1. **项目根目录**
   - `README.md` - 项目介绍和快速开始
   - `CONTRIBUTING.md` - 贡献指南
   - `Development.md` - 开发指南

2. **核心文档**
   - `openhands/README.md` - 后端架构说明
   - `openhands/core/message_format.md` - 消息格式说明
   - `frontend/README.md` - 前端开发指南

3. **配置文件**
   - `pyproject.toml` - Python 项目配置
   - `frontend/package.json` - 前端项目配置
   - `config.template.toml` - 配置模板

## 🛠️ 实践项目

### 项目1：环境验证
创建一个简单的测试脚本，验证环境搭建是否成功：

```bash
# 创建测试脚本
cat > test_environment.sh << 'EOF'
#!/bin/bash
echo "Testing OpenHands environment..."

# 检查 Python 版本
python --version

# 检查依赖安装
poetry --version
npm --version
docker --version

# 检查项目构建
cd /path/to/OpenHands
make check-dependencies

echo "Environment test completed!"
EOF

chmod +x test_environment.sh
./test_environment.sh
```

### 项目2：代码探索
创建一个代码分析脚本，统计项目的基本信息：

```python
# code_analysis.py
import os
from pathlib import Path

def analyze_project():
    project_root = Path(".")

    # 统计文件类型
    file_types = {}
    total_files = 0

    for file_path in project_root.rglob("*"):
        if file_path.is_file():
            suffix = file_path.suffix or "no_extension"
            file_types[suffix] = file_types.get(suffix, 0) + 1
            total_files += 1

    print(f"Total files: {total_files}")
    print("\nFile types:")
    for ext, count in sorted(file_types.items(), key=lambda x: x[1], reverse=True)[:10]:
        print(f"  {ext}: {count}")

    # 统计代码行数
    python_lines = 0
    ts_lines = 0

    for file_path in project_root.rglob("*.py"):
        try:
            with open(file_path, 'r', encoding='utf-8') as f:
                python_lines += len(f.readlines())
        except:
            pass

    for file_path in project_root.rglob("*.ts"):
        try:
            with open(file_path, 'r', encoding='utf-8') as f:
                ts_lines += len(f.readlines())
        except:
            pass

    print(f"\nPython lines: {python_lines}")
    print(f"TypeScript lines: {ts_lines}")

if __name__ == "__main__":
    analyze_project()
```

## 🔍 关键概念理解

### Agent-Action-Observation 模式
```
用户输入 → Agent思考 → 执行Action → 获得Observation → 继续思考 → ...
```

**深入学习资源：**
- [Agent-Action-Observation模式详解](./agent-action-observation-pattern.md) - 完整的AAO模式解析
- [模块依赖关系图](./module-dependency-diagrams.md) - 按应用场景分类的依赖图谱
- [代码示例详解](./code-examples.md) - 完整的代码实现示例和最佳实践

### 事件驱动架构
- **Event**: 系统中的基本单位
- **Action**: Agent执行的操作
- **Observation**: 操作的结果反馈
- **EventStore**: 事件的存储和管理

### 运行时系统
- **LocalRuntime**: 本地环境执行
- **DockerRuntime**: Docker容器执行
- **RemoteRuntime**: 远程环境执行

## 📊 学习进度跟踪

使用以下表格跟踪你的学习进度：

| 学习内容 | 状态 | 完成日期 | 备注 |
|---------|------|----------|------|
| 环境搭建 | ⏳ | | |
| 核心概念 | ⏳ | | |
| 代码结构 | ⏳ | | |
| 基础实践 | ⏳ | | |
| 深入理解 | ⏳ | | |

状态说明：⏳ 进行中 | ✅ 已完成 | ❌ 遇到问题

## 🤔 常见问题

### Q: 环境搭建失败怎么办？
A:
1. 检查系统依赖是否正确安装
2. 查看错误日志，定位具体问题
3. 参考官方文档的故障排除部分
4. 在社区寻求帮助

### Q: 如何理解复杂的代码结构？
A:
1. 从入口文件开始，逐步深入
2. 使用IDE的代码导航功能
3. 画出模块依赖图
4. 运行代码并观察执行流程

### Q: 学习进度慢怎么办？
A:
1. 调整学习计划，适当延长时间
2. 重点关注核心概念，暂时跳过细节
3. 多做实践项目，加深理解
4. 与其他学习者交流经验

## ➡️ 下一阶段

完成本阶段学习后，你应该：
- 能够独立搭建和运行 OpenHands
- 理解项目的基本架构和核心概念
- 熟悉代码结构和主要模块
- 为深入学习做好准备

准备好了吗？让我们进入 [阶段二：深入理解](../stage2-deep-dive/) 吧！
