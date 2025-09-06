## Language
- Always response in Chinese

## Preference

### Dependency Management
- 使用uv来管理python依赖
- 使用npm管理javascript依赖

### Context Management

在项目目录下通过 ai_memeory 目录来管理上下文信息。ai_memory 目录结构如下:

```
project_root/
    `- ai_memory/
        `|- project/    # 项目信息. 阅读此目录来完整了解项目信息.
         |  `|- overview.md                    # 项目的概要信息: 1. 从业务层视角描述整个项目的信息。2. 目录结构。3. 技术栈
         |   |- backend.md                     # 后端的详细信息：1. 系统架构 2. 层次划分 3. 模块划分 4. 参考文档
         |   |- frontend.md                    # 前端的详细信息：1. 系统架构 2. 层次划分 3. 模块划分 4. 参考文档
         |   |- coding_standards.md            # 当前项目的编码规范
         |   |- research_cache.md              # 当前项目的 research 缓存
         |   `- api.md                         # 前后端交互的REST接口：1. 接口描述 2. 数据模型
         |
         `- features/   # directory for all feature development
            `|- 000_xxx-feature-description/   # feature 信息目录
             |  `|- requirements.md            # Describe the feature requirement
             |   |- design.md                  # Design docment based on the requirements.md
             |   |- plan.md                    # Development plan based on the design.md. It is a checklist for tracking task status
             |   |- issues.md                  # Issues found in developmenet
             |   `- tests/                     # Test scripts for this feature
             |
             |- 001_yyy-feature-description/
             `- ...
```

### Function Test

- 将测试脚本保存到项目根目录下的 test/ 目录
- 在测试完成后，删除临时测试脚本

### Design Principles
- When design, first review the current project structure, then think carefully how to integrate the new function.
- When design a new function, first check if the function has been implmentend in this project or by 3rd party library. **AVOID** redundant code.
- Consider data storage and persistent when design a new functionality.

#### Data Model Design
- **Schemas vs Entities 区分原则**: 在设计数据模型时，应当明确区分 Schemas 和 Entities：
  - **Schemas** (`schemas.py`): API 数据模型，使用 Pydantic，面向业务逻辑和外部接口，用于请求/响应数据验证和序列化
  - **Entities** (`entities.py`): 数据库数据模型，使用 SQLModel/SQLAlchemy，面向数据持久化，用于 ORM 表结构映射
- 避免在同一文件中混合 API 模型和数据库模型，保持清晰的架构分层

### Infrastructure Module Design Principles

**Core Rule**: Infrastructure services must be business-agnostic to achieve reusability and proper dependency relationships.

#### Correct Design Approach

1. **Technical Neutrality**
   - Use purely technical concepts (`USER_INPUTS`, `SYSTEM_OUTPUTS`, `TEMPLATES`)
   - Zero knowledge of domain-specific terminology in shared services
   - Business semantics handled entirely in the business/application layer

2. **Clear Dependency Hierarchy**
   ```
   Business Layer → Infrastructure Layer (unidirectional only)
   ```
   - Business layer adapts business requirements to technical parameters
   - Infrastructure provides generic, reusable technical capabilities

3. **Design Process**
   - Identify technical patterns across all use cases first
   - Extract purely technical categorization (avoid business context naming)
   - Validate against SOLID principles, especially Dependency Inversion
   - Test: Can this service support unknown future business requirements?

**Key Principle**: Business adaptation belongs in the business layer, not infrastructure. Proper dependency management trumps API convenience.