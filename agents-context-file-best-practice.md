# 最佳实践：精简高效的上下文文件

## ❌ 问题：过大的上下文文件

**为什么巨大的上下文文件是有害的：**
1. **Token浪费** - 每次对话都要消耗大量token
2. **注意力稀释** - AI难以聚焦真正重要的信息
3. **性能下降** - 增加延迟，降低响应质量
4. **维护困难** - 信息过多难以保持更新
5. **成本增加** - 更多token = 更高费用

## ✅ 解决方案：分层上下文策略

### 1. 主上下文文件（500-800词）
<agents-context-file-structure>
# Project: E-Commerce API

## Tech Stack
- Node.js 20 + TypeScript 5.3 (strict mode)
- NestJS 10 + PostgreSQL 15
- Docker + pnpm (not npm!)

## Quick Start
```bash
pnpm install && docker-compose up -d
pnpm run start:dev  # http://localhost:3000
```

## Project Structure
- `src/modules/*` - Business modules (auth, payment, order)
- `src/core/*` - Framework code (don't modify)
- `src/shared/*` - Shared DTOs and utils

## Key Commands
- `pnpm test:watch` - Test during development
- `pnpm run migration:create` - New migration
- `pnpm run build` - Production build

## Critical Rules
1. NEVER commit .env files
2. All money calculations use Decimal type
3. Use transactions for multi-table updates
4. New endpoints need @ApiDoc decorator
5. Test coverage must be >80%

## Code Style
- Controllers: thin, delegates to services
- Services: business logic only
- Repositories: database queries only
- File naming: kebab-case.type.ts
- Commit format: type(scope): message

## Before You Code
- Check existing patterns in `src/modules/auth/*`
- Run tests first to ensure clean state
- Use dependency injection, not imports

## Common Issues
- "Cannot find module" → run `pnpm install`
- Migration fails → check Docker postgres is running
- Timeout errors → increase Jest timeout in test files

## Get Help
- Architecture decisions: `docs/adr/`
- API docs: http://localhost:3000/api
- Team conventions: `docs/team-guide.md`
</agents-context-file-structure>

### 2. 模块级上下文（每个200-300词）

**frontend/AGENTS.md**

<front-end-agent-context-file>
# Frontend Module

## Stack
- React 18 + Vite + Tailwind CSS
- Tanstack Query for API calls
- Zustand for state management

## Structure
- `components/` - Reusable UI components
- `pages/` - Route components
- `hooks/` - Custom React hooks
- `api/` - API client functions

## Rules
- Components use .tsx extension
- One component per file
- Props interfaces required
- Use Tailwind classes, no inline styles

## Testing
- Component tests with Testing Library
- Mock API calls with MSW
- Coverage target: 70%
</front-end-agent-context-file>

**backend/AGENTS.md**
<backend-agent-context-file>
# Backend Module

## Database Rules
- Tables: plural names (users, orders)
- All tables have: id, created_at, updated_at
- Soft deletes: add deleted_at
- Money: DECIMAL(10,2)

## API Standards
- RESTful routes only
- Response format: `{ success, data, error }`
- Pagination: `?page=1&limit=20`
- Auth: Bearer token in header

## Error Handling
- Throw HttpException with proper status
- Log errors with context
- Never expose internal errors to client
</backend-agent-context-file>

### 3. 任务特定指令（临时文件）

**当执行特定任务时创建：**
<temporary-agent-context-file>
# Task: Implement Stripe Payment

## Requirements
- Use Stripe SDK v14
- Support cards and Apple Pay
- Implement webhook for payment confirmation
- Store payment_intent_id in orders table

## Steps
1. Add stripe package
2. Create PaymentService
3. Add webhook endpoint
4. Update Order entity
5. Write integration tests

## Reference
- Stripe docs: https://stripe.com/docs/api
- Similar code: src/modules/email/email.service.ts

---
# 任务完成后删除此文件
</temporary-agent-context-file>

## 📊 推荐的上下文大小

| 文件类型 | 推荐大小 | 用途 |
|---------|---------|------|
| 主AGENTS.md | 500-800词 | 全局规则、快速参考 |
| 模块AGENTS.md | 200-300词 | 特定领域规则 |
| 任务指令 | 100-200词 | 临时任务上下文 |
| **总计** | **<1500词** | 保持高效 |

## 🎯 精简技巧

### 1. 使用链接而非内容
```markdown
❌ 错误：包含完整API文档
✅ 正确：API文档：http://localhost:3000/api
```

### 2. 示例胜于说明
```markdown
❌ 错误：长篇解释命名规则
✅ 正确：文件命名：user-profile.component.ts
```

### 3. 只包含异常情况
```markdown
❌ 错误：解释所有TypeScript特性
✅ 正确：注意：我们使用strict模式，不允许any类型
```

### 4. 使用模式参考
```markdown
❌ 错误：包含完整代码示例
✅ 正确：参考实现：src/modules/auth/*
```

### 5. 层级化组织
```markdown
/project
  AGENTS.md          # 500词 - 仅全局规则
  /frontend
    AGENTS.md        # 200词 - React特定
  /backend  
    AGENTS.md        # 200词 - API特定
  /database
    AGENTS.md        # 200词 - 数据库特定
```

## 🚀 实际效果对比

### 巨大上下文文件（5000+词）
- ❌ 每次请求消耗 $0.15-0.30
- ❌ 响应延迟增加 3-5秒
- ❌ AI经常忽略重要细节
- ❌ 需要经常提醒规则

### 优化后（<1000词）
- ✅ 每次请求仅 $0.03-0.05
- ✅ 快速响应（<1秒）
- ✅ AI准确遵循规则
- ✅ 更容易维护和更新

## 💡 核心原则

1. **少即是多** - 只包含真正影响每次交互的信息
2. **具体优于抽象** - 给出具体例子，而非冗长解释
3. **异常优于常规** - 只记录特殊规则，不记录标准做法
4. **引用优于复制** - 链接到文档，而非复制内容
5. **迭代优于完美** - 从小开始，根据需要添加

## 📝 检查清单

创建上下文文件时，问自己：
- [ ] 能否删除50%的内容而不失去关键信息？
- [ ] 是否有重复或显而易见的内容？
- [ ] 能否用一个例子代替一段解释？
- [ ] 这些信息是否每次交互都需要？
- [ ] 能否移到子文件夹的局部上下文中？

## 🎪 记住

> "完美的上下文文件不是没有什么可以添加的，而是没有什么可以删除的。" 
> 
> — 改编自 Antoine de Saint-Exupéry

最好的AGENTS.md是**活文档**：
- 从最小版本开始
- 观察AI的错误
- 只添加防止重复错误的规则
- 定期清理不再需要的规则

**目标：让AI像经验丰富的团队成员一样工作，而不是像读了整本百科全书的新手。**