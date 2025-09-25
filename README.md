# 项目概述

Claude Code和CodeX是AI编码界的TOP产品。对于AI编程工具，上下文工程（Prompt Engineering）是AI高效工作的关键因素。
本项目的基于个人的工作实践与思考，试图探索上下文工程的最佳实践。

---

## 上下文工程的构建

### 经验一：精简上下文文件

**精炼**是上下文的核心。庞大的上下文不光会浪费Token，而且会给AI理解用户意图带来困扰。注意力机制告诉我们，离当前的
工作任务越近的信息会获得更高的关注度。而庞大的上下文会造成注意力**失焦**，导致LLM无法聚焦到核心的提示信息上。

因此，将上下文信息进行分层和结构化，能够提升上下文的提示效率。

[Agent上下文最佳实践](agents-context-file-best-practice)

### 经验二：只提示必要的信息

**只提示必要信息**和**精炼**的出发点是一致的。如果只提示必要信息，那么上下文文件就会变得精炼。那么什么是必要信息？

**必要信息**是能够影响LLM默认行为的信息。如果一个行为是AI编程工具的默认行为，那么是不需要提示的。举个例子，AI编程工具的身份。
有些工程师会习惯性地在提示词中指定LLM的身份，这对于使用通用大模型来说是必要的，但是对于AI编程工具来说则是不必要的。因为编程
工具已经在内置的系统提示词中对LLM的角色、能力等做了很好的提示词引导。因此，一般情况下是不需要在上下文中指定LLM角色的。
但是对于某些特殊场景，指定LLM角色又是必要的。比如：希望LLM从UX工程师的角度来Review前端页面设计，这是默认身份不擅长的，此时
需要在对话中加入相关提示信息。此外，很多的`bash`命令，比如`git`的使用方法，也是无需提示的。

### 经验三：迭代式优化

很多时候，无法在项目的开始阶段确定哪些信息是**必要信息**，因此，上下文文件在初始时应当是非常简单的。随着项目的持续进行，
问题会逐步浮出水面，再逐渐添加更多的规则来**矫正**LLM的行为。

---

## 参考信息

- [PRD for claude code](https://www.chatprd.ai/resources/PRD-for-Claude-Code)
- [Prompt Engineering: The Art of Instructing AI](https://southbridge-research.notion.site/Prompt-Engineering-The-Art-of-Instructing-AI-2055fec70db181369002dcdea7d9e732)
- [Everyone Is Using Claude Code Sub-agents Wrong](https://www.youtube.com/watch?v=3564u77Vyqk)
