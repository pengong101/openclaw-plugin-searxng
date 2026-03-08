# 📦 OpenClaw SearXNG Plugin 部署指南

**版本：** 1.0.0  
**更新日期：** 2026-03-08  
**预计耗时：** 15-30 分钟

---

## 🎯 部署方式选择

### 方式 1：单机部署（适合个人使用）

**特点：**
- 简单快速
- 资源消耗少
- 适合测试和学习

**预计耗时：** 15 分钟

---

### 方式 2：Docker 部署（推荐）

**特点：**
- 环境隔离
- 易于管理
- 生产环境首选

**预计耗时：** 20 分钟

---

### 方式 3：双核心部署（适合团队）

**特点：**
- PC 端 + 网关端协作
- 隐私与便利兼顾
- 最佳性能

**预计耗时：** 30 分钟

---

## 📋 前置要求

### 系统要求

| 组件 | 最低要求 | 推荐配置 |
|------|----------|----------|
| CPU | 2 核 | 4 核+ |
| 内存 | 2GB | 4GB+ |
| 存储 | 10GB | 20GB+ |
| Node.js | v18+ | v20+ |
| Docker | 20.10+ | 24.0+ |

### 网络要求

- 可访问互联网
- SearXNG 服务可达
- （可选）Redis 服务

---

## 🚀 方式 1：单机部署

### 步骤 1：安装 Node.js

**Windows：**
```powershell
# 下载安装 https://nodejs.org/
# 验证安装
node --version  # 应显示 v18+
npm --version   # 应显示 9+
```

**Linux：**
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# 验证安装
node --version
npm --version
```

**macOS：**
```bash
brew install node@20

# 验证安装
node --version
npm --version
```

---

### 步骤 2：安装插件

```bash
# 全局安装
npm install -g openclaw-plugin-searxng

# 或本地安装
npm install openclaw-plugin-searxng
```

---

### 步骤 3：配置插件

```bash
# 创建配置目录
mkdir -p ~/.openclaw/plugins/searxng
cd ~/.openclaw/plugins/searxng

# 创建配置文件
cat > config.json << EOF
{
  "enabled": true,
  "baseUrl": "http://localhost:8083",
  "timeout": 20000,
  "cache": {
    "enabled": false
  }
}
EOF
```

---

### 步骤 4：启用插件

```bash
# OpenClaw 中启用
openclaw plugins enable searxng

# 验证状态
openclaw plugins status
```

---

### 步骤 5：测试

```bash
# 在 OpenClaw 对话中
"搜索 测试"

# 或命令行测试
openclaw search "毫米波雷达 2026"
```

---

## 🐳 方式 2：Docker 部署

### 步骤 1：准备环境

```bash
# 检查 Docker
docker --version
docker-compose --version

# 创建工作目录
mkdir -p /opt/openclaw-searxng
cd /opt/openclaw-searxng
```

---

### 步骤 2：创建 docker-compose.yml

```yaml
version: '3.8'

services:
  openclaw:
    image: pengong101/openclaw-plugin-searxng:latest
    container_name: openclaw-searxng
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - ./config:/app/config
      - ./workspace:/app/workspace
      - ./cache:/app/cache
    environment:
      - NODE_ENV=production
      - SEARXNG_BASE_URL=http://192.168.1.122:8083
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    depends_on:
      - redis
    networks:
      - openclaw-net

  redis:
    image: redis:alpine
    container_name: openclaw-redis
    restart: unless-stopped
    volumes:
      - redis-data:/data
    networks:
      - openclaw-net

volumes:
  redis-data:

networks:
  openclaw-net:
    driver: bridge
```

---

### 步骤 3：创建配置文件

```bash
mkdir -p config

cat > config/config.json << 'EOF'
{
  "plugins": {
    "searxng": {
      "enabled": true,
      "baseUrl": "http://searxng:8083",
      "timeout": 20000,
      "cache": {
        "enabled": true,
        "ttl": 3600,
        "redis": {
          "host": "redis",
          "port": 6379
        }
      },
      "engines": {
        "priority": ["baidu", "bing-cn", "wikipedia"],
        "timeout": {
          "baidu": 20,
          "bing": 15,
          "wikipedia": 10
        }
      }
    }
  }
}
EOF
```

---

### 步骤 4：启动服务

```bash
# 启动所有服务
docker-compose up -d

# 查看状态
docker-compose ps

# 查看日志
docker-compose logs -f openclaw
```

---

### 步骤 5：验证部署

```bash
# 健康检查
curl http://localhost:8080/health

# 测试搜索
curl "http://localhost:8080/api/search?q=test"

# 查看缓存状态
docker exec openclaw-redis redis-cli ping
```

---

## 🤝 方式 3：双核心部署

### 架构说明

```
┌─────────────────┐         ┌─────────────────┐
│   老马 (PC 端)    │ ←──→   │   小马 (网关端)   │
│  本地 OpenClaw   │  协作   │  云端 OpenClaw   │
└────────┬────────┘         └────────┬────────┘
         │                           │
    本地文件/隐私               远程访问/自动化
    NAS/摄像头                  飞书/微信
```

---

### 步骤 1：部署老马（PC 端）

参考 [双核心部署指南](DUAL-CORE.md) 的老马部分。

**关键配置：**
```json
{
  "name": "laoma",
  "mode": "local",
  "collaboration": {
    "enabled": true,
    "peer": {
      "name": "xiaoma",
      "endpoint": "http://192.168.1.100:8080"
    }
  }
}
```

---

### 步骤 2：部署小马（网关端）

参考 [双核心部署指南](DUAL-CORE.md) 的小马部分。

**关键配置：**
```json
{
  "name": "xiaoma",
  "mode": "gateway",
  "collaboration": {
    "enabled": true,
    "peer": {
      "name": "laoma",
      "endpoint": "http://192.168.1.50:8080"
    }
  }
}
```

---

### 步骤 3：配置协作

**老马配置（PC 端）：**
```json
{
  "plugins": {
    "searxng": {
      "enabled": true,
      "baseUrl": "http://192.168.1.122:8083",
      "collaboration": {
        "delegate": "xiaoma",
        "tasks": ["remote-search", "cache-sync"]
      }
    }
  }
}
```

**小马配置（网关端）：**
```json
{
  "plugins": {
    "searxng": {
      "enabled": true,
      "baseUrl": "http://192.168.1.122:8083",
      "collaboration": {
        "delegate": "laoma",
        "tasks": ["local-cache", "privacy-filter"]
      }
    }
  }
}
```

---

### 步骤 4：测试协作

```bash
# 通过飞书发送指令给小马
"搜索 毫米波雷达 2026"

# 预期流程：
# 小马接收 → 执行搜索 → 结果缓存到老马
# 小马推送结果到飞书
```

---

## 🧪 部署验证

### 验证清单

- [ ] 插件状态正常
- [ ] SearXNG 连接成功
- [ ] 搜索返回结果
- [ ] 缓存功能正常（如启用）
- [ ] 双核心通信正常（如启用）

---

### 测试命令

```bash
# 基础搜索测试
curl "http://localhost:8080/api/search?q=test"

# 中文搜索测试
curl "http://localhost:8080/api/search?q=测试&language=zh-CN"

# 缓存测试（第二次应该更快）
curl "http://localhost:8080/api/search?q=test"
curl "http://localhost:8080/api/search?q=test"

# 健康检查
curl "http://localhost:8080/health"
```

---

### 预期输出

```json
{
  "success": true,
  "query": "test",
  "results": [
    {
      "title": "示例结果",
      "url": "https://example.com",
      "content": "结果摘要..."
    }
  ],
  "count": 10,
  "engine": "searxng",
  "cached": false,
  "duration": 3500
}
```

---

## 🐛 故障排查

### 问题 1：插件无法启用

**症状：** `openclaw plugins enable searxng` 失败

**排查：**
```bash
# 检查 npm 全局路径
npm root -g

# 检查插件安装
npm list -g openclaw-plugin-searxng

# 重新安装
npm uninstall -g openclaw-plugin-searxng
npm install -g openclaw-plugin-searxng
```

---

### 问题 2：SearXNG 连接失败

**症状：** 搜索返回 "Connection refused"

**排查：**
```bash
# 检查 SearXNG 状态
docker ps | grep searxng

# 测试连接
curl http://localhost:8083/search?q=test

# 检查防火墙
sudo ufw status
```

**解决：**
```bash
# 重启 SearXNG
docker restart searxng

# 开放端口
sudo ufw allow 8083
```

---

### 问题 3：缓存不工作

**症状：** 重复搜索没有加速

**排查：**
```bash
# 检查 Redis 状态
docker ps | grep redis

# 测试 Redis 连接
docker exec openclaw-redis redis-cli ping

# 查看缓存日志
docker logs openclaw-searxng | grep cache
```

**解决：**
```bash
# 重启 Redis
docker restart openclaw-redis

# 检查配置
cat config/config.json | grep -A 5 cache
```

---

### 问题 4：双核心通信失败

**症状：** 老马和小马无法协作

**排查：**
```bash
# 测试网络连通性
ping 192.168.1.50
ping 192.168.1.100

# 检查 API 访问
curl http://192.168.1.50:8080/status
curl http://192.168.1.100:8080/status

# 查看协作日志
docker logs openclaw-searxng | grep collaboration
```

**解决：**
```bash
# 检查防火墙规则
sudo ufw allow 8080

# 验证 apiKey 配置
cat config/config.json | grep apiKey
```

---

## 📊 性能优化

### 优化 1：调整超时设置

```json
{
  "timeout": {
    "baidu": 20,
    "bing": 15,
    "wikipedia": 10
  }
}
```

---

### 优化 2：启用 Redis 缓存

```yaml
# docker-compose.yml
services:
  redis:
    image: redis:alpine
    command: redis-server --save "" --appendonly no
```

---

### 优化 3：调整并发数

```json
{
  "concurrency": {
    "maxRequests": 10,
    "maxPerEngine": 5
  }
}
```

---

## 📈 监控与维护

### 日常监控

```bash
# 查看服务状态
docker-compose ps

# 查看资源使用
docker stats

# 查看错误日志
docker logs openclaw-searxng | grep ERROR
```

---

### 定期维护

**每周：**
- [ ] 清理缓存
- [ ] 检查磁盘空间
- [ ] 查看错误日志

**每月：**
- [ ] 更新依赖
- [ ] 审查安全配置
- [ ] 性能基准测试

---

### 备份配置

```bash
# 备份配置文件
tar -czf openclaw-searxng-backup-$(date +%Y%m%d).tar.gz \
  config/ workspace/

# 备份 Redis 数据
docker exec openclaw-redis redis-cli BGSAVE
cp /var/lib/docker/volumes/redis-data/_data/dump.rdb \
  ./backup/dump-$(date +%Y%m%d).rdb
```

---

## 📚 相关文档

- [配置详解](CONFIGURATION.md)
- [使用示例](EXAMPLES.md)
- [双核心部署](DUAL-CORE.md)
- [故障排查](TROUBLESHOOTING.md)

---

**部署成功！** 🎉

如有问题，请查看 [故障排查指南](TROUBLESHOOTING.md) 或提 Issue。
