# 阶段四：扩展开发（4-6周）

## 🎯 学习目标

在这个阶段，你将：
- 开发自定义工具和Agent扩展
- 创建专业化的微代理系统
- 扩展运行时环境支持
- 开发完整的插件系统
- 掌握系统集成和扩展技术

## 📋 学习清单

### 第1-7天：自定义工具开发
- [ ] 理解工具系统的设计模式
- [ ] 开发基础工具类
- [ ] 实现复杂工具功能
- [ ] 集成外部API和服务
- [ ] 优化工具性能和错误处理

### 第8-14天：微代理开发
- [ ] 深入理解微代理架构
- [ ] 创建领域特定微代理
- [ ] 实现触发机制和上下文管理
- [ ] 开发微代理注册和发现系统
- [ ] 优化微代理性能和可维护性

### 第15-21天：运行时扩展
- [ ] 分析现有运行时架构
- [ ] 设计自定义运行时接口
- [ ] 实现特定环境支持
- [ ] 集成安全和监控机制
- [ ] 测试和优化运行时性能

### 第22-28天：插件系统开发
- [ ] 设计插件架构和接口
- [ ] 实现插件加载和管理
- [ ] 开发插件通信机制
- [ ] 创建插件开发工具
- [ ] 建立插件生态系统

### 第29-42天：系统集成和优化
- [ ] 集成所有扩展组件
- [ ] 优化系统性能和稳定性
- [ ] 实现完整的测试覆盖
- [ ] 编写详细的文档和示例
- [ ] 准备生产环境部署

## 📚 扩展开发核心概念

### 1. 工具系统架构
```
工具系统
├── 工具基类 (BaseTool)
├── 工具注册器 (ToolRegistry)
├── 工具执行器 (ToolExecutor)
├── 工具管理器 (ToolManager)
└── 工具插件 (ToolPlugin)
```

### 2. 微代理系统架构
```
微代理系统
├── 微代理基类 (BaseMicroagent)
├── 触发器系统 (TriggerSystem)
├── 上下文管理 (ContextManager)
├── 知识库集成 (KnowledgeBase)
└── 动态加载器 (DynamicLoader)
```

### 3. 运行时扩展架构
```
运行时扩展
├── 运行时接口 (RuntimeInterface)
├── 环境适配器 (EnvironmentAdapter)
├── 资源管理器 (ResourceManager)
├── 安全控制器 (SecurityController)
└── 监控集成 (MonitoringIntegration)
```

## 🛠️ 实践项目

### 项目1：高级数据分析工具
创建一个集成多种数据分析功能的工具：

```python
# advanced_data_tool.py
from typing import Dict, List, Any, Optional, Union
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report
import plotly.express as px
import plotly.graph_objects as go
from openhands.events.action import Action
from openhands.events.observation import Observation

class AdvancedDataAnalysisTool:
    """高级数据分析工具"""

    def __init__(self):
        self.name = "advanced_data_analysis"
        self.description = "Advanced data analysis and visualization tool"
        self.supported_formats = ['csv', 'json', 'excel', 'parquet']
        self.analysis_cache = {}

    async def execute(self, action: Action) -> Observation:
        """执行数据分析任务"""
        try:
            task_type = action.task_type
            data_source = action.data_source
            parameters = action.parameters or {}

            # 加载数据
            data = await self._load_data(data_source)

            # 根据任务类型执行分析
            if task_type == "exploratory_analysis":
                result = await self._exploratory_analysis(data, parameters)
            elif task_type == "statistical_analysis":
                result = await self._statistical_analysis(data, parameters)
            elif task_type == "machine_learning":
                result = await self._machine_learning_analysis(data, parameters)
            elif task_type == "visualization":
                result = await self._create_visualizations(data, parameters)
            elif task_type == "data_cleaning":
                result = await self._clean_data(data, parameters)
            else:
                raise ValueError(f"Unsupported task type: {task_type}")

            return Observation(
                content=result,
                observation="DataAnalysisObservation",
                success=True
            )

        except Exception as e:
            return Observation(
                content=f"Data analysis failed: {str(e)}",
                observation="ErrorObservation",
                success=False
            )

    async def _load_data(self, data_source: Union[str, Dict]) -> pd.DataFrame:
        """加载数据"""
        if isinstance(data_source, str):
            # 从文件路径加载
            if data_source.endswith('.csv'):
                return pd.read_csv(data_source)
            elif data_source.endswith('.json'):
                return pd.read_json(data_source)
            elif data_source.endswith(('.xlsx', '.xls')):
                return pd.read_excel(data_source)
            elif data_source.endswith('.parquet'):
                return pd.read_parquet(data_source)
            else:
                raise ValueError(f"Unsupported file format: {data_source}")
        elif isinstance(data_source, dict):
            # 从字典创建DataFrame
            return pd.DataFrame(data_source)
        else:
            raise ValueError("Invalid data source format")

    async def _exploratory_analysis(self, data: pd.DataFrame, params: Dict) -> Dict:
        """探索性数据分析"""
        analysis = {
            'basic_info': {
                'shape': data.shape,
                'columns': list(data.columns),
                'dtypes': data.dtypes.to_dict(),
                'memory_usage': data.memory_usage(deep=True).sum()
            },
            'missing_values': data.isnull().sum().to_dict(),
            'summary_statistics': data.describe().to_dict(),
            'unique_values': {col: data[col].nunique() for col in data.columns},
            'correlations': data.corr().to_dict() if data.select_dtypes(include=[np.number]).shape[1] > 1 else {}
        }

        # 检测异常值
        numeric_columns = data.select_dtypes(include=[np.number]).columns
        outliers = {}
        for col in numeric_columns:
            Q1 = data[col].quantile(0.25)
            Q3 = data[col].quantile(0.75)
            IQR = Q3 - Q1
            lower_bound = Q1 - 1.5 * IQR
            upper_bound = Q3 + 1.5 * IQR
            outliers[col] = {
                'count': ((data[col] < lower_bound) | (data[col] > upper_bound)).sum(),
                'percentage': ((data[col] < lower_bound) | (data[col] > upper_bound)).mean() * 100
            }

        analysis['outliers'] = outliers

        return analysis

    async def _statistical_analysis(self, data: pd.DataFrame, params: Dict) -> Dict:
        """统计分析"""
        from scipy import stats

        analysis = {}

        # 假设检验
        if 'hypothesis_test' in params:
            test_type = params['hypothesis_test']['type']
            columns = params['hypothesis_test']['columns']

            if test_type == 't_test' and len(columns) == 2:
                # 独立样本t检验
                group1 = data[columns[0]].dropna()
                group2 = data[columns[1]].dropna()
                t_stat, p_value = stats.ttest_ind(group1, group2)
                analysis['t_test'] = {
                    't_statistic': t_stat,
                    'p_value': p_value,
                    'significant': p_value < 0.05
                }

            elif test_type == 'chi_square' and len(columns) == 2:
                # 卡方检验
                contingency_table = pd.crosstab(data[columns[0]], data[columns[1]])
                chi2, p_value, dof, expected = stats.chi2_contingency(contingency_table)
                analysis['chi_square'] = {
                    'chi2_statistic': chi2,
                    'p_value': p_value,
                    'degrees_of_freedom': dof,
                    'significant': p_value < 0.05
                }

        # 相关性分析
        if 'correlation_analysis' in params:
            method = params['correlation_analysis'].get('method', 'pearson')
            numeric_data = data.select_dtypes(include=[np.number])

            if method == 'pearson':
                corr_matrix = numeric_data.corr(method='pearson')
            elif method == 'spearman':
                corr_matrix = numeric_data.corr(method='spearman')
            else:
                corr_matrix = numeric_data.corr(method='kendall')

            analysis['correlation_matrix'] = corr_matrix.to_dict()

        return analysis

    async def _machine_learning_analysis(self, data: pd.DataFrame, params: Dict) -> Dict:
        """机器学习分析"""
        ml_type = params.get('ml_type', 'classification')
        target_column = params.get('target_column')
        feature_columns = params.get('feature_columns', [])

        if not target_column or target_column not in data.columns:
            raise ValueError("Target column not specified or not found in data")

        # 准备特征和目标变量
        if not feature_columns:
            feature_columns = [col for col in data.columns if col != target_column]

        X = data[feature_columns].select_dtypes(include=[np.number])
        y = data[target_column]

        # 处理缺失值
        X = X.fillna(X.mean())
        y = y.fillna(y.mode()[0] if not y.mode().empty else 0)

        # 分割数据
        X_train, X_test, y_train, y_test = train_test_split(
            X, y, test_size=0.2, random_state=42
        )

        analysis = {}

        if ml_type == 'classification':
            # 分类任务
            model = RandomForestClassifier(n_estimators=100, random_state=42)
            model.fit(X_train, y_train)

            # 预测和评估
            y_pred = model.predict(X_test)

            analysis['classification'] = {
                'accuracy': model.score(X_test, y_test),
                'feature_importance': dict(zip(X.columns, model.feature_importances_)),
                'classification_report': classification_report(y_test, y_pred, output_dict=True)
            }

        elif ml_type == 'regression':
            # 回归任务
            from sklearn.ensemble import RandomForestRegressor
            from sklearn.metrics import mean_squared_error, r2_score

            model = RandomForestRegressor(n_estimators=100, random_state=42)
            model.fit(X_train, y_train)

            y_pred = model.predict(X_test)

            analysis['regression'] = {
                'r2_score': r2_score(y_test, y_pred),
                'mse': mean_squared_error(y_test, y_pred),
                'feature_importance': dict(zip(X.columns, model.feature_importances_))
            }

        return analysis

    async def _create_visualizations(self, data: pd.DataFrame, params: Dict) -> Dict:
        """创建可视化"""
        viz_types = params.get('visualization_types', ['histogram', 'scatter', 'correlation_heatmap'])
        output_format = params.get('output_format', 'html')

        visualizations = {}

        for viz_type in viz_types:
            if viz_type == 'histogram':
                # 直方图
                numeric_columns = data.select_dtypes(include=[np.number]).columns[:5]  # 限制数量
                fig = px.histogram(data, x=numeric_columns[0] if len(numeric_columns) > 0 else data.columns[0])
                visualizations['histogram'] = fig.to_html() if output_format == 'html' else fig.to_json()

            elif viz_type == 'scatter':
                # 散点图
                numeric_columns = data.select_dtypes(include=[np.number]).columns
                if len(numeric_columns) >= 2:
                    fig = px.scatter(data, x=numeric_columns[0], y=numeric_columns[1])
                    visualizations['scatter'] = fig.to_html() if output_format == 'html' else fig.to_json()

            elif viz_type == 'correlation_heatmap':
                # 相关性热力图
                numeric_data = data.select_dtypes(include=[np.number])
                if numeric_data.shape[1] > 1:
                    corr_matrix = numeric_data.corr()
                    fig = px.imshow(corr_matrix, text_auto=True, aspect="auto")
                    visualizations['correlation_heatmap'] = fig.to_html() if output_format == 'html' else fig.to_json()

            elif viz_type == 'box_plot':
                # 箱线图
                numeric_columns = data.select_dtypes(include=[np.number]).columns[:3]
                for col in numeric_columns:
                    fig = px.box(data, y=col)
                    visualizations[f'box_plot_{col}'] = fig.to_html() if output_format == 'html' else fig.to_json()

        return visualizations

    async def _clean_data(self, data: pd.DataFrame, params: Dict) -> Dict:
        """数据清洗"""
        cleaning_operations = params.get('operations', ['remove_duplicates', 'handle_missing', 'remove_outliers'])

        original_shape = data.shape
        cleaned_data = data.copy()
        operations_performed = []

        for operation in cleaning_operations:
            if operation == 'remove_duplicates':
                before_count = len(cleaned_data)
                cleaned_data = cleaned_data.drop_duplicates()
                after_count = len(cleaned_data)
                operations_performed.append({
                    'operation': 'remove_duplicates',
                    'removed_rows': before_count - after_count
                })

            elif operation == 'handle_missing':
                missing_strategy = params.get('missing_strategy', 'drop')
                missing_before = cleaned_data.isnull().sum().sum()

                if missing_strategy == 'drop':
                    cleaned_data = cleaned_data.dropna()
                elif missing_strategy == 'fill_mean':
                    numeric_columns = cleaned_data.select_dtypes(include=[np.number]).columns
                    cleaned_data[numeric_columns] = cleaned_data[numeric_columns].fillna(
                        cleaned_data[numeric_columns].mean()
                    )
                elif missing_strategy == 'fill_mode':
                    for col in cleaned_data.columns:
                        mode_value = cleaned_data[col].mode()
                        if not mode_value.empty:
                            cleaned_data[col] = cleaned_data[col].fillna(mode_value[0])

                missing_after = cleaned_data.isnull().sum().sum()
                operations_performed.append({
                    'operation': 'handle_missing',
                    'strategy': missing_strategy,
                    'missing_values_handled': missing_before - missing_after
                })

            elif operation == 'remove_outliers':
                numeric_columns = cleaned_data.select_dtypes(include=[np.number]).columns
                outliers_removed = 0

                for col in numeric_columns:
                    Q1 = cleaned_data[col].quantile(0.25)
                    Q3 = cleaned_data[col].quantile(0.75)
                    IQR = Q3 - Q1
                    lower_bound = Q1 - 1.5 * IQR
                    upper_bound = Q3 + 1.5 * IQR

                    before_count = len(cleaned_data)
                    cleaned_data = cleaned_data[
                        (cleaned_data[col] >= lower_bound) &
                        (cleaned_data[col] <= upper_bound)
                    ]
                    outliers_removed += before_count - len(cleaned_data)

                operations_performed.append({
                    'operation': 'remove_outliers',
                    'outliers_removed': outliers_removed
                })

        # 保存清洗后的数据
        output_path = params.get('output_path', 'cleaned_data.csv')
        cleaned_data.to_csv(output_path, index=False)

        return {
            'original_shape': original_shape,
            'cleaned_shape': cleaned_data.shape,
            'operations_performed': operations_performed,
            'output_path': output_path,
            'data_quality_score': self._calculate_data_quality_score(cleaned_data)
        }

    def _calculate_data_quality_score(self, data: pd.DataFrame) -> float:
        """计算数据质量分数"""
        # 简单的数据质量评分算法
        completeness = 1 - (data.isnull().sum().sum() / (data.shape[0] * data.shape[1]))
        uniqueness = data.nunique().sum() / (data.shape[0] * data.shape[1])
        consistency = 1.0  # 简化处理，实际应该检查数据一致性

        quality_score = (completeness * 0.4 + uniqueness * 0.3 + consistency * 0.3) * 100
        return round(quality_score, 2)

# 工具注册
def register_advanced_data_tool():
    """注册高级数据分析工具"""
    tool = AdvancedDataAnalysisTool()
    return {
        'name': tool.name,
        'description': tool.description,
        'execute': tool.execute,
        'supported_formats': tool.supported_formats
    }
```

### 项目2：专业化微代理开发
创建一个专门处理代码审查的微代理：

```python
# code_review_microagent.py
from typing import Dict, List, Any, Optional
import ast
import re
from pathlib import Path
from openhands.microagent.microagent import Microagent
from openhands.core.message import Message

class CodeReviewMicroagent(Microagent):
    """代码审查微代理"""

    def __init__(self):
        super().__init__()
        self.name = "code_review_specialist"
        self.description = "Specialized microagent for code review and quality analysis"
        self.triggers = [
            "code review", "review code", "check code quality",
            "analyze code", "code analysis", "refactor suggestion"
        ]
        self.expertise_areas = [
            "code_quality", "security", "performance",
            "maintainability", "best_practices"
        ]
        self.supported_languages = [
            "python", "javascript", "typescript", "java",
            "go", "rust", "cpp", "csharp"
        ]

    async def process_request(self, message: Message, context: Dict) -> Dict:
        """处理代码审查请求"""
        try:
            # 解析请求
            request_info = await self._parse_request(message.content, context)

            # 获取代码
            code_content = await self._extract_code(request_info)

            # 执行代码审查
            review_results = await self._perform_code_review(
                code_content, request_info
            )

            # 生成审查报告
            report = await self._generate_review_report(review_results)

            return {
                'success': True,
                'report': report,
                'recommendations': review_results.get('recommendations', []),
                'metrics': review_results.get('metrics', {}),
                'priority_issues': review_results.get('priority_issues', [])
            }

        except Exception as e:
            return {
                'success': False,
                'error': str(e),
                'fallback_suggestions': await self._get_fallback_suggestions()
            }

    async def _parse_request(self, content: str, context: Dict) -> Dict:
        """解析审查请求"""
        request_info = {
            'review_type': 'comprehensive',  # comprehensive, security, performance
            'language': 'python',  # 默认语言
            'focus_areas': [],
            'code_source': None,
            'file_paths': []
        }

        # 检测编程语言
        language_patterns = {
            'python': r'\.(py|pyw)$|python|def |class |import ',
            'javascript': r'\.(js|jsx)$|javascript|function |const |let ',
            'typescript': r'\.(ts|tsx)$|typescript|interface |type ',
            'java': r'\.(java)$|java|public class |private |protected ',
            'go': r'\.(go)$|golang|func |package |import ',
            'rust': r'\.(rs)$|rust|fn |struct |impl ',
            'cpp': r'\.(cpp|cc|cxx|h|hpp)$|c\+\+|#include|class |namespace ',
            'csharp': r'\.(cs)$|c#|csharp|public class |namespace |using '
        }

        for lang, pattern in language_patterns.items():
            if re.search(pattern, content, re.IGNORECASE):
                request_info['language'] = lang
                break

        # 检测审查类型
        if any(keyword in content.lower() for keyword in ['security', 'vulnerability', 'exploit']):
            request_info['review_type'] = 'security'
            request_info['focus_areas'].append('security')

        if any(keyword in content.lower() for keyword in ['performance', 'optimization', 'speed']):
            request_info['review_type'] = 'performance'
            request_info['focus_areas'].append('performance')

        if any(keyword in content.lower() for keyword in ['maintainability', 'readability', 'clean']):
            request_info['focus_areas'].append('maintainability')

        # 提取文件路径
        file_path_pattern = r'([a-zA-Z0-9_/\\.-]+\.(py|js|ts|java|go|rs|cpp|cs|h|hpp))'
        file_paths = re.findall(file_path_pattern, content)
        request_info['file_paths'] = [path[0] for path in file_paths]

        return request_info

    async def _extract_code(self, request_info: Dict) -> Dict:
        """提取代码内容"""
        code_content = {}

        # 从文件路径读取代码
        for file_path in request_info['file_paths']:
            try:
                path = Path(file_path)
                if path.exists() and path.is_file():
                    with open(path, 'r', encoding='utf-8') as f:
                        code_content[str(path)] = {
                            'content': f.read(),
                            'language': self._detect_language_from_extension(path.suffix),
                            'size': path.stat().st_size
                        }
            except Exception as e:
                print(f"Error reading file {file_path}: {e}")

        return code_content

    async def _perform_code_review(self, code_content: Dict, request_info: Dict) -> Dict:
        """执行代码审查"""
        review_results = {
            'issues': [],
            'recommendations': [],
            'metrics': {},
            'priority_issues': []
        }

        for file_path, file_info in code_content.items():
            content = file_info['content']
            language = file_info['language']

            # 基于语言的特定审查
            if language == 'python':
                file_results = await self._review_python_code(content, file_path)
            elif language in ['javascript', 'typescript']:
                file_results = await self._review_js_ts_code(content, file_path)
            else:
                file_results = await self._review_generic_code(content, file_path)

            # 合并结果
            review_results['issues'].extend(file_results.get('issues', []))
            review_results['recommendations'].extend(file_results.get('recommendations', []))
            review_results['metrics'][file_path] = file_results.get('metrics', {})

        # 识别优先级问题
        review_results['priority_issues'] = await self._identify_priority_issues(
            review_results['issues']
        )

        return review_results

    async def _review_python_code(self, content: str, file_path: str) -> Dict:
        """审查Python代码"""
        issues = []
        recommendations = []
        metrics = {}

        try:
            # 解析AST
            tree = ast.parse(content)

            # 代码复杂度分析
            complexity = self._calculate_cyclomatic_complexity(tree)
            metrics['cyclomatic_complexity'] = complexity

            if complexity > 10:
                issues.append({
                    'type': 'complexity',
                    'severity': 'high',
                    'message': f'High cyclomatic complexity: {complexity}',
                    'file': file_path,
                    'suggestion': 'Consider breaking down complex functions into smaller ones'
                })

            # 检查函数长度
            function_lengths = self._analyze_function_lengths(tree)
            for func_name, length in function_lengths.items():
                if length > 50:
                    issues.append({
                        'type': 'maintainability',
                        'severity': 'medium',
                        'message': f'Function {func_name} is too long ({length} lines)',
                        'file': file_path,
                        'suggestion': 'Consider breaking down the function into smaller functions'
                    })

            # 检查导入语句
            import_issues = self._check_imports(tree)
            issues.extend(import_issues)

            # 检查命名规范
            naming_issues = self._check_naming_conventions(tree)
            issues.extend(naming_issues)

            # 安全检查
            security_issues = self._check_security_issues(content)
            issues.extend(security_issues)

            # 性能检查
            performance_issues = self._check_performance_issues(tree, content)
            issues.extend(performance_issues)

        except SyntaxError as e:
            issues.append({
                'type': 'syntax',
                'severity': 'critical',
                'message': f'Syntax error: {str(e)}',
                'file': file_path,
                'line': e.lineno
            })

        # 生成建议
        recommendations.extend(self._generate_python_recommendations(issues))

        return {
            'issues': issues,
            'recommendations': recommendations,
            'metrics': metrics
        }

    def _calculate_cyclomatic_complexity(self, tree: ast.AST) -> int:
        """计算圈复杂度"""
        complexity = 1  # 基础复杂度

        for node in ast.walk(tree):
            if isinstance(node, (ast.If, ast.While, ast.For, ast.AsyncFor)):
                complexity += 1
            elif isinstance(node, ast.ExceptHandler):
                complexity += 1
            elif isinstance(node, (ast.And, ast.Or)):
                complexity += 1

        return complexity

    def _analyze_function_lengths(self, tree: ast.AST) -> Dict[str, int]:
        """分析函数长度"""
        function_lengths = {}

        for node in ast.walk(tree):
            if isinstance(node, (ast.FunctionDef, ast.AsyncFunctionDef)):
                # 计算函数行数
                if hasattr(node, 'end_lineno') and hasattr(node, 'lineno'):
                    length = node.end_lineno - node.lineno + 1
                    function_lengths[node.name] = length

        return function_lengths

    def _check_imports(self, tree: ast.AST) -> List[Dict]:
        """检查导入语句"""
        issues = []
        imports = []

        for node in ast.walk(tree):
            if isinstance(node, ast.Import):
                for alias in node.names:
                    imports.append(alias.name)
            elif isinstance(node, ast.ImportFrom):
                if node.module:
                    imports.append(node.module)

        # 检查未使用的导入（简化版）
        # 实际实现需要更复杂的分析

        # 检查危险的导入
        dangerous_imports = ['os', 'subprocess', 'eval', 'exec']
        for imp in imports:
            if imp in dangerous_imports:
                issues.append({
                    'type': 'security',
                    'severity': 'medium',
                    'message': f'Potentially dangerous import: {imp}',
                    'suggestion': 'Review the usage of this import for security implications'
                })

        return issues

    def _check_naming_conventions(self, tree: ast.AST) -> List[Dict]:
        """检查命名规范"""
        issues = []

        for node in ast.walk(tree):
            if isinstance(node, ast.FunctionDef):
                if not re.match(r'^[a-z_][a-z0-9_]*$', node.name):
                    issues.append({
                        'type': 'style',
                        'severity': 'low',
                        'message': f'Function name {node.name} does not follow snake_case convention',
                        'suggestion': 'Use snake_case for function names'
                    })

            elif isinstance(node, ast.ClassDef):
                if not re.match(r'^[A-Z][a-zA-Z0-9]*$', node.name):
                    issues.append({
                        'type': 'style',
                        'severity': 'low',
                        'message': f'Class name {node.name} does not follow PascalCase convention',
                        'suggestion': 'Use PascalCase for class names'
                    })

        return issues

    def _check_security_issues(self, content: str) -> List[Dict]:
        """检查安全问题"""
        issues = []

        # 检查SQL注入风险
        sql_patterns = [
            r'execute\s*\(\s*["\'].*%.*["\']',
            r'cursor\.execute\s*\(\s*["\'].*\+.*["\']',
            r'query\s*=.*\+.*'
        ]

        for pattern in sql_patterns:
            if re.search(pattern, content, re.IGNORECASE):
                issues.append({
                    'type': 'security',
                    'severity': 'high',
                    'message': 'Potential SQL injection vulnerability',
                    'suggestion': 'Use parameterized queries instead of string concatenation'
                })

        # 检查硬编码密码
        password_patterns = [
            r'password\s*=\s*["\'][^"\']+["\']',
            r'pwd\s*=\s*["\'][^"\']+["\']',
            r'secret\s*=\s*["\'][^"\']+["\']'
        ]

        for pattern in password_patterns:
            if re.search(pattern, content, re.IGNORECASE):
                issues.append({
                    'type': 'security',
                    'severity': 'high',
                    'message': 'Hardcoded password or secret detected',
                    'suggestion': 'Use environment variables or secure configuration files'
                })

        return issues

    def _check_performance_issues(self, tree: ast.AST, content: str) -> List[Dict]:
        """检查性能问题"""
        issues = []

        # 检查循环中的字符串拼接
        for node in ast.walk(tree):
            if isinstance(node, (ast.For, ast.While)):
                for child in ast.walk(node):
                    if isinstance(child, ast.AugAssign) and isinstance(child.op, ast.Add):
                        if isinstance(child.target, ast.Name):
                            issues.append({
                                'type': 'performance',
                                'severity': 'medium',
                                'message': 'String concatenation in loop detected',
                                'suggestion': 'Use list.append() and join() for better performance'
                            })

        # 检查全局变量的使用
        global_vars = []
        for node in ast.walk(tree):
            if isinstance(node, ast.Global):
                global_vars.extend(node.names)

        if global_vars:
            issues.append({
                'type': 'maintainability',
                'severity': 'medium',
                'message': f'Global variables used: {", ".join(global_vars)}',
                'suggestion': 'Consider using function parameters or class attributes instead'
            })

        return issues

    def _generate_python_recommendations(self, issues: List[Dict]) -> List[str]:
        """生成Python特定的建议"""
        recommendations = []

        issue_types = [issue['type'] for issue in issues]

        if 'complexity' in issue_types:
            recommendations.append(
                "Consider using design patterns like Strategy or Command to reduce complexity"
            )

        if 'security' in issue_types:
            recommendations.append(
                "Run security linters like bandit to identify additional security issues"
            )

        if 'performance' in issue_types:
            recommendations.append(
                "Profile your code with cProfile to identify performance bottlenecks"
            )

        if 'style' in issue_types:
            recommendations.append(
                "Use tools like black and flake8 to automatically format and check code style"
            )

        return recommendations

    async def _identify_priority_issues(self, issues: List[Dict]) -> List[Dict]:
        """识别优先级问题"""
        priority_issues = []

        # 按严重程度排序
        severity_order = {'critical': 0, 'high': 1, 'medium': 2, 'low': 3}
        sorted_issues = sorted(issues, key=lambda x: severity_order.get(x.get('severity', 'low'), 3))

        # 取前5个最严重的问题
        priority_issues = sorted_issues[:5]

        return priority_issues

    async def _generate_review_report(self, review_results: Dict) -> str:
        """生成审查报告"""
        issues = review_results.get('issues', [])
        recommendations = review_results.get('recommendations', [])
        metrics = review_results.get('metrics', {})
        priority_issues = review_results.get('priority_issues', [])

        report = "# Code Review Report\n\n"

        # 总结
        report += f"## Summary\n"
        report += f"- Total issues found: {len(issues)}\n"
        report += f"- Critical issues: {len([i for i in issues if i.get('severity') == 'critical'])}\n"
        report += f"- High priority issues: {len([i for i in issues if i.get('severity') == 'high'])}\n"
        report += f"- Medium priority issues: {len([i for i in issues if i.get('severity') == 'medium'])}\n"
        report += f"- Low priority issues: {len([i for i in issues if i.get('severity') == 'low'])}\n\n"

        # 优先级问题
        if priority_issues:
            report += "## Priority Issues\n\n"
            for i, issue in enumerate(priority_issues, 1):
                report += f"{i}. **{issue.get('type', 'Unknown').title()}** ({issue.get('severity', 'unknown')})\n"
                report += f"   - {issue.get('message', 'No message')}\n"
                if issue.get('suggestion'):
                    report += f"   - Suggestion: {issue['suggestion']}\n"
                report += "\n"

        # 指标
        if metrics:
            report += "## Code Metrics\n\n"
            for file_path, file_metrics in metrics.items():
                report += f"### {file_path}\n"
                for metric, value in file_metrics.items():
                    report += f"- {metric.replace('_', ' ').title()}: {value}\n"
                report += "\n"

        # 建议
        if recommendations:
            report += "## Recommendations\n\n"
            for i, rec in enumerate(recommendations, 1):
                report += f"{i}. {rec}\n"
            report += "\n"

        return report

# 微代理注册
def register_code_review_microagent():
    """注册代码审查微代理"""
    microagent = CodeReviewMicroagent()
    return {
        'name': microagent.name,
        'description': microagent.description,
        'triggers': microagent.triggers,
        'process_request': microagent.process_request,
        'expertise_areas': microagent.expertise_areas
    }
```

## 📊 学习进度跟踪

| 扩展领域 | 设计阶段 | 开发阶段 | 测试阶段 | 集成阶段 | 完成度 |
|---------|----------|----------|----------|----------|--------|
| 自定义工具 | ⏳ | ⏳ | ⏳ | ⏳ | 0% |
| 微代理系统 | ⏳ | ⏳ | ⏳ | ⏳ | 0% |
| 运行时扩展 | ⏳ | ⏳ | ⏳ | ⏳ | 0% |
| 插件系统 | ⏳ | ⏳ | ⏳ | ⏳ | 0% |
| 系统集成 | ⏳ | ⏳ | ⏳ | ⏳ | 0% |

## 🎯 扩展开发最佳实践

### 1. 工具开发原则
- **单一职责**：每个工具专注于一个特定功能
- **接口一致**：遵循统一的工具接口规范
- **错误处理**：完善的错误处理和恢复机制
- **性能优化**：考虑工具的执行效率和资源使用

### 2. 微代理设计原则
- **领域专业化**：专注于特定领域的专业知识
- **上下文感知**：能够理解和利用上下文信息
- **动态适应**：根据任务需求动态调整行为
- **知识更新**：支持知识库的持续更新

### 3. 运行时扩展原则
- **安全隔离**：确保运行时环境的安全性
- **资源管理**：有效管理计算资源和内存
- **可扩展性**：支持水平和垂直扩展
- **监控集成**：内置监控和日志功能

## ➡️ 下一阶段

完成本阶段学习后，你应该：
- 能够开发复杂的系统扩展
- 掌握插件化架构设计
- 具备系统集成能力
- 为生产部署做好准备

准备好了吗？让我们进入 [阶段五：生产部署](../stage5-production/) 吧！
