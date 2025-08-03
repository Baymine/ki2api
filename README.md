# Ki2API - Claude Sonnet 4 OpenAI & Claude 兼容API

一个简单易用的Docker化API服务，同时支持OpenAI和Claude API格式，专门用于Claude Sonnet 4模型。

## 功能特点

- 🐳 **Docker傻瓜式运行** - 一行命令启动服务
- 🔑 **固定API密钥** - 使用 `ki2api-key-2024`
- 🎯 **多模型支持** - 支持多个Claude模型
- 🌐 **双API兼容** - 完全兼容OpenAI和Claude API格式
- 📡 **流式传输** - 支持SSE流式响应
- 🔄 **自动token刷新** - 支持token过期自动刷新
- 🛠️ **工具调用** - 支持function calling

## 快速开始

### 零配置启动（推荐）

只需确保已登录Kiro，然后一键启动：

```bash
docker-compose up -d
```

服务将在 http://localhost:8989 启动

### 自动读取token

容器会自动读取你本地的token文件：
- **macOS/Linux**: `~/.aws/sso/cache/kiro-auth-token.json`
- **Windows**: `%USERPROFILE%\.aws\sso\cache\kiro-auth-token.json`

### 3. 测试API

#### OpenAI API 格式

##### 获取模型列表
```bash
curl -H "Authorization: Bearer ki2api-key-2024" \
     http://localhost:8989/v1/models
```

##### 非流式对话
```bash
curl -X POST http://localhost:8989/v1/chat/completions \
  -H "Authorization: Bearer ki2api-key-2024" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-3-5-sonnet-20241022",
    "messages": [
      {"role": "user", "content": "你好，请介绍一下自己"}
    ],
    "max_tokens": 1000
  }'
```

##### 流式对话
```bash
curl -X POST http://localhost:8989/v1/chat/completions \
  -H "Authorization: Bearer ki2api-key-2024" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-3-5-sonnet-20241022",
    "messages": [
      {"role": "user", "content": "写一首关于春天的诗"}
    ],
    "stream": true,
    "max_tokens": 500
  }'
```

#### Claude API 格式

##### 非流式对话
```bash
curl -X POST http://localhost:8989/v1/messages \
  -H "Authorization: Bearer ki2api-key-2024" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-3-5-sonnet-20241022",
    "max_tokens": 1000,
    "messages": [
      {"role": "user", "content": "你好，请介绍一下自己"}
    ]
  }'
```

##### 流式对话
```bash
curl -X POST http://localhost:8989/v1/messages \
  -H "Authorization: Bearer ki2api-key-2024" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-3-5-sonnet-20241022",
    "max_tokens": 1000,
    "messages": [
      {"role": "user", "content": "写一首关于春天的诗"}
    ],
    "stream": true
  }'
```

##### 带系统提示的对话
```bash
curl -X POST http://localhost:8989/v1/messages \
  -H "Authorization: Bearer ki2api-key-2024" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-3-5-sonnet-20241022",
    "max_tokens": 1000,
    "system": "你是一个专业的AI助手，请用简洁明了的方式回答问题。",
    "messages": [
      {"role": "user", "content": "什么是机器学习？"}
    ]
  }'
```

## Docker使用方法

### 使用Docker Compose（推荐）
```bash
# 启动服务
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止服务
docker-compose down
```

### 使用Docker命令
```bash
# 构建镜像
docker build -t ki2api .

# 运行容器
docker run -d \
  -p 8989:8989 \
  -e KIRO_ACCESS_TOKEN=your_token \
  -e KIRO_REFRESH_TOKEN=your_refresh_token \
  --name ki2api \
  ki2api
```

## API端点

### OpenAI 兼容端点

#### 1. 获取模型列表
- **端点**: `GET /v1/models`
- **描述**: 获取可用模型列表
- **认证**: 需要API密钥

#### 2. 聊天完成
- **端点**: `POST /v1/chat/completions`
- **描述**: 创建聊天完成（OpenAI格式）
- **认证**: 需要API密钥
- **支持**: 流式和非流式响应

### Claude 兼容端点

#### 3. 消息创建
- **端点**: `POST /v1/messages`
- **描述**: 创建消息（Claude格式）
- **认证**: 需要API密钥
- **支持**: 流式和非流式响应
- **特性**: 支持系统提示、工具调用、图片输入

### 通用端点

#### 4. 健康检查
- **端点**: `GET /health`
- **描述**: 服务健康状态检查
- **认证**: 不需要

#### 5. 服务信息
- **端点**: `GET /`
- **描述**: 获取服务信息和支持的功能
- **认证**: 不需要

## 支持的模型

本服务支持以下Claude模型：

| 模型名称 | 描述 | 支持的API |
|---------|------|----------|
| `claude-3-5-sonnet-20241022` | Claude 3.5 Sonnet (最新版本) | OpenAI & Claude |
| `claude-3-5-sonnet-20240620` | Claude 3.5 Sonnet (旧版本) | OpenAI & Claude |
| `claude-3-sonnet-20240229` | Claude 3 Sonnet | OpenAI & Claude |
| `claude-3-haiku-20240307` | Claude 3 Haiku | OpenAI & Claude |
| `claude-sonnet-4-20250514` | Claude Sonnet 4 (内部版本) | OpenAI & Claude |

所有模型都支持：
- 文本对话
- 流式响应
- 工具调用（Function Calling）
- 图片输入（Vision）
- 系统提示

## 环境变量

| 变量名 | 默认值 | 说明 |
|--------|--------|------|
| API_KEY | ki2api-key-2024 | API访问密钥 |
| KIRO_ACCESS_TOKEN | - | Kiro访问令牌（必需） |
| KIRO_REFRESH_TOKEN | - | Kiro刷新令牌（必需） |

## 开发模式

### 本地运行
```bash
# 安装依赖
pip install -r requirements.txt

# 设置环境变量
export KIRO_ACCESS_TOKEN=your_token
export KIRO_REFRESH_TOKEN=your_refresh_token

# 启动服务
python app.py
```

## 故障排除

### 常见问题

1. **Token过期**
   - 确保refresh token有效
   - 重新获取最新的token

2. **连接失败**
   - 检查端口8989是否被占用
   - 确认Docker容器正常运行

3. **API返回401**
   - 确认使用了正确的API密钥：`ki2api-key-2024`
   - 检查token是否有效

### 查看日志
```bash
# Docker日志
docker-compose logs -f ki2api

# 本地日志
python app.py 2>&1 | tee ki2api.log
```

## 项目结构
```
ki2api/
├── app.py              # 主应用文件
├── Dockerfile          # Docker镜像定义
├── docker-compose.yml  # Docker Compose配置
├── requirements.txt    # Python依赖
└── README.md          # 本文档
```

## 许可证

MIT License