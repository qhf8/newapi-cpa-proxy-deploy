# New API + CPA/CLIProxyAPI 中转站部署模板

这是一个已脱敏的公开部署模板，用来记录一套小型 OpenAI-compatible API 中转站的基础结构。

主要组件：

- New API：负责用户、令牌、模型路由和额度统计
- CPA/CLIProxyAPI：负责 ChatGPT/Codex OAuth 账号池代理
- PostgreSQL + Redis：New API 的数据库和缓存
- CPA Usage Keeper：采集 CPA 用量、辅助账号池状态管理
- 可选 Grok2API：作为 New API 的独立 Grok 上游渠道
- 可选上游代理：给需要特殊 User-Agent 的 OpenAI-compatible 上游做一层 Nginx 代理

本仓库不是生产目录原样备份，而是公开可读的部署骨架。真实密钥、账号 JSON、数据库、日志、备份和 token 池都没有上传。

## 目录结构

```text
docker-compose.yml              New API + CPA + Postgres + Redis + Usage Keeper
docker-compose.grok2api.yml     可选 Grok2API 服务编排
.env.example                    环境变量示例
cpa/config.example.yaml         CPA 配置模板
cpa-usage-keeper/               FastAPI 用量采集服务
ezouapi-proxy/                  可选 Nginx 上游代理模板
ops/                            运维辅助脚本
scripts/                        CPA 账号池辅助脚本
usage-proxy.py                  本地用量查询小服务
```

## 快速开始

复制示例配置：

```bash
cp .env.example .env
cp cpa/config.example.yaml cpa/config.yaml
mkdir -p cpa/auths cpa/logs newapi/data newapi/logs
```

编辑 `.env` 和 `cpa/config.yaml` 后启动：

```bash
docker compose up -d
```

`cpa/config.example.yaml` 里的 CPA API Key 和管理后台密码哈希需要替换成你自己的值，因为 CLIProxyAPI 会直接读取 YAML 文件。

在 Windows 上可以用脚本从环境变量渲染这两个占位值：

```powershell
$env:CPA_API_KEY = "sk-your-cpa-key"
$env:CPA_MANAGEMENT_SECRET_HASH = "your-bcrypt-hash"
.\scripts\render-cpa-config.ps1
```

如果你还把 Grok2API fork 放到 `./grok2api`，可以这样同时启动：

```bash
docker compose -f docker-compose.yml -f docker-compose.grok2api.yml up -d
```

## 公开脱敏边界

不要提交以下内容：

- `.env`
- `cpa/auths/*.json`
- New API 数据库文件或 SQL dump
- Grok SSO token 池
- 日志、备份、生成的图片/视频
- 账号状态文件、临时文件、本地密钥

仓库里的公开文件只保留模板、代码和占位值，真实生产配置应只放在服务器本地。
