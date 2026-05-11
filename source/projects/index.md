---
title: Projects
date: 2026-03-25
---

# Projects / 项目作品

A collection of what I build and explore. 这里记录我做过的项目、阶段性作品，以及一些从想法到可运行产品的实践。

## PocketLedger / 随心记账本
- Type: Vibe Coding Project / Open-source Desktop Tool
- Stack: React, Vite, Electron, local JSON storage
- Role: Product definition, experience direction, review, and iteration
- Description: A local-first accounting tool for students. It focuses on recording daily expense, income, transfer, account balance, monthly allowance, and spending structure without requiring login or cloud sync.
- Statement: This is a vibecoding project. I provided the product goal, usage scenarios, experience judgment, and acceptance direction; Codex helped turn the conversation into a PRD, UI, React + Electron implementation, README, release workflow, and GitHub-ready project.

### Key Features
- 支持支出、收入、转账三类交易。
- 支持微信、支付宝、银行卡、现金等账户管理。
- 支持自定义收入 / 支出分类。
- 首页展示本月生活费、已花、剩余、收入、支出、结余。
- 支持账单筛选、搜索、最近交易回看。
- 支持分类统计、账户支出分布、近 6 个月趋势图。
- 支持 JSON / CSV 导出，支持 JSON 导入。
- 支持账户余额校准和校准日志。
- 默认本地保存，不需要注册登录。

### Creation Process
这次创作不是先写代码，而是先把一个模糊想法收敛成可执行的产品：

1. 先明确定位：不是复杂理财平台，而是一个面向大学生日常生活费管理的安静账本。
2. 再写 PRD：确定核心用户、使用场景、数据模型、首页信息优先级、交易类型、账户系统、分类系统和导入导出能力。
3. 再做体验取舍：第一版坚持本地优先、单用户、无登录，先把“快速记一笔”和“这个月还剩多少钱”做好。
4. 再落到工程：使用 React + Vite 做界面，Electron 做桌面壳，本地 JSON 文件保存数据。
5. 最后做开源化：补 README、LICENSE、CI、Release 配置，让它不仅是一个 demo，而是可以继续维护和发布的开源项目。

我把这次项目看成一次 vibe coding 练习：重点不只是“让 AI 写代码”，而是学习如何把需求、判断、体验和验收标准表达清楚，再通过连续反馈把它压成一个真正能运行的工具。

## Blog Website
- Type: Personal Project
- Stack: Hexo, Butterfly
- Description: My personal blog for documenting technical learning and growth.

## C++ Learning Projects
- Type: Learning Project
- Stack: C++
- Description: Small practice projects and exercises built while learning C++.
