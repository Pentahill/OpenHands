# 阶段五：生产部署（1-2周）

## 🎯 学习目标

在这个阶段，你将：
- 掌握生产环境部署配置
- 实现完整的监控和日志系统
- 配置安全防护和访问控制
- 优化系统性能和可靠性
- 建立运维和故障处理流程

## 📋 学习清单

### 第1-3天：部署配置
- [ ] 设计生产环境架构
- [ ] 配置容器化部署
- [ ] 设置负载均衡和反向代理
- [ ] 配置数据库和缓存
- [ ] 实现配置管理和环境变量

### 第4-6天：监控和日志
- [ ] 部署监控系统（Prometheus + Grafana）
- [ ] 配置日志收集和分析
- [ ] 设置告警和通知机制
- [ ] 实现健康检查和探针
- [ ] 建立性能基线和SLA

### 第7-9天：安全配置
- [ ] 配置HTTPS和SSL证书
- [ ] 实现身份认证和授权
- [ ] 设置防火墙和网络安全
- [ ] 配置数据加密和备份
- [ ] 实施安全审计和合规

### 第10-12天：性能优化
- [ ] 分析和优化系统瓶颈
- [ ] 配置缓存策略
- [ ] 实现数据库优化
- [ ] 设置CDN和静态资源优化
- [ ] 进行压力测试和调优

### 第13-14天：运维流程
- [ ] 建立CI/CD流水线
- [ ] 制定故障处理流程
- [ ] 实现自动化运维脚本
- [ ] 编写运维文档和手册
- [ ] 进行灾难恢复演练

## 📚 生产部署架构

### 1. 整体架构图
```
                    ┌─────────────────┐
                    │   Load Balancer │
                    │    (Nginx)      │
                    └─────────┬───────┘
                              │
                    ┌─────────┴───────┐
                    │                 │
            ┌───────▼────────┐ ┌──────▼────────┐
            │  Frontend       │ │   Backend     │
            │  (React App)    │ │  (FastAPI)    │
            └────────────────┘ └───────┬───────┘
                                       │
                              ┌────────┴────────┐
                              │                 │
                    ┌─────────▼────────┐ ┌──────▼────────┐
                    │    Database      │ │     Cache     │
                    │  (PostgreSQL)    │ │    (Redis)    │
                    └──────────────────┘ └───────────────┘
```

### 2. 容器化架构
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - backend

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    environment:
      - NODE_ENV=production
    volumes:
      - frontend_build:/app/build

  backend:
    build:
      context: .
      dockerfile: Dockerfile.prod
    environment:
      - ENVIRONMENT=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    depends_on:
      - postgres
      - redis
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 1G
          cpus: '0.5'

  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backups:/backups

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
  frontend_build:
```

## 🛠️ 实践项目

### 项目1：完整的监控系统
创建一个全面的监控和告警系统：

```python
# monitoring_system.py
import asyncio
import time
import psutil
import aioredis
from typing import Dict, List, Any, Optional
from dataclasses import dataclass
from enum import Enum
import json
import logging
from datetime import datetime, timedelta

class MetricType(Enum):
    COUNTER = "counter"
    GAUGE = "gauge"
    HISTOGRAM = "histogram"
    SUMMARY = "summary"

@dataclass
class Metric:
    name: str
    value: float
    metric_type: MetricType
    labels: Dict[str, str]
    timestamp: datetime

class MetricsCollector:
    """指标收集器"""

    def __init__(self, redis_url: str = "redis://localhost:6379"):
        self.redis_url = redis_url
        self.redis = None
        self.metrics_buffer: List[Metric] = []
        self.collection_interval = 10  # 秒
        self.buffer_size = 1000

    async def initialize(self):
        """初始化连接"""
        self.redis = await aioredis.from_url(self.redis_url)

    async def collect_system_metrics(self):
        """收集系统指标"""
        now = datetime.now()

        # CPU使用率
        cpu_percent = psutil.cpu_percent(interval=1)
        await self.record_metric(Metric(
            name="system_cpu_usage_percent",
            value=cpu_percent,
            metric_type=MetricType.GAUGE,
            labels={"host": "localhost"},
            timestamp=now
        ))

        # 内存使用
        memory = psutil.virtual_memory()
        await self.record_metric(Metric(
            name="system_memory_usage_bytes",
            value=memory.used,
            metric_type=MetricType.GAUGE,
            labels={"host": "localhost"},
            timestamp=now
        ))

        await self.record_metric(Metric(
            name="system_memory_usage_percent",
            value=memory.percent,
            metric_type=MetricType.GAUGE,
            labels={"host": "localhost"},
            timestamp=now
        ))

        # 磁盘使用
        disk = psutil.disk_usage('/')
        await self.record_metric(Metric(
            name="system_disk_usage_bytes",
            value=disk.used,
            metric_type=MetricType.GAUGE,
            labels={"host": "localhost", "mount": "/"},
            timestamp=now
        ))

        # 网络IO
        network = psutil.net_io_counters()
        await self.record_metric(Metric(
            name="system_network_bytes_sent",
            value=network.bytes_sent,
            metric_type=MetricType.COUNTER,
            labels={"host": "localhost"},
            timestamp=now
        ))

        await self.record_metric(Metric(
            name="system_network_bytes_recv",
            value=network.bytes_recv,
            metric_type=MetricType.COUNTER,
            labels={"host": "localhost"},
            timestamp=now
        ))

    async def collect_application_metrics(self, app_stats: Dict[str, Any]):
        """收集应用指标"""
        now = datetime.now()

        # 请求计数
        if 'request_count' in app_stats:
            await self.record_metric(Metric(
                name="app_requests_total",
                value=app_stats['request_count'],
                metric_type=MetricType.COUNTER,
                labels={"app": "openhands"},
                timestamp=now
            ))

        # 响应时间
        if 'avg_response_time' in app_stats:
            await self.record_metric(Metric(
                name="app_response_time_seconds",
                value=app_stats['avg_response_time'],
                metric_type=MetricType.GAUGE,
                labels={"app": "openhands"},
                timestamp=now
            ))

        # 错误率
        if 'error_rate' in app_stats:
            await self.record_metric(Metric(
                name="app_error_rate_percent",
                value=app_stats['error_rate'],
                metric_type=MetricType.GAUGE,
                labels={"app": "openhands"},
                timestamp=now
            ))

        # 活跃连接数
        if 'active_connections' in app_stats:
            await self.record_metric(Metric(
                name="app_active_connections",
                value=app_stats['active_connections'],
                metric_type=MetricType.GAUGE,
                labels={"app": "openhands"},
                timestamp=now
            ))

    async def record_metric(self, metric: Metric):
        """记录指标"""
        self.metrics_buffer.append(metric)

        # 当缓冲区满时，批量写入
        if len(self.metrics_buffer) >= self.buffer_size:
            await self.flush_metrics()

    async def flush_metrics(self):
        """刷新指标到存储"""
        if not self.metrics_buffer:
            return

        # 转换为Prometheus格式
        prometheus_metrics = []
        for metric in self.metrics_buffer:
            labels_str = ','.join([f'{k}="{v}"' for k, v in metric.labels.items()])
            metric_line = f'{metric.name}{{{labels_str}}} {metric.value} {int(metric.timestamp.timestamp() * 1000)}'
            prometheus_metrics.append(metric_line)

        # 存储到Redis
        await self.redis.lpush("metrics", *prometheus_metrics)
        await self.redis.expire("metrics", 86400)  # 24小时过期

        # 清空缓冲区
        self.metrics_buffer.clear()

    async def start_collection(self):
        """开始指标收集"""
        while True:
            try:
                await self.collect_system_metrics()

                # 这里可以添加应用特定的指标收集
                # app_stats = await get_application_stats()
                # await self.collect_application_metrics(app_stats)

                await asyncio.sleep(self.collection_interval)
            except Exception as e:
                logging.error(f"Error collecting metrics: {e}")
                await asyncio.sleep(5)

class AlertManager:
    """告警管理器"""

    def __init__(self, redis_url: str = "redis://localhost:6379"):
        self.redis_url = redis_url
        self.redis = None
        self.alert_rules: List[AlertRule] = []
        self.notification_channels: List[NotificationChannel] = []

    async def initialize(self):
        """初始化"""
        self.redis = await aioredis.from_url(self.redis_url)
        await self.load_alert_rules()
        await self.load_notification_channels()

    async def load_alert_rules(self):
        """加载告警规则"""
        # CPU使用率告警
        self.alert_rules.append(AlertRule(
            name="high_cpu_usage",
            condition="system_cpu_usage_percent > 80",
            severity="warning",
            duration=300,  # 5分钟
            message="CPU usage is above 80% for 5 minutes"
        ))

        # 内存使用率告警
        self.alert_rules.append(AlertRule(
            name="high_memory_usage",
            condition="system_memory_usage_percent > 90",
            severity="critical",
            duration=60,  # 1分钟
            message="Memory usage is above 90%"
        ))

        # 磁盘使用率告警
        self.alert_rules.append(AlertRule(
            name="high_disk_usage",
            condition="system_disk_usage_percent > 85",
            severity="warning",
            duration=600,  # 10分钟
            message="Disk usage is above 85%"
        ))

        # 应用错误率告警
        self.alert_rules.append(AlertRule(
            name="high_error_rate",
            condition="app_error_rate_percent > 5",
            severity="critical",
            duration=120,  # 2分钟
            message="Application error rate is above 5%"
        ))

    async def load_notification_channels(self):
        """加载通知渠道"""
        # 邮件通知
        self.notification_channels.append(EmailNotificationChannel(
            smtp_server="smtp.gmail.com",
            smtp_port=587,
            username="alerts@company.com",
            password="password",
            recipients=["admin@company.com", "ops@company.com"]
        ))

        # Slack通知
        self.notification_channels.append(SlackNotificationChannel(
            webhook_url="https://hooks.slack.com/services/...",
            channel="#alerts"
        ))

    async def evaluate_alerts(self):
        """评估告警规则"""
        current_time = datetime.now()

        for rule in self.alert_rules:
            try:
                # 获取相关指标
                metrics = await self.get_metrics_for_rule(rule)

                # 评估条件
                if await self.evaluate_condition(rule.condition, metrics):
                    # 检查是否已经在告警状态
                    alert_key = f"alert:{rule.name}"
                    alert_start = await self.redis.get(alert_key)

                    if alert_start:
                        # 检查持续时间
                        start_time = datetime.fromisoformat(alert_start.decode())
                        if (current_time - start_time).total_seconds() >= rule.duration:
                            # 触发告警
                            await self.trigger_alert(rule, start_time)
                    else:
                        # 开始新的告警计时
                        await self.redis.set(alert_key, current_time.isoformat(), ex=rule.duration + 60)
                else:
                    # 条件不满足，清除告警状态
                    alert_key = f"alert:{rule.name}"
                    await self.redis.delete(alert_key)

                    # 如果之前有告警，发送恢复通知
                    recovery_key = f"alert_fired:{rule.name}"
                    if await self.redis.get(recovery_key):
                        await self.send_recovery_notification(rule)
                        await self.redis.delete(recovery_key)

            except Exception as e:
                logging.error(f"Error evaluating alert rule {rule.name}: {e}")

    async def trigger_alert(self, rule: AlertRule, start_time: datetime):
        """触发告警"""
        # 检查是否已经发送过告警
        fired_key = f"alert_fired:{rule.name}"
        if await self.redis.get(fired_key):
            return

        # 标记告警已发送
        await self.redis.set(fired_key, "true", ex=3600)  # 1小时内不重复发送

        # 创建告警消息
        alert = Alert(
            name=rule.name,
            severity=rule.severity,
            message=rule.message,
            start_time=start_time,
            labels={"rule": rule.name}
        )

        # 发送通知
        for channel in self.notification_channels:
            try:
                await channel.send_alert(alert)
            except Exception as e:
                logging.error(f"Failed to send alert via {channel.__class__.__name__}: {e}")

    async def start_monitoring(self):
        """开始监控"""
        while True:
            try:
                await self.evaluate_alerts()
                await asyncio.sleep(30)  # 每30秒检查一次
            except Exception as e:
                logging.error(f"Error in alert monitoring: {e}")
                await asyncio.sleep(10)

@dataclass
class AlertRule:
    name: str
    condition: str
    severity: str
    duration: int  # 秒
    message: str

@dataclass
class Alert:
    name: str
    severity: str
    message: str
    start_time: datetime
    labels: Dict[str, str]

class NotificationChannel:
    """通知渠道基类"""

    async def send_alert(self, alert: Alert):
        raise NotImplementedError

class EmailNotificationChannel(NotificationChannel):
    """邮件通知渠道"""

    def __init__(self, smtp_server: str, smtp_port: int, username: str, password: str, recipients: List[str]):
        self.smtp_server = smtp_server
        self.smtp_port = smtp_port
        self.username = username
        self.password = password
        self.recipients = recipients

    async def send_alert(self, alert: Alert):
        import smtplib
        from email.mime.text import MIMEText
        from email.mime.multipart import MIMEMultipart

        msg = MIMEMultipart()
        msg['From'] = self.username
        msg['To'] = ', '.join(self.recipients)
        msg['Subject'] = f"[{alert.severity.upper()}] {alert.name}"

        body = f"""
        Alert: {alert.name}
        Severity: {alert.severity}
        Message: {alert.message}
        Start Time: {alert.start_time}
        Labels: {alert.labels}
        """

        msg.attach(MIMEText(body, 'plain'))

        server = smtplib.SMTP(self.smtp_server, self.smtp_port)
        server.starttls()
        server.login(self.username, self.password)
        server.send_message(msg)
        server.quit()

class SlackNotificationChannel(NotificationChannel):
    """Slack通知渠道"""

    def __init__(self, webhook_url: str, channel: str):
        self.webhook_url = webhook_url
        self.channel = channel

    async def send_alert(self, alert: Alert):
        import aiohttp

        color = {
            'critical': 'danger',
            'warning': 'warning',
            'info': 'good'
        }.get(alert.severity, 'warning')

        payload = {
            'channel': self.channel,
            'username': 'AlertBot',
            'attachments': [{
                'color': color,
                'title': f'{alert.severity.upper()}: {alert.name}',
                'text': alert.message,
                'fields': [
                    {'title': 'Start Time', 'value': str(alert.start_time), 'short': True},
                    {'title': 'Labels', 'value': str(alert.labels), 'short': True}
                ],
                'ts': int(alert.start_time.timestamp())
            }]
        }

        async with aiohttp.ClientSession() as session:
            async with session.post(self.webhook_url, json=payload) as response:
                if response.status != 200:
                    raise Exception(f"Failed to send Slack notification: {response.status}")

# 主监控服务
class MonitoringService:
    """监控服务"""

    def __init__(self):
        self.metrics_collector = MetricsCollector()
        self.alert_manager = AlertManager()

    async def start(self):
        """启动监控服务"""
        await self.metrics_collector.initialize()
        await self.alert_manager.initialize()

        # 启动指标收集和告警监控
        tasks = [
            asyncio.create_task(self.metrics_collector.start_collection()),
            asyncio.create_task(self.alert_manager.start_monitoring())
        ]

        await asyncio.gather(*tasks)

# 使用示例
async def main():
    monitoring = MonitoringService()
    await monitoring.start()

if __name__ == "__main__":
    asyncio.run(main())
```

### 项目2：自动化部署系统
创建一个完整的CI/CD流水线：

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
    - uses: actions/checkout@v4

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.12'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install poetry
        poetry install

    - name: Run tests
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
        REDIS_URL: redis://localhost:6379
      run: |
        poetry run pytest tests/ -v --cov=openhands --cov-report=xml

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.xml

    - name: Run security scan
      run: |
        poetry run bandit -r openhands/ -f json -o bandit-report.json

    - name: Frontend tests
      run: |
        cd frontend
        npm ci
        npm run test
        npm run build

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    permissions:
      contents: read
      packages: write

    steps:
    - uses: actions/checkout@v4

    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}

    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        file: ./Dockerfile.prod
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    environment:
      name: production
      url: https://openhands.example.com

    steps:
    - uses: actions/checkout@v4

    - name: Deploy to production
      uses: appleboy/ssh-action@v1.0.0
      with:
        host: ${{ secrets.PROD_HOST }}
        username: ${{ secrets.PROD_USER }}
        key: ${{ secrets.PROD_SSH_KEY }}
        script: |
          cd /opt/openhands

          # 备份当前版本
          docker-compose -f docker-compose.prod.yml down
          docker tag ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:backup

          # 拉取新镜像
          docker pull ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest

          # 更新配置
          git pull origin main

          # 运行数据库迁移
          docker-compose -f docker-compose.prod.yml run --rm backend python -m alembic upgrade head

          # 启动新版本
          docker-compose -f docker-compose.prod.yml up -d

          # 健康检查
          sleep 30
          curl -f http://localhost/health || exit 1

          # 清理旧镜像
          docker image prune -f

    - name: Notify deployment
      if: always()
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        channel: '#deployments'
        webhook_url: ${{ secrets.SLACK_WEBHOOK }}
        fields: repo,message,commit,author,action,eventName,ref,workflow
```

### 项目3：安全配置和合规
创建一个全面的安全配置系统：

```python
# security_config.py
import os
import ssl
import hashlib
import secrets
from typing import Dict, List, Optional, Any
from dataclasses import dataclass
from enum import Enum
import jwt
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
import base64

class SecurityLevel(Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"
    CRITICAL = "critical"

@dataclass
class SecurityPolicy:
    name: str
    level: SecurityLevel
    rules: List[str]
    enforcement: bool = True

class SecurityManager:
    """安全管理器"""

    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.encryption_key = self._generate_or_load_key()
        self.cipher_suite = Fernet(self.encryption_key)
        self.security_policies: List[SecurityPolicy] = []
        self._load_security_policies()

    def _generate_or_load_key(self) -> bytes:
        """生成或加载加密密钥"""
        key_file = self.config.get('encryption_key_file', '.encryption_key')

        if os.path.exists(key_file):
            with open(key_file, 'rb') as f:
                return f.read()
        else:
            key = Fernet.generate_key()
            with open(key_file, 'wb') as f:
                f.write(key)
            os.chmod(key_file, 0o600)  # 只有所有者可读写
            return key

    def _load_security_policies(self):
        """加载安全策略"""
        # 密码策略
        self.security_policies.append(SecurityPolicy(
            name="password_policy",
            level=SecurityLevel.HIGH,
            rules=[
                "minimum_length:12",
                "require_uppercase:true",
                "require_lowercase:true",
                "require_numbers:true",
                "require_special_chars:true",
                "max_age_days:90",
                "history_count:5"
            ]
        ))

        # 会话策略
        self.security_policies.append(SecurityPolicy(
            name="session_policy",
            level=SecurityLevel.HIGH,
            rules=[
                "max_duration:3600",  # 1小时
                "idle_timeout:1800",  # 30分钟
                "secure_cookies:true",
                "httponly_cookies:true",
                "samesite:strict"
            ]
        ))

        # API访问策略
        self.security_policies.append(SecurityPolicy(
            name="api_access_policy",
            level=SecurityLevel.MEDIUM,
            rules=[
                "rate_limit:100/minute",
                "require_authentication:true",
                "require_https:true",
                "log_all_requests:true"
            ]
        ))

        # 数据保护策略
        self.security_policies.append(SecurityPolicy(
            name="data_protection_policy",
            level=SecurityLevel.CRITICAL,
            rules=[
                "encrypt_at_rest:true",
                "encrypt_in_transit:true",
                "backup_encryption:true",
                "pii_masking:true",
                "audit_data_access:true"
            ]
        ))

    def encrypt_data(self, data: str) -> str:
        """加密数据"""
        return self.cipher_suite.encrypt(data.encode()).decode()

    def decrypt_data(self, encrypted_data: str) -> str:
        """解密数据"""
        return self.cipher_suite.decrypt(encrypted_data.encode()).decode()

    def hash_password(self, password: str, salt: Optional[str] = None) -> tuple[str, str]:
        """哈希密码"""
        if salt is None:
            salt = secrets.token_hex(32)

        # 使用PBKDF2进行密码哈希
        kdf = PBKDF2HMAC(
            algorithm=hashes.SHA256(),
            length=32,
            salt=salt.encode(),
            iterations=100000,
        )
        key = base64.urlsafe_b64encode(kdf.derive(password.encode()))
        return key.decode(), salt

    def verify_password(self, password: str, hashed_password: str, salt: str) -> bool:
        """验证密码"""
        key, _ = self.hash_password(password, salt)
        return secrets.compare_digest(key, hashed_password)

    def generate_jwt_token(self, payload: Dict[str, Any], expires_in: int = 3600) -> str:
        """生成JWT令牌"""
        import time

        payload.update({
            'iat': int(time.time()),
            'exp': int(time.time()) + expires_in
        })

        secret_key = self.config.get('jwt_secret_key', 'default-secret-key')
        return jwt.encode(payload, secret_key, algorithm='HS256')

    def verify_jwt_token(self, token: str) -> Optional[Dict[str, Any]]:
        """验证JWT令牌"""
        try:
            secret_key = self.config.get('jwt_secret_key', 'default-secret-key')
            payload = jwt.decode(token, secret_key, algorithms=['HS256'])
            return payload
        except jwt.ExpiredSignatureError:
            return None
        except jwt.InvalidTokenError:
            return None

    def validate_input(self, input_data: str, input_type: str) -> bool:
        """输入验证"""
        import re

        validation_rules = {
            'email': r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$',
            'username': r'^[a-zA-Z0-9_]{3,20}$',
            'filename': r'^[a-zA-Z0-9._-]+$',
            'url': r'^https?://[^\s/$.?#].[^\s]*$'
        }

        if input_type in validation_rules:
            return bool(re.match(validation_rules[input_type], input_data))

        return True

    def sanitize_input(self, input_data: str) -> str:
        """输入清理"""
        import html

        # HTML转义
        sanitized = html.escape(input_data)

        # 移除潜在的SQL注入字符
        dangerous_chars = ["'", '"', ';', '--', '/*', '*/', 'xp_', 'sp_']
        for char in dangerous_chars:
            sanitized = sanitized.replace(char, '')

        return sanitized

    def check_security_compliance(self) -> Dict[str, Any]:
        """检查安全合规性"""
        compliance_report = {
            'overall_score': 0,
            'policies_checked': 0,
            'policies_compliant': 0,
            'violations': [],
            'recommendations': []
        }

        for policy in self.security_policies:
            compliance_report['policies_checked'] += 1

            # 检查策略合规性
            violations = self._check_policy_compliance(policy)

            if not violations:
                compliance_report['policies_compliant'] += 1
            else:
                compliance_report['violations'].extend(violations)

        # 计算合规分数
        if compliance_report['policies_checked'] > 0:
            compliance_report['overall_score'] = (
                compliance_report['policies_compliant'] /
                compliance_report['policies_checked'] * 100
            )

        # 生成建议
        compliance_report['recommendations'] = self._generate_security_recommendations(
            compliance_report['violations']
        )

        return compliance_report

    def _check_policy_compliance(self, policy: SecurityPolicy) -> List[str]:
        """检查单个策略的合规性"""
        violations = []

        for rule in policy.rules:
            if not self._check_rule_compliance(rule, policy):
                violations.append(f"Policy '{policy.name}' rule '{rule}' violated")

        return violations

    def _check_rule_compliance(self, rule: str, policy: SecurityPolicy) -> bool:
        """检查单个规则的合规性"""
        # 这里应该实现具体的规则检查逻辑
        # 简化示例
        if "require_https" in rule:
            return self.config.get('use_https', False)
        elif "encrypt_at_rest" in rule:
            return self.config.get('database_encryption', False)
        elif "rate_limit" in rule:
            return self.config.get('rate_limiting_enabled', False)

        return True

    def _generate_security_recommendations(self, violations: List[str]) -> List[str]:
        """生成安全建议"""
        recommendations = []

        if any("https" in v for v in violations):
            recommendations.append("Enable HTTPS for all communications")

        if any("encrypt" in v for v in violations):
            recommendations.append("Enable encryption for data at rest and in transit")

        if any("rate_limit" in v for v in violations):
            recommendations.append("Implement rate limiting to prevent abuse")

        return recommendations

class AuditLogger:
    """审计日志记录器"""

    def __init__(self, log_file: str = "audit.log"):
        self.log_file = log_file
        self.setup_logging()

    def setup_logging(self):
        """设置日志记录"""
        import logging

        # 创建审计日志记录器
        self.logger = logging.getLogger('audit')
        self.logger.setLevel(logging.INFO)

        # 创建文件处理器
        handler = logging.FileHandler(self.log_file)
        handler.setLevel(logging.INFO)

        # 创建格式器
        formatter = logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s'
        )
        handler.setFormatter(formatter)

        self.logger.addHandler(handler)

    def log_authentication(self, user_id: str, success: bool, ip_address: str):
        """记录认证事件"""
        status = "SUCCESS" if success else "FAILURE"
        self.logger.info(f"AUTH_{status} - User: {user_id}, IP: {ip_address}")

    def log_authorization(self, user_id: str, resource: str, action: str, success: bool):
        """记录授权事件"""
        status = "GRANTED" if success else "DENIED"
        self.logger.info(f"AUTHZ_{status} - User: {user_id}, Resource: {resource}, Action: {action}")

    def log_data_access(self, user_id: str, resource: str, operation: str):
        """记录数据访问事件"""
        self.logger.info(f"DATA_ACCESS - User: {user_id}, Resource: {resource}, Operation: {operation}")

    def log_security_event(self, event_type: str, details: Dict[str, Any]):
        """记录安全事件"""
        self.logger.warning(f"SECURITY_EVENT - Type: {event_type}, Details: {details}")

    def log_configuration_change(self, user_id: str, component: str, old_value: str, new_value: str):
        """记录配置变更"""
        self.logger.info(f"CONFIG_CHANGE - User: {user_id}, Component: {component}, Old: {old_value}, New: {new_value}")

# 安全中间件
class SecurityMiddleware:
    """安全中间件"""

    def __init__(self, security_manager: SecurityManager, audit_logger: AuditLogger):
        self.security_manager = security_manager
        self.audit_logger = audit_logger
        self.rate_limiter = RateLimiter()

    async def __call__(self, request, call_next):
        """中间件处理"""
        # 获取客户端IP
        client_ip = self._get_client_ip(request)

        # 速率限制检查
        if not await self.rate_limiter.check_rate_limit(client_ip):
            self.audit_logger.log_security_event("RATE_LIMIT_EXCEEDED", {"ip": client_ip})
            return {"error": "Rate limit exceeded", "status_code": 429}

        # HTTPS检查
        if not request.url.scheme == "https" and not self._is_development():
            self.audit_logger.log_security_event("HTTP_REQUEST", {"ip": client_ip, "url": str(request.url)})
            return {"error": "HTTPS required", "status_code": 400}

        # 输入验证
        if request.method in ["POST", "PUT", "PATCH"]:
            if not await self._validate_request_data(request):
                self.audit_logger.log_security_event("INVALID_INPUT", {"ip": client_ip})
                return {"error": "Invalid input data", "status_code": 400}

        # 执行请求
        response = await call_next(request)

        # 添加安全头
        response = self._add_security_headers(response)

        return response

    def _get_client_ip(self, request) -> str:
        """获取客户端IP"""
        forwarded_for = request.headers.get("X-Forwarded-For")
        if forwarded_for:
            return forwarded_for.split(",")[0].strip()
        return request.client.host

    def _is_development(self) -> bool:
        """检查是否为开发环境"""
        return os.getenv("ENVIRONMENT", "production") == "development"

    async def _validate_request_data(self, request) -> bool:
        """验证请求数据"""
        try:
            # 这里应该实现具体的输入验证逻辑
            return True
        except Exception:
            return False

    def _add_security_headers(self, response):
        """添加安全头"""
        security_headers = {
            "X-Content-Type-Options": "nosniff",
            "X-Frame-Options": "DENY",
            "X-XSS-Protection": "1; mode=block",
            "Strict-Transport-Security": "max-age=31536000; includeSubDomains",
            "Content-Security-Policy": "default-src 'self'",
            "Referrer-Policy": "strict-origin-when-cross-origin"
        }

        for header, value in security_headers.items():
            response.headers[header] = value

        return response

class RateLimiter:
    """速率限制器"""

    def __init__(self):
        self.requests = {}  # 在生产环境中应该使用Redis
        self.limits = {
            'default': {'requests': 100, 'window': 60},  # 每分钟100请求
            'auth': {'requests': 10, 'window': 60},      # 认证接口每分钟10请求
            'api': {'requests': 1000, 'window': 3600}    # API每小时1000请求
        }

    async def check_rate_limit(self, identifier: str, limit_type: str = 'default') -> bool:
        """检查速率限制"""
        import time

        current_time = int(time.time())
        limit_config = self.limits.get(limit_type, self.limits['default'])
        window_size = limit_config['window']
        max_requests = limit_config['requests']

        # 清理过期记录
        if identifier in self.requests:
            self.requests[identifier] = [
                req_time for req_time in self.requests[identifier]
                if current_time - req_time < window_size
            ]
        else:
            self.requests[identifier] = []

        # 检查是否超过限制
        if len(self.requests[identifier]) >= max_requests:
            return False

        # 记录当前请求
        self.requests[identifier].append(current_time)
        return True

# 使用示例
def create_security_config():
    """创建安全配置"""
    config = {
        'jwt_secret_key': os.getenv('JWT_SECRET_KEY', secrets.token_urlsafe(32)),
        'encryption_key_file': '.encryption_key',
        'use_https': True,
        'database_encryption': True,
        'rate_limiting_enabled': True
    }

    security_manager = SecurityManager(config)
    audit_logger = AuditLogger()

    return security_manager, audit_logger
```

## 📊 部署检查清单

### 1. 基础设施检查
- [ ] 服务器资源配置充足（CPU、内存、磁盘）
- [ ] 网络配置正确（防火墙、负载均衡）
- [ ] 域名和SSL证书配置
- [ ] 数据库和缓存服务正常运行
- [ ] 备份和恢复机制就绪

### 2. 应用配置检查
- [ ] 环境变量正确配置
- [ ] 数据库连接和迁移完成
- [ ] 静态文件和CDN配置
- [ ] 日志级别和输出配置
- [ ] 健康检查端点正常

### 3. 安全配置检查
- [ ] HTTPS强制启用
- [ ] 身份认证和授权配置
- [ ] 安全头和CORS配置
- [ ] 敏感数据加密
- [ ] 审计日志启用

### 4. 监控配置检查
- [ ] 系统监控指标收集
- [ ] 应用性能监控
- [ ] 错误追踪和告警
- [ ] 日志聚合和分析
- [ ] 仪表板和可视化

### 5. 运维流程检查
- [ ] CI/CD流水线配置
- [ ] 自动化部署脚本
- [ ] 回滚机制准备
- [ ] 故障处理流程
- [ ] 文档和手册完整

## 🎯 性能优化建议

### 1. 应用层优化
- **代码优化**：消除性能瓶颈，优化算法复杂度
- **缓存策略**：多层缓存，合理的缓存失效策略
- **数据库优化**：索引优化，查询优化，连接池配置
- **异步处理**：使用异步编程，避免阻塞操作

### 2. 基础设施优化
- **负载均衡**：合理的负载分发策略
- **CDN配置**：静态资源加速，全球分发
- **数据库分离**：读写分离，主从复制
- **容器优化**：资源限制，镜像优化

### 3. 监控和调优
- **性能基线**：建立性能基准，持续监控
- **瓶颈分析**：定期分析系统瓶颈
- **容量规划**：预测资源需求，提前扩容
- **故障演练**：定期进行故障演练

## ➡️ 总结

完成本阶段学习后，你应该：
- 具备完整的生产部署能力
- 掌握监控和运维技能
- 建立安全防护体系
- 形成持续改进的运维文化

恭喜你完成了OpenHands项目的完整学习之旅！现在你已经具备了从基础理解到生产部署的全栈能力。继续实践和探索，成为OpenHands生态的贡献者吧！🎉
