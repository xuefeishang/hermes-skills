# Hermes Skills 全流程开发知识库

> 🚀 **萝卜飞的AI编程Skills库** — 从需求到测试，覆盖信息系统开发全生命周期

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![中文文档](https://img.shields.io/badge/documentation-中文-brightgreen.svg)](#)
[![Skills](https://img.shields.io/badge/skills-20+-brightblue.svg)](#)
[![GitHub stars](https://img.shields.io/github/stars/萝卜飞/hermes-skills.svg?style=social&label=Star)](#)

---

## 简介

这是一个**面向信息化建设全流程**的AI编程Skills知识库，专为：

- 🏭 **矿业信息化从业者** — 企业级信息系统开发
- 🖥️ **全栈开发工程师** — 前后端分离架构
- 🤖 **AI辅助开发者** — 利用AI提升开发效率

覆盖**需求 → 分析 → 架构 → 编码 → 测试**完整流程！

---

## 核心覆盖

| 开发阶段 | 技能数量 | 主要内容 |
|----------|----------|----------|
| 需求分析 | 3个 | 业务需求调研、需求文档编写、用户故事映射 |
| 系统分析 | 3个 | 业务流程分析、数据流程分析、功能建模 |
| 架构设计 | 4个 | 总体架构设计、技术选型、数据库设计、接口设计 |
| 编码实现 | 5个 | 前端开发、后端开发、API开发、数据库实现、脚本工具 |
| 测试验证 | 3个 | 单元测试、集成测试、系统测试 |

**总计：18个核心技能**

---

## 为什么选择 Hermes Skills？

- ✅ **全流程覆盖** — 从需求到测试，不留死角
- ✅ **中文友好** — 完全中文文档，降低理解门槛
- ✅ **实战导向** — 每个技能都附带可直接运行的代码
- ✅ **AI原生** — 专为AI编程助手设计，可直接加载使用
- ✅ **企业级** — 适配矿业等重工业信息化场景

---

## 快速开始

### 场景一：AI编程助手直接使用（推荐）

将Skills复制到AI编程助手的skills目录：

```bash
# Claude Desktop
cp -r ~/hermes-skills ~/.claude/skills/

# 其他AI工具类似...
```

### 场景二：手动复制提示词

打开任意 `SKILL.md`，复制内容给 ChatGPT/DeepSeek 等通用AI

### 场景三：当作技术手册查阅

直接在 GitHub 上浏览学习

---

## 核心Skills一览

### 📋 需求分析阶段 (3个)

| 技能名称 | 介绍 | 难度 |
|---------|------|------|
| [业务需求调研](./requirement/business-research/) | 系统化需求调研方法和访谈技巧 | ⭐⭐ |
| [需求文档编写](./requirement/requirement-doc/) | 需求规格说明书编写规范和模板 | ⭐⭐ |
| [用户故事映射](./requirement/user-story/) | 用户故事地图和敏捷需求管理 | ⭐⭐⭐ |

### 🔍 系统分析阶段 (3个)

| 技能名称 | 介绍 | 难度 |
|---------|------|------|
| [业务流程分析](./analysis/business-process/) | BPMN流程建模和优化方法 | ⭐⭐⭐ |
| [数据流程分析](./analysis/data-flow/) | DFD数据流图和实体关系建模 | ⭐⭐ |
| [功能建模分析](./analysis/functional-model/) | 用例图和功能分解结构 | ⭐⭐⭐ |

### 🏗️ 架构设计阶段 (4个)

| 技能名称 | 介绍 | 难度 |
|---------|------|------|
| [总体架构设计](./architecture/system-design/) | 系统架构设计和选型策略 | ⭐⭐⭐⭐ |
| [技术选型评估](./architecture/tech-selection/) | 技术栈评估和决策框架 | ⭐⭐⭐ |
| [数据库设计](./architecture/database-design/) | ER图和数据库范式设计 | ⭐⭐⭐ |
| [接口API设计](./architecture/api-design/) | RESTful API设计规范 | ⭐⭐⭐ |

### 💻 编码实现阶段 (5个)

| 技能名称 | 介绍 | 难度 |
|---------|------|------|
| [前端开发实战](./coding/frontend-dev/) | Vue3 + TypeScript开发实践 | ⭐⭐⭐ |
| [后端开发实战](./coding/backend-dev/) | Java/Spring Boot企业级开发 | ⭐⭐⭐⭐ |
| [API接口开发](./coding/api-dev/) | RESTful API实现和文档生成 | ⭐⭐⭐ |
| [数据库实现](./coding/database-impl/) | SQL脚本和存储过程开发 | ⭐⭐ |
| [脚本工具开发](./coding/script-tools/) | Python/Shell自动化脚本 | ⭐⭐ |

### 🧪 测试验证阶段 (3个)

| 技能名称 | 介绍 | 难度 |
|---------|------|------|
| [单元测试设计](./testing/unit-testing/) | JUnit/Vitest单元测试编写 | ⭐⭐⭐ |
| [集成测试实施](./testing/integration-testing/) | API和数据库集成测试 | ⭐⭐⭐ |
| [系统测试管理](./testing/system-testing/) | 端到端测试和性能测试 | ⭐⭐⭐⭐ |

---

## 项目结构

```
hermes-skills/
├── README.md                  # 项目主页（本文件）
├── SKILLS_INDEX.md            # 完整技能索引
├── LICENSE                    # MIT开源协议
├── requirement/               # 需求分析技能
│   ├── business-research/      # 业务需求调研
│   ├── requirement-doc/        # 需求文档编写
│   └── user-story/            # 用户故事映射
├── analysis/                  # 系统分析技能
│   ├── business-process/       # 业务流程分析
│   ├── data-flow/             # 数据流程分析
│   └── functional-model/       # 功能建模分析
├── architecture/              # 架构设计技能
│   ├── system-design/          # 总体架构设计
│   ├── tech-selection/         # 技术选型评估
│   ├── database-design/        # 数据库设计
│   └── api-design/             # 接口API设计
├── coding/                    # 编码实现技能
│   ├── frontend-dev/           # 前端开发实战
│   ├── backend-dev/            # 后端开发实战
│   ├── api-dev/               # API接口开发
│   ├── database-impl/          # 数据库实现
│   └── script-tools/           # 脚本工具开发
├── testing/                   # 测试验证技能
│   ├── unit-testing/           # 单元测试设计
│   ├── integration-testing/    # 集成测试实施
│   └── system-testing/         # 系统测试管理
└── docs/                      # 文档模板
    ├── templates/             # 文档模板
    └── guides/                # 使用指南
```

---

## Skill标准结构

每个Skill都包含标准文件：

```
skill-name/
├── SKILL.md              # 主要学习文档（必需）
├── forms.md              # 实践表单和检查清单（必需）
├── reference.md          # 技术参考文档（必需）
├── scripts/              # 示例代码目录
│   ├── main.py           # Python示例
│   ├── app.js            # JavaScript示例
│   └── Main.java         # Java示例
└── assets/               # 资源文件
    ├── diagrams/         # 图表
    └── templates/        # 模板
```

---

## 如何贡献

欢迎提交PR或Issue！贡献指南请查看 [docs/CONTRIBUTING.md](./docs/CONTRIBUTING.md)

---

## 致谢

- 灵感来源：[microwind/ai-skills](https://github.com/microwind/ai-skills)
- 参考项目：[Anthropic Skills](https://github.com/anthropics/skills)
- 工具支持：[ClawHub](https://clawhub.ai)

---

**萝卜飞出品 | 让AI更好地帮你打工！** 🚀
