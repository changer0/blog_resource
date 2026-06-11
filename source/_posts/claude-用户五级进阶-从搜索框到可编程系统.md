---
title: Claude 用户五级进阶：从搜索框到可编程系统
source: https://x.com/elliotchen100/status/2056560995305390146?s=46
author:
  - "[[@elliotchen100]]"
published: 2026-05-19
created: 2026-05-20
description: 先把 credit 放前面：这篇整理来自 NerdHack / Nate Herk 的 YouTube 视频《Every Level of Claude Explained in 21 Minutes》。https://www.youtube.com/watch?v=ZRb7D6...
tags:
  - clippings
date: 2026-06-11 10:40:36
---

![图像](https://pbs.twimg.com/media/HIpSudxbgAAGdz4?format=jpg&name=large)

先把 credit 放前面：这篇整理来自 NerdHack / Nate Herk 的 YouTube 视频《Every Level of Claude Explained in 21 Minutes》。

[https://www.youtube.com/watch?v=ZRb7D6R64hM&t=17s](https://www.youtube.com/watch?v=ZRb7D6R64hM&t=17s)

最近看了 Nate Herk 这个 21 分钟视频，受益匪浅。他自己在 Claude 里泡了 400 小时，把用户分成了 5 级。下面是我看完之后的整理和思考，credit 全归 Nate。分级本身没什么稀奇，真正值得讲的是「卡点」。每一级跨到下一级，卡住的往往不是技能，而是心智模型。

![图像](https://pbs.twimg.com/media/HIpFuJZbwAAD3rr?format=jpg&name=large)

Level 1：一次性问答能解决当下问题，但不会形成长期杠杆。

Level 1：把 Claude 当搜索引擎

典型用法是一次性问答：你问，它答。关掉页面，关系结束。下一次再来，又从零开始解释背景。这一层不是不好，大部分人第一次接触 AI 本来就是从「更会说话的搜索框」开始。但如果一直停在这里，Claude 只能回答当下问题，不能持续理解你是谁、你在做什么、你怎么判断好坏。卡点是：没有把 Claude 看成一个可以沉淀上下文的工作系统。

![图像](https://pbs.twimg.com/media/HIpFwzYa4AA_xOk?format=jpg&name=large)

Level 2：记忆、Project 和连接器，让上下文开始沉淀。

Level 2：让 Claude 记住你

这一层开始用 memory，开始建 project，开始把 Slack、Google Drive 这些连接器接进来。重点不是功能本身，而是心智模型变了：上下文不再只存在于这一轮对话里，而是可以被积累、被复用、被组织。大部分 ChatGPT 用户被训练成了「一次性提问」的肌肉记忆，根本不知道可以让模型记住自己。Level 1 到 Level 2，卡在「记忆」。

![图像](https://pbs.twimg.com/media/HIpFzeDbcAANEPW?format=jpg&name=large)

Level 3：把重复任务交给 Claude 自己跑，效率开始成倍放大。

Level 3：让 Claude 自己跑

Level 3 引入 Cowork、定时任务、手机远程、Claude Design。本质上是从「我驱动 Claude」变成「Claude 自己跑」。Level 2 的人在想：「这件事我让 Claude 帮我做。」Level 3 的人在想：「这类事我让 Claude 每天自己做。」一字差，效率差一个数量级。Nate 把 Claude Design 叫做「Figma killer」，我不同意。它取代不了 Figma，但它让你不需要为了 0.1 版本去找设计师，这才是真正的杀点。Level 2 到 Level 3，卡在「自动化」。

![图像](https://pbs.twimg.com/media/HIpF2HcbEAA_KSw?format=jpg&name=large)

Level 4：从功能使用进入系统设计，上下文就是杠杆。

Level 4：上下文工程

这一级才是真正的分水岭。L1 到 L3 都是在「使用界面」，L4 开始是在「设计系统」。claude.md 不是文档，是给 Claude 看的 system prompt，写好它顶得上你做半小时口头解释。Shift+Tab 按两下进 plan mode，看上去是高级功能，其实任何需求模糊的任务都该先 plan，越早越省 token。/rewind 一键回滚，是 Claude 最被低估的命令，比重新写一遍 prompt 快太多。sub agents 并行跑，不是越多越好，关键是把角色切干净：一个查资料，一个写代码，互不污染。worktree 多 git 分支同时干，最大的价值不是速度，是不再因为「先做哪个」消耗决策力。MCP 听起来很玄，本质就是结构化的函数调用。工具不在多，在能不能被 Claude 自然挑出来用。到这一级你已经不是「用 Claude」，是在「设计 Claude 怎么帮你」。Level 3 到 Level 4，卡在「上下文工程」。

![图像](https://pbs.twimg.com/media/HIpF4xCbAAACtED?format=jpg&name=large)

Level 5：从有人盯着的工作流，走向无人值守的生产系统。

Level 5：不再亲自坐镇

Architect 级是 Cloud routines、lifecycle hooks、headless mode、Agent SDK。Claude 从「我打开它它才工作」变成「它一直在跑」。但这一级得有真实的生产需求，不然没意义。Nate 同时挂 5 个 session 听上去很猛，但对 95% 的人是过拟合。不要为了到 Level 5 而到 Level 5。先把 Level 4 打透。Level 4 到 Level 5，卡在「不再亲自坐镇」。

写在最后：Level 1 到 Level 3 是「会用」，Level 4 开始是「会想」。99% 的人不是技能不够，是没意识到 Claude 已经从一个 chat 工具变成一个能被「编程」的系统。你给它多少结构，它就还你多少杠杆。

注意：此推文是我和 Codex 共同