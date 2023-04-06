---
title: "Runtime for AIGC"
pubDate: "2023-04-06"
slug: "Necessary runtime for AIGC"
description: "Necessary runtime for AIGC."
hero: "/images/AIGC.jpeg"
tags: ["AIGC"]
layout: "../../layouts/BlogPostLayout.astro"
---

## Problems with AI-generated code
- Cannot be directly executed. The generated code is mostly just a fragment without a - complete environment, such as dependencies and so on.
- Poor debuggability: In some cases, AI-generated code may be difficult to debug and test because it can be hard for people to trace errors and logical mistakes in the code.
- Infrastructure issues: Non-business-related parts, such as HTTP ports, database access, etc.
- Limitations of existing solutions: Currently, the main solution for AI-generated code is using REPLs but they have limitations such as lack of code editing and saving functionality as well as limited support for programming languages and frameworks.

## What should runtime do?
- Environment isolation: providing independent running environments for each unit, allowing multiple units to run simultaneously in the same process without interfering with each other.
- Dependency management: through automated dependency analysis and management, developers do not need to manually install and manage dependencies.
- Debugging support: by providing an interactive console, developers can easily debug and test their code.
- Infrastructure support: provides support for basic infrastructure such as HTTP ports and database access, making it easier for developers to write and run business code.