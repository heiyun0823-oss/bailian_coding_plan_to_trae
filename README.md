启动不成功的直接用压缩包，我的问题上传的时候由于temp目录是空的文件夹上传不了github，导致很多人启动不成功

# Nginx AI Proxy

本项目是一个基于 Nginx 的反向代理服务，用于将 OpenAI API 请求转发到多个国内 AI 服务。

## 项目作用

将针对 `api.openai.com` 的 API 请求代理转发到多个 AI 服务提供商，使得使用 OpenAI SDK 或 API 的应用可以无缝切换到国内 AI 服务。

## 支持的 AI 服务

| 端口 | 服务商 | 代理目标 | 可用模型 |
|------|--------|----------|----------|
| **443** | 百炼 Coding Plan | coding.dashscope.aliyuncs.com | qwen3.5-plus, qwen3-max, qwen3-coder-next, qwen3-coder-plus, MiniMax-M2.5, glm-5, glm-4.7, kimi-k2.5 |
| **8443** | 百炼普通 API | dashscope.aliyuncs.com | qwen3-max-2026-01-23, qwen3-coder-plus |
| **8444** | 智谱 AI | open.bigmodel.cn | glm-4.7 |
| **8445** | 月之暗面 | api.moonshot.cn | kimi-k2.5 |

## 目录结构

```
nginx-ai-proxy/
├── cert/                    # SSL 证书目录
│   ├── api.openai.com.crt   # OpenAI 域名证书
│   ├── api.openai.com.key   # OpenAI 域名私钥
│   └── openaica.crt         # CA 证书
├── conf/                    # Nginx 配置目录
│   └── nginx.conf           # 主配置文件
├── logs/                    # 日志目录
│   ├── access.log           # 访问日志
│   └── error.log            # 错误日志
├── nginx.exe                # Nginx 可执行文件
└── killnginx.bat            # 停止 Nginx 脚本
```

## 使用方法

### 1. 配置 hosts 文件

以管理员身份编辑 `C:\Windows\System32\drivers\etc\hosts`，添加以下内容：

```
127.0.0.1 api.openai.com
```

### 2. 安装并信任 SSL 证书

1. 双击 `cert/openaica.crt` 文件
2. 点击"安装证书"
3. 选择"本地计算机" -> "将所有的证书都放入下列存储" -> "受信任的根证书颁发机构"
4. 完成安装

### 3. 启动 Nginx

双击 `nginx.exe` 或在命令行中执行：

```bash
./nginx.exe
```

### 4. 停止 Nginx

双击 `killnginx.bat` 或在命令行中执行：

```bash
./killnginx.bat
```

或者使用命令：

```bash
nginx -s stop
```

## 重要配置说明

### nginx.conf 关键配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `worker_processes` | 10 | 工作进程数 |
| `worker_connections` | 1024 | 每个进程的最大连接数 |
| `proxy_buffering` | off | 关闭缓冲，支持流式响应 |
| `proxy_read_timeout` | 3600s | 超时时间，适应长时间 AI 响应 |
| `proxy_ssl_verify` | off | 关闭 SSL 验证 |

## 在 Trae 中使用

### 百炼 Coding Plan (端口 443)

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "bailian-coding": {
        "baseUrl": "https://api.openai.com/v1",
        "apiKey": "你的百炼API Key",
        "api": "openai-completions",
        "models": [
          { "id": "qwen3.5-plus" },
          { "id": "qwen3-max-2026-01-23" },
          { "id": "qwen3-coder-next" },
          { "id": "qwen3-coder-plus" },
          { "id": "MiniMax-M2.5" },
          { "id": "glm-5" },
          { "id": "glm-4.7" },
          { "id": "kimi-k2.5" }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "bailian-coding/qwen3.5-plus"
      }
    }
  }
}
```

### 百炼普通 API (端口 8443)

```json
{
  "baseUrl": "https://api.openai.com:8443/v1",
  "apiKey": "你的百炼API Key"
}
```

### 智谱 AI (端口 8444)

```json
{
  "baseUrl": "https://api.openai.com:8444/v1",
  "apiKey": "你的智谱API Key"
}
```

### 月之暗面 (端口 8445)

```json
{
  "baseUrl": "https://api.openai.com:8445/v1",
  "apiKey": "你的月之暗面API Key"
}
```

## 注意事项

1. **证书信任**：必须将 CA 证书安装到"受信任的根证书颁发机构"，否则 HTTPS 请求会失败。

2. **hosts 配置**：确保 hosts 文件配置正确，否则请求不会经过本地代理。

3. **端口占用**：确保 443、8443、8444、8445 端口没有被其他程序占用。

4. **日志查看**：如遇问题，请查看 `logs/error.log` 获取详细错误信息。

5. **API Key**：使用时需要在请求头中携带对应服务商的 Authorization，格式与 OpenAI 一致。

6. **防火墙**：可能需要配置防火墙允许 nginx.exe 通过。

## 常见问题

### Q: 启动后无法访问？

检查：
- hosts 文件是否正确配置
- 证书是否已信任
- 对应端口是否被占用

### Q: 如何验证代理是否工作？

查看 `logs/access.log` 文件，应该能看到请求记录。

### Q: 如何修改代理目标？

编辑 `conf/nginx.conf` 中的 `proxy_pass` 配置项，修改为目标 API 地址。

### Q: 如何验证模型列表？

```powershell
curl -k https://api.openai.com/v1/models
curl -k https://api.openai.com:8443/v1/models
curl -k https://api.openai.com:8444/v1/models
curl -k https://api.openai.com:8445/v1/models
```

## 模型推荐

| 用途 | 推荐模型 | 端口 | 原因 |
|------|---------|------|------|
| 日常对话 | qwen3.5-plus | 443 | 1M上下文，支持图像 |
| 代码生成 | qwen3-coder-plus | 443/8443 | 1M上下文，专为代码优化 |
| 复杂推理 | qwen3-max-2026-01-23 | 443/8443 | 最新最强模型 |
| 国产替代 | glm-4.7 | 443/8444 | 智谱最新模型 |
| 长文本 | kimi-k2.5 | 443/8445 | 月之暗面擅长长文本 |
