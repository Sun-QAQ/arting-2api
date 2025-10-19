# 🎨 arting-2api：AI 绘画「万能转换插头」

![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)
![Python Version](https://img.shields.io/badge/Python-3.10+-brightgreen.svg)
![Docker](https://img.shields.io/badge/Docker-Ready-blue.svg?logo=docker)
![GitHub Repo](https://img.shields.io/badge/GitHub-lzA6/arting--2api-violet.svg)

> "我们不是在编写代码，我们是在构建连接思想的桥梁。" —— 某位不愿透露姓名的开发者

`arting-2api` 是一个创新的开源项目，它就像一个万能转换插头，能够将 [Arting.ai](https://arting.ai/) 的强大绘画能力，无缝转换成与 **OpenAI DALL-E**、**Stable Diffusion WebUI** 等业界主流应用完全兼容的 API 接口。

这意味着，你现在可以用最熟悉的工具和客户端，免费享受 Arting.ai 的高质量绘画服务！

---

## ✨ 核心价值与特色

### 💸 极致性价比
将优秀的免费绘画服务接入到需要付费 API 的工作流中，实现"零成本"高质量出图。

### 🧩 无缝兼容
无需修改现有工具！无论是支持 OpenAI 图像接口的客户端，还是为 Stable Diffusion WebUI 设计的插件，都能直接使用。

### 🚀 一键部署
通过 Docker 将复杂的环境配置压缩成简单命令，真正实现"开箱即用"。

### 🌐 多模式适配
- **OpenAI 原生模式** (`/v1/images/generations`)：兼容所有 DALL-E 接口应用
- **智能聊天模式** (`/v1/chat/completions`)：在聊天中直接生成图片
- **SD-WebUI 模式** (`/sdapi/v1/txt2img`)：让 SD 工作流用上 Arting 模型

### 💖 开放精神
鼓励探索、改造和分享，用技术力量将封闭服务重新带回开放生态。

---

## 🎯 适用场景

### 👨‍💻 开发者
在自有应用中集成免费、高质量的 AI 绘画能力

### 🎨 设计师与艺术家
在熟悉的 UI 界面中体验不同模型风格

### 🤖 AI 爱好者
低成本学习 API 转换、反向代理等技术的实践平台

### 💰 精打细算的用户
寻找 OpenAI DALL-E 等付费服务的完美替代方案

---

## 🚀 快速开始

### 环境要求
- **Docker** & **Docker Compose** [安装指南](https://www.docker.com/get-started/)

### 部署步骤

**1. 获取项目**
```bash
git clone https://github.com/lzA6/arting-2api.git
cd arting-2api
```

**2. 配置认证令牌 🔑**
```bash
# 复制环境配置模板
cp .env.example .env
```

获取 `ARTING_AUTH_TOKEN`：
- 登录 [Arting.ai](https://arting.ai/)
- 按 `F12` 打开开发者工具，切换到 `Network` 面板
- 生成一张图片，在网络请求中找到对 `api.arting.ai` 的请求
- 复制 `Authorization` 请求头的值
- 粘贴到 `.env` 文件的 `ARTING_AUTH_TOKEN` 中

**3. 启动服务**
```bash
docker-compose up -d
```

**完成！** 🎉 服务已在 `http://localhost:8090` 运行

---

## 🛠️ API 使用指南

### Web UI 测试
访问 `http://localhost:8090/` 使用内置测试界面

### OpenAI 客户端配置
- **API Base URL**: `http://localhost:8090/v1`
- **API Key**: `.env` 中设置的 `API_MASTER_KEY`（默认为 `1`）

兼容客户端：
- [NextChat](https://github.com/ChatGPTNextWeb/ChatGPT-Next-Web)
- [LobeChat](https://github.com/lobehub/lobe-chat)
- 其他支持 OpenAI 格式的应用

### Stable Diffusion WebUI 集成
将外部 API 指向：`http://localhost:8090/sdapi/v1/txt2img`

---

## 🔬 技术架构

### 核心设计理念：API 适配器模式

`arting-2api` 充当了一个智能翻译官，在不同 API 协议间进行实时转换：

```
┌─────────────────┐    OpenAI Format    ┌──────────────────┐    Arting Format    ┌─────────────┐
│                 │ ──────────────────> │                  │ ──────────────────> │             │
│   Your Client   │                     │   arting-2api    │                     │  Arting.ai  │
│                 │ <────────────────── │                  │ <────────────────── │             │
└─────────────────┘    Image URL/Data   └──────────────────┘    Polling Result   └─────────────┘
```

### 技术栈详解

| 组件 | 作用 | 技术选型理由 |
|------|------|--------------|
| **FastAPI** | 核心 Web 框架 | 🚀 异步高性能，适合处理并发轮询请求 |
| **Cloudscraper** | 绕过 Cloudflare 保护 | 🛡️ 模拟真实浏览器行为，确保服务稳定 |
| **Docker & Nginx** | 容器化与反向代理 | 📦 环境隔离，简化部署，提升性能 |
| **轮询机制** | 异步任务状态检查 | 📞 定期查询任务结果，保证数据同步 |
| **Pydantic** | 数据验证与配置管理 | 📏 类型安全，配置验证，代码健壮 |
| **aiohttp** | 异步 HTTP 客户端 | ⚡️ 高效处理图片下载等 I/O 密集型操作 |

### 核心工作流程

1. **请求接收**：接收标准 OpenAI 格式请求
2. **格式转换**：将请求参数转换为 Arting.ai 格式
3. **任务提交**：向 Arting.ai 提交绘画任务
4. **状态轮询**：定期检查任务完成状态
5. **结果返回**：获取图片 URL 并转换为客户端期望格式

---

## 📂 项目结构

```
arting-2api/
├── 🐳 Docker 配置
│   ├── Dockerfile           # 容器构建定义
│   └── docker-compose.yml   # 服务编排配置
├── 🔧 核心配置
│   ├── .env.example         # 环境变量模板
│   └── nginx.conf          # Nginx 反向代理配置
├── 🐍 Python 源码
│   ├── main.py              # FastAPI 应用入口
│   ├── requirements.txt     # Python 依赖列表
│   └── app/
│       ├── core/
│       │   └── config.py    # 配置管理
│       ├── providers/
│       │   ├── base_provider.py    # Provider 基类
│       │   └── arting_provider.py  # Arting.ai 适配器
│       └── utils/
│           └── sse_utils.py # 服务器发送事件工具
└── 🌐 静态资源
    └── static/              # Web UI 界面文件
        ├── index.html
        ├── style.css
        └── script.js
```

---

## 🗺️ 发展路线图

### 🎯 短期目标（v1.1）
- [ ] **性能优化**：实现请求缓存机制
- [ ] **错误处理**：增强异常处理和用户反馈
- [ ] **文档完善**：添加更多使用示例和故障排查指南

### 🚀 中期规划（v2.0）
- [ ] **WebSocket 支持**：替换轮询机制，实现实时进度更新
- [ ] **多 Provider 支持**：集成 Midjourney、DreamStudio 等服务
- [ ] **图生图功能**：支持图像到图像的转换

### 🌟 长期愿景
- [ ] **分布式架构**：支持多节点部署和负载均衡
- [ ] **用户管理系统**：添加使用统计和配额管理
- [ ] **插件生态**：建立第三方插件开发规范

---

## 🤝 贡献指南

我们欢迎各种形式的贡献！

### 如何参与
1. **报告问题**：在 GitHub Issues 中提交 bug 报告或功能建议
2. **代码贡献**：Fork 项目，创建功能分支，提交 Pull Request
3. **文档改进**：优化文档、翻译或添加使用教程
4. **技术讨论**：参与技术方案设计和架构优化

### 开发环境搭建
```bash
# 克隆项目
git clone https://github.com/lzA6/arting-2api.git
cd arting-2api

# 安装依赖
pip install -r requirements.txt

# 配置环境变量
cp .env.example .env
# 编辑 .env 文件配置认证信息

# 启动开发服务器
uvicorn main:app --reload --port 8090
```

### 贡献者协议
- 遵循现有的代码风格和项目结构
- 确保提交前通过所有测试
- 更新相关文档和示例
- 在 Pull Request 中详细描述变更内容

---

## 📜 开源协议

本项目采用 **Apache 2.0** 开源许可证。

**允许事项：**
- ✅ 自由使用、修改和分发代码
- ✅ 用于个人或商业项目
- ✅ 专利授权

**要求事项：**
- 📝 保留原始版权和许可证声明
- ℹ️ 声明对源代码的修改

**禁止事项：**
- ❌ 使用项目商标
- ❌ 追究原作者责任

---

## 🆘 常见问题

### Q: 如何获取最新的 Arting.ai 认证令牌？
A: 令牌可能定期失效，请按以下步骤更新：
1. 重新登录 Arting.ai
2. 按 F12 打开开发者工具
3. 生成新图片并捕获最新请求
4. 复制新的 Authorization 头值
5. 更新 .env 文件并重启服务

### Q: 服务启动失败怎么办？
A: 检查以下常见问题：
- Docker 服务是否正常运行
- .env 文件格式是否正确（无多余空格）
- 端口 8090 是否被其他程序占用
- 网络连接是否正常

### Q: 如何查看服务日志？
A: 使用以下命令查看容器日志：
```bash
docker-compose logs -f
```

---

## 🙏 致谢

感谢所有为这个项目做出贡献的开发者、测试者和文档编写者。特别感谢：

- **Arting.ai** 团队提供优秀的免费绘画服务
- **FastAPI** 社区提供出色的 Web 框架
- 所有开源项目的贡献者，你们的工作让这个世界更美好

---

## 📞 支持与联系

- **GitHub Issues**: [提交问题与建议](https://github.com/lzA6/arting-2api/issues)
- **文档更新**: 欢迎提交文档改进的 Pull Request
- **功能讨论**: 在 GitHub Discussions 中参与技术交流

---

**记住，每一个伟大的创意都始于一次简单的尝试。现在，去创造属于你的视觉奇迹吧！** ✨
