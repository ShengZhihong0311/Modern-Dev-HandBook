# Modern-Dev-HandBook

> 现代 AI 原生开发者进阶指南：从代码到工程

## 简介

这是一本面向开发者的系统性技术指南，按照"一个项目的生命周期"从底层到高层构建技术认知体系。

## 目标读者

- 有一定编程基础，想系统性提升工程能力的开发者
- 希望理解工具背后原理，而非死记命令的开发者
- 正在从"写代码"进阶到"做工程"的开发者

## 目录

- [第一章：地基——环境隔离与依赖管理](docs/chapter-01-environment.md)
  - [1.0 开发环境基础：Windows、Linux 与 WSL](docs/chapter-01-environment.md#10-开发环境基础windows-linux-与-wsl) — 环境对比、IDE 界面详解
  - [隔离技术对比](docs/chapter-01-environment.md#隔离技术对比虚拟机虚拟环境容器本地) — 虚拟机、虚拟环境、容器、本地的区别
  - [WSL + 虚拟环境层级关系](docs/chapter-01-environment.md#wsl--虚拟环境的层级关系) — 为什么 WSL 里还需要 .venv
- [第二章：触角——终端（CLI）与数据流转](docs/chapter-02-terminal.md)
- [第三章：侦探——程序诊断与异常处理](docs/chapter-03-debugging.md)
- [第四章：时空——Git 版本控制与协作](docs/chapter-04-git.md)
- [第五章：封装——Docker 容器化与部署](docs/chapter-05-docker.md)
  - [本地启动 vs Docker 启动](docs/chapter-05-docker.md#58-本地启动-vs-docker-启动对比) — 两种启动方式的对比与选择
  - [Docker 热重载开发](docs/chapter-05-docker.md#59-docker-热重载开发) — 开发环境代码热更新配置
- [第六章：认知——AI 原生开发（Vibe Coding）](docs/chapter-06-ai-native.md)
  - [6.0 概念基础：范式与实践](docs/chapter-06-ai-native.md#60-概念基础范式与实践) — 理解"范式"和"实践"的含义
- [第七章：工程规范与职业操守](docs/chapter-07-engineering.md)
- [第八章：实战——不同开发环境的差异与注意事项](docs/chapter-08-environments.md)
  - 本地、云服务器、实验室服务器的对比与注意事项

## 写作风格

每个知识点包含：
1. **原理层**：为什么这样设计？解决什么问题？
2. **实战层**：具体命令、代码示例
3. **对比层**：不同方案的取舍

## 参与贡献

欢迎提交 Issue 和 Pull Request。

## 许可证

MIT License