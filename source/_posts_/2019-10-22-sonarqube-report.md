---
title: 怎样基于 SonarQube 生成代码质量报告
---

[`SonarQube`]()是一个管理代码质量的开放平台，它可以自动地检查代码中的缺陷、漏洞，坏味道，重复度等，它可以与你现有的工作流无缝对接，可以扫描固定代码仓库分支，也可以对`Pull Request`进行扫描；支持所有常见的编程语言，还包括最新的`Kotlin`和`Swift`等语言；通过[`sonarlint`](https://www.sonarlint.org/)插件还可以与常见编辑器或`IDE`（比如`VS Code`，`Eclipse`等）进行集成，让你在编码时就能对代码进行实时检查。