# Google ADK Agent 开发指南

## 目录
1. [ADK概述](#adk概述)
2. [核心概念](#核心概念)
3. [开发环境设置](#开发环境设置)
4. [Agent架构设计](#agent架构设计)
5. [项目结构规范](#项目结构规范)
6. [工具系统](#工具系统)
7. [状态管理](#状态管理)
8. [事件机制](#事件机制)
9. [常用代码片段](#常用代码片段)
10. [测试与调试](#测试与调试)
11. [部署指南](#部署指南)
12. [最佳实践](#最佳实践)
13. [参考资料](#参考资料)

## ADK概述

Google Agent Development Kit (ADK) 是一个开源的、模块化的框架，专为构建和部署AI智能体而设计。ADK采用事件驱动架构，让Agent开发更像传统软件开发。

### 核心特性
- **多Agent设计**：支持复杂的多Agent协作系统
- **模型无关性**：支持Gemini、GPT、Claude等多种模型
- **工具生态丰富**：内置工具、自定义函数、第三方集成
- **流式处理**：支持实时双向音频/视频交互
- **部署灵活**：本地开发、Cloud Run、Vertex AI等多种部署选项

**官方文档**: [https://google.github.io/adk-docs/](https://google.github.io/adk-docs/)  
**GitHub仓库**: [https://github.com/google/adk-python](https://github.com/google/adk-python)

## 核心概念

### Agent类型体系

ADK提供三大类Agent类型：

#### 1. LLM Agents
基于大语言模型的智能体，用于推理和决策：
- **LlmAgent**: 标准的LLM驱动Agent
- **Agent**: LlmAgent的简化版本

#### 2. Workflow Agents  
专门控制执行流程的Agent，不使用LLM进行流程控制：
- **SequentialAgent**: 顺序执行子Agent
- **ParallelAgent**: 并发执行子Agent
- **LoopAgent**: 循环执行直到条件满足

#### 3. Custom Agents
继承BaseAgent实现自定义逻辑的Agent

**详细文档**: [https://google.github.io/adk-docs/agents/](https://google.github.io/adk-docs/agents/)

### 核心组件架构

```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│   Runner    │────│    Agent     │────│    Tools    │
│  (执行引擎)   │    │   (智能体)    │    │   (工具)     │
└─────────────┘    └──────────────┘    └─────────────┘
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
                  ┌──────────────┐
                  │   Session    │
                  │  (会话管理)    │
                  └──────────────┘
```

## 开发环境设置

### 安装ADK

```bash
# 创建虚拟环境
python -m venv .venv

# 激活虚拟环境
# macOS/Linux:
source .venv/bin/activate
# Windows:
.venv\Scripts\activate

# 安装ADK
pip install google-adk
```

### 环境配置

创建`.env`文件配置API密钥：

```bash
# Gemini via Vertex AI
GOOGLE_GENAI_USE_VERTEXAI=true
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1

# 或使用Google AI Studio
GOOGLE_API_KEY=your-api-key
```

**设置指南**: [https://google.github.io/adk-docs/get-started/quickstart/](https://google.github.io/adk-docs/get-started/quickstart/)

## Agent架构设计

### 单一Agent架构

适用于简单的对话型应用：

```python
from google.adk.agents import Agent
from google.adk.tools import google_search

root_agent = Agent(
    name="search_assistant",
    model="gemini-2.0-flash",
    instruction="You are a helpful assistant. Use Google Search when needed.",
    description="An assistant that can search the web.",
    tools=[google_search]
)
```

### 多Agent协作架构

适用于复杂工作流：

```python
from google.adk.agents import Agent, SequentialAgent

# 定义专门化的子Agent
extractor_agent = Agent(
    name="extractor",
    model="gemini-2.0-flash",
    instruction="Extract key information from text",
    output_key="extracted_data"
)

analyzer_agent = Agent(
    name="analyzer", 
    model="gemini-2.0-flash",
    instruction="Analyze the extracted data: {extracted_data}",
    output_key="analysis_result"
)

# 组装Sequential工作流
root_agent = SequentialAgent(
    name="data_processor",
    sub_agents=[extractor_agent, analyzer_agent],
    description="Extract and analyze data sequentially"
)
```

**架构文档**: [https://google.github.io/adk-docs/agents/multi-agents/](https://google.github.io/adk-docs/agents/multi-agents/)

### ParallelAgent并发处理

```python
from google.adk.agents import ParallelAgent, Agent

# 并发处理多个数据源
parallel_fetcher = ParallelAgent(
    name="data_fetcher",
    sub_agents=[
        Agent(name="api1_fetcher", output_key="api1_data", ...),
        Agent(name="api2_fetcher", output_key="api2_data", ...),
        Agent(name="api3_fetcher", output_key="api3_data", ...)
    ]
)
```

**并发文档**: [https://google.github.io/adk-docs/agents/workflow-agents/parallel-agents/](https://google.github.io/adk-docs/agents/workflow-agents/parallel-agents/)

## 项目结构规范

### ADK项目结构的严格要求

#### agent.py变量命名规范（重要！）

**Python项目中，主Agent变量必须严格命名为`root_agent`**（小写，下划线分隔），这是ADK框架的硬编码要求，不可配置：

```python
from google.adk.agents import Agent

# ✅ 正确：必须使用root_agent
root_agent = Agent(
    name="weather_assistant", 
    model="gemini-2.0-flash",
    instruction="You are a helpful weather assistant"
)

# ❌ 错误：其他变量名ADK无法自动发现
main_agent = Agent(...)      # 不会被识别
primary_agent = Agent(...)   # 不会被识别
my_agent = Agent(...)        # 不会被识别
```

**官方文档确认**: [ADK Quickstart](https://google.github.io/adk-docs/get-started/quickstart/) 和 [Google Cloud ADK Guide](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-development-kit/quickstart)

#### ADK自动发现机制

ADK通过以下步骤自动发现Agent：
1. 从工作目录递归扫描包含`__init__.py`的Python包
2. 在每个包中查找`agent.py`文件  
3. 使用反射机制查找名为`root_agent`的模块级变量
4. 验证变量是否为有效的ADK Agent实例
5. 将发现的Agent注册到框架运行时

### 单一Agent项目结构

适用于简单的单一功能Agent：

```
my_simple_agent/
├── __init__.py           # Python包标识 (必须)
├── agent.py              # 主Agent定义 (必须包含root_agent)
├── tools.py              # 自定义工具定义
├── config.py             # 配置管理
├── .env                  # 环境变量配置
├── .env.example          # 环境变量示例
├── requirements.txt      # Python依赖
├── README.md             # 项目说明
├── tests/                # 测试文件
│   ├── __init__.py
│   ├── test_agent.py
│   └── test_tools.py
├── evaluation/           # 评估相关
│   ├── eval_set.json
│   └── eval_agent.py
└── deployment/           # 部署脚本
    ├── deploy.py
    └── Dockerfile
```

### 多Agent项目结构

适用于复杂的多Agent协作系统，参考官方示例：[Travel Concierge](https://github.com/google/adk-samples/blob/main/agents/travel-concierge/README.md)

```
multi_agent_project/
├── pyproject.toml              # 项目配置
├── .env                       # 全局环境变量
├── agents/                    # Agent集合目录
│   ├── __init__.py            # 包标识
│   ├── coordinator/           # 主协调器Agent
│   │   ├── __init__.py
│   │   ├── agent.py          # 包含root_agent
│   │   ├── tools.py
│   │   └── config.py
│   ├── specialists/           # 专业Agent集合
│   │   ├── __init__.py
│   │   ├── web_search/
│   │   │   ├── __init__.py
│   │   │   ├── agent.py      # 包含root_agent
│   │   │   └── tools.py
│   │   ├── data_analysis/
│   │   │   ├── __init__.py
│   │   │   ├── agent.py      # 包含root_agent
│   │   │   ├── tools.py
│   │   │   └── models.py
│   │   └── content_generation/
│   │       ├── __init__.py
│   │       ├── agent.py      # 包含root_agent
│   │       └── templates/
├── shared/                    # 共享模块
│   ├── __init__.py
│   ├── schemas.py            # 数据模式定义
│   ├── utils.py              # 通用工具函数
│   ├── constants.py          # 常量定义
│   └── database.py           # 数据库连接
├── tools/                     # 全局工具模块
│   ├── __init__.py
│   ├── common/               # 通用工具
│   │   ├── __init__.py
│   │   ├── text_processing.py
│   │   └── file_handling.py
│   └── external/             # 外部API工具
│       ├── __init__.py
│       ├── weather_api.py
│       └── search_api.py
├── workflows/                 # 工作流Agent
│   ├── __init__.py
│   ├── sequential_workflow/
│   │   ├── __init__.py
│   │   └── agent.py          # 包含root_agent
│   └── parallel_workflow/
│       ├── __init__.py
│       └── agent.py          # 包含root_agent
├── tests/                     # 测试目录
│   ├── __init__.py
│   ├── test_agents/
│   └── test_tools/
└── deployment/                # 部署相关
    ├── docker/
    ├── kubernetes/
    └── scripts/
```

### 核心文件规范

#### agent.py主文件

每个Agent目录的`agent.py`文件必须包含`root_agent`变量：

```python
"""
主Agent定义文件
ADK会自动查找名为root_agent的变量作为入口点
"""
from google.adk.agents import Agent, SequentialAgent
from ..shared.utils import common_tool
from .tools import specialized_tool
from .config import get_model_config

# 定义子Agent（可以任意命名）
processor_agent = Agent(
    name="data_processor",
    model=get_model_config(),
    instruction="Process input data using specialized tools",
    tools=[specialized_tool],
    output_key="processed_result"
)

formatter_agent = Agent(
    name="result_formatter",
    model=get_model_config(),
    instruction="Format the processed result: {processed_result}",
    tools=[common_tool],
    output_key="final_output"
)

# 主Agent（必须命名为root_agent）
root_agent = SequentialAgent(
    name="my_workflow_agent",
    sub_agents=[processor_agent, formatter_agent],
    description="A complete data processing workflow"
)
```

#### 多Agent协调模式

```python
"""
协调器Agent - 管理多个专业Agent
"""
from google.adk.agents import LlmAgent
from ..specialists.web_search.agent import root_agent as search_agent
from ..specialists.data_analysis.agent import root_agent as analysis_agent
from ..specialists.content_generation.agent import root_agent as content_agent

# 主协调器Agent
root_agent = LlmAgent(
    name="multi_agent_coordinator",
    model="gemini-2.0-flash",
    description="Coordinates multiple specialist agents",
    instruction="""
    You are a coordinator that manages specialist agents.
    Route requests to appropriate agents based on the task type.
    """,
    sub_agents=[
        search_agent,     # 子Agent引用
        analysis_agent,   # 可以来自不同模块
        content_agent
    ]
)
```

#### shared/schemas.py - 数据模式定义

```python
"""
跨Agent共享的数据结构定义
"""
from typing import List, Optional
from dataclasses import dataclass
from datetime import datetime

@dataclass
class URLInfo:
    url: str
    source: str
    metadata: dict = None
    
@dataclass
class ProcessingResult:
    url_info: URLInfo
    title: str
    content: str
    status: str  # "success" | "error" | "pending"
    error_msg: Optional[str] = None
    timestamp: datetime = None
    
@dataclass
class WorkflowState:
    current_step: str
    progress: float
    results: List[ProcessingResult]
    errors: List[str]
```

#### shared/utils.py - 通用工具函数

```python
"""
可被多个Agent复用的工具函数
"""
import re
from typing import List, Dict, Any

def extract_urls(text: str) -> List[str]:
    """从文本中提取URL - 可被多个Agent共享"""
    pattern = r'https?://[^\s<>"{}|\\^`[\]]+'
    urls = re.findall(pattern, text)
    return list(set(urls))  # 去重

def validate_email(email: str) -> bool:
    """邮箱验证工具"""
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))

def format_response(data: Dict[str, Any], format_type: str = "json") -> str:
    """标准化响应格式"""
    if format_type == "json":
        import json
        return json.dumps(data, indent=2, ensure_ascii=False)
    elif format_type == "markdown":
        # 转换为Markdown格式
        pass
    return str(data)
```

### Agent间通信模式

#### 1. 通过shared state传递数据

```python
# Agent A 存储数据
processor_agent = Agent(
    name="data_processor",
    instruction="Extract and process data",
    output_key="extracted_data"  # 保存到shared state
)

# Agent B 读取数据  
analyzer_agent = Agent(
    name="data_analyzer",
    instruction="Analyze the extracted data: {extracted_data}",  # 从state读取
    output_key="analysis_result"
)

# 组装Sequential工作流
root_agent = SequentialAgent(
    name="data_pipeline",
    sub_agents=[processor_agent, analyzer_agent]
)
```

#### 2. 直接Agent引用模式

```python
"""
高级模式：直接引用其他模块中的Agent
适用于复杂多Agent系统
"""
from ..specialists.web_search.agent import root_agent as search_agent
from ..specialists.data_analysis.agent import root_agent as analysis_agent

# 协调器Agent可以直接调用专业Agent
root_agent = LlmAgent(
    name="coordinator",
    instruction="Route tasks to appropriate specialists",
    sub_agents=[search_agent, analysis_agent]
)
```

### 项目规模扩展策略

#### 小型项目（1-3个Agent）
```
simple_project/
├── __init__.py
├── agent.py              # 单一root_agent
├── tools.py
└── .env
```

#### 中型项目（4-10个Agent）  
```
medium_project/
├── main/
│   ├── __init__.py
│   └── agent.py          # 主协调器root_agent
├── processors/
│   ├── __init__.py  
│   └── agent.py          # 处理器root_agent
├── analyzers/
│   ├── __init__.py
│   └── agent.py          # 分析器root_agent
└── shared/
    ├── __init__.py
    ├── tools.py
    └── schemas.py
```

#### 大型项目（10+个Agent）
使用完整的多层级结构，参考前面的"多Agent项目结构"示例。

### 运行多Agent项目

#### Web UI模式
```bash
cd multi_agent_project/

# ADK会自动发现所有包含root_agent的模块
adk web

# 在浏览器中打开 http://localhost:8000
# 在下拉菜单中选择要运行的Agent：
# - coordinator (来自agents/coordinator/)
# - web_search (来自agents/specialists/web_search/)  
# - data_analysis (来自agents/specialists/data_analysis/)
# - sequential_workflow (来自workflows/sequential_workflow/)
```

#### 程序化运行特定Agent

```python
"""
在代码中运行特定的Agent
"""
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from agents.coordinator.agent import root_agent as coordinator
from agents.specialists.web_search.agent import root_agent as search_agent

# 运行协调器Agent
runner = Runner(
    agent=coordinator,
    app_name="multi_agent_app",
    session_service=InMemorySessionService()
)

# 或直接运行专业Agent
search_runner = Runner(
    agent=search_agent,
    app_name="search_app", 
    session_service=InMemorySessionService()
)
```

## 工具系统

### Function Tools (推荐用于简单逻辑)

```python
def extract_urls(text: str) -> list[str]:
    """从文本中提取URL
    
    Args:
        text: 包含URL的文本
        
    Returns:
        提取出的URL列表
    """
    import re
    url_pattern = r'https?://[^\s<>"{}|\\^`[\]]+'
    return re.findall(url_pattern, text)

# 直接传递给Agent
agent = Agent(
    name="url_extractor",
    tools=[extract_urls]  # ADK自动包装为Function Tool
)
```

### Built-in Tools

ADK提供多种预制工具：

```python
from google.adk.tools import google_search, code_interpreter
from google.adk.tools.bigquery import BigQueryToolset

# Google搜索
search_agent = Agent(
    name="researcher",
    tools=[google_search]
)

# 代码执行
coding_agent = Agent(
    name="coder", 
    tools=[code_interpreter]
)

# BigQuery集成
data_agent = Agent(
    name="data_analyst",
    tools=[BigQueryToolset(project_id="your-project")]
)
```

**工具文档**: [https://google.github.io/adk-docs/tools/built-in-tools/](https://google.github.io/adk-docs/tools/built-in-tools/)

### MCP集成

Model Context Protocol集成外部工具：

```python
# 需要先启动MCP Toolbox服务器
# 然后在Agent中引用MCP工具
from google.adk.tools.mcp import MCPClient

mcp_client = MCPClient("http://localhost:5000")
agent = Agent(
    name="mcp_agent",
    tools=[mcp_client.get_tools()]
)
```

**MCP文档**: [https://github.com/google/multitool](https://github.com/google/multitool)

## 状态管理

ADK的状态管理支持跨会话的数据持久化：

### 状态作用域

```python
session.state = {
    # 会话级状态 (默认)
    "current_task": "processing_urls",
    "progress": 75,
    
    # 用户级状态 (跨会话持久化)
    "user:name": "张三",
    "user:preferences": {"language": "zh-CN"},
    
    # 应用级状态 (所有用户共享)
    "app:version": "1.0.0",
    "app:feature_flags": {"new_ui": True},
    
    # 临时状态 (不持久化)
    "temp:calculation_cache": {"result": 42}
}
```

### 状态更新方式

#### 1. 使用output_key (推荐)

```python
agent = Agent(
    name="processor",
    instruction="Process the data and save result",
    output_key="processed_data"  # 自动保存Agent输出到state
)
```

#### 2. 使用EventActions手动更新

```python
from google.adk.events import Event, EventActions

# 手动创建状态更新事件
state_update = {"user:login_count": login_count + 1}
event = Event(
    author="system",
    actions=EventActions(state_delta=state_update)
)
```

#### 3. 在Tool/Callback中更新

```python
def update_progress(context, progress: int):
    """更新处理进度"""
    context.state["progress"] = progress
    context.state["last_update"] = datetime.now().isoformat()
```

**状态管理文档**: [https://google.github.io/adk-docs/sessions/state/](https://google.github.io/adk-docs/sessions/state/)

## 事件机制

ADK采用事件驱动架构，Events是所有组件间通信的标准格式：

### Event结构

```python
from google.adk.events import Event, EventActions

event = Event(
    author="agent_name",           # 事件来源
    content=Content(...),          # 消息内容
    actions=EventActions(          # 副作用指令
        state_delta={"key": "value"},
        transfer_to_agent="other_agent"
    ),
    invocation_id="session_123",   # 会话ID
    id="event_456",               # 事件ID
    timestamp=1640995200.0,        # 时间戳
    partial=False                  # 是否为部分结果
)
```

### 实时进度反馈

```python
class ProgressReportingAgent(BaseAgent):
    async def _run_async_impl(self, ctx) -> AsyncGenerator[Event, None]:
        total_tasks = 10
        
        for i in range(total_tasks):
            # 发送进度事件
            yield Event(
                author=self.name,
                content=types.Content(parts=[
                    types.Part(text=f"Processing task {i+1}/{total_tasks}")
                ]),
                partial=True  # 进度信息
            )
            
            # 执行任务
            result = await self.process_task(i)
            
        # 发送最终结果
        yield Event(
            author=self.name,
            content=types.Content(parts=[
                types.Part(text="All tasks completed")
            ]),
            partial=False  # 最终结果
        )
```

**事件文档**: [https://google.github.io/adk-docs/events/](https://google.github.io/adk-docs/events/)

## 常用代码片段

### 1. 基础Agent定义

```python
from google.adk.agents import Agent

# 最简单的Agent
simple_agent = Agent(
    name="simple_assistant",
    model="gemini-2.0-flash",
    instruction="You are a helpful assistant",
    description="A basic conversational agent"
)

# 带工具的Agent
from google.adk.tools import google_search

search_agent = Agent(
    name="search_assistant", 
    model="gemini-2.0-flash",
    instruction="Use Google Search to answer questions with current information",
    tools=[google_search]
)
```

### 2. Sequential工作流

```python
from google.adk.agents import Agent, SequentialAgent

# 定义工作流步骤
step1 = Agent(
    name="data_extractor",
    instruction="Extract data from user input",
    output_key="extracted_data"
)

step2 = Agent(
    name="data_processor", 
    instruction="Process the extracted data: {extracted_data}",
    output_key="processed_data"
)

step3 = Agent(
    name="result_formatter",
    instruction="Format results: {processed_data}",
    output_key="final_result"
)

# 组装工作流
workflow = SequentialAgent(
    name="data_pipeline",
    sub_agents=[step1, step2, step3],
    description="Complete data processing pipeline"
)
```

### 3. Parallel并发处理

```python
from google.adk.agents import ParallelAgent, SequentialAgent

# 并发获取多个数据源
parallel_fetcher = ParallelAgent(
    name="multi_source_fetcher",
    sub_agents=[
        Agent(name="source1", output_key="data1", ...),
        Agent(name="source2", output_key="data2", ...),
        Agent(name="source3", output_key="data3", ...)
    ]
)

# 合并结果
merger = Agent(
    name="data_merger",
    instruction="Merge all data: {data1}, {data2}, {data3}",
    output_key="merged_data"
)

# 完整流程：并发获取 → 合并
complete_flow = SequentialAgent(
    name="fetch_and_merge",
    sub_agents=[parallel_fetcher, merger]
)
```

### 4. 自定义Tool集成

```python
import pandas as pd

def process_excel(file_path: str) -> str:
    """处理Excel文件并返回Markdown格式表格
    
    Args:
        file_path: Excel文件路径
        
    Returns:
        Markdown格式的表格字符串
    """
    try:
        # 读取Excel文件
        df = pd.read_excel(file_path, sheet_name=None)
        
        markdown_tables = []
        for sheet_name, sheet_df in df.items():
            # 转换为Markdown
            markdown_table = sheet_df.to_markdown(index=False)
            markdown_tables.append(f"## {sheet_name}\n\n{markdown_table}")
            
        return "\n\n".join(markdown_tables)
    except Exception as e:
        return f"Error processing Excel file: {str(e)}"

# 使用工具
excel_processor = Agent(
    name="excel_processor",
    tools=[process_excel],
    instruction="Process Excel files and extract content"
)
```

### 5. 错误处理模式

```python
from google.adk.agents import BaseAgent
from google.adk.events import Event
from typing import AsyncGenerator

class RobustAgent(BaseAgent):
    async def _run_async_impl(self, ctx) -> AsyncGenerator[Event, None]:
        try:
            # 主要逻辑
            result = await self.process_data(ctx.session.state)
            
            yield Event(
                author=self.name,
                content=types.Content(parts=[
                    types.Part(text=f"Success: {result}")
                ])
            )
            
        except ValueError as e:
            # 特定错误处理
            yield Event(
                author=self.name,
                content=types.Content(parts=[
                    types.Part(text=f"输入数据格式错误: {str(e)}")
                ])
            )
            
        except Exception as e:
            # 通用错误处理
            yield Event(
                author=self.name,
                content=types.Content(parts=[
                    types.Part(text=f"处理失败，请稍后重试: {str(e)}")
                ])
            )
```

### 6. Session和State管理

```python
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

async def run_agent_with_session():
    # 创建Session服务
    session_service = InMemorySessionService()
    
    # 创建Runner
    runner = Runner(
        agent=root_agent,
        app_name="my_app",
        session_service=session_service
    )
    
    # 创建会话
    session = await session_service.create_session(
        app_name="my_app",
        user_id="user123",
        state={"initial_data": "some_value"}
    )
    
    # 发送消息
    content = types.Content(
        role='user', 
        parts=[types.Part(text="Hello, agent!")]
    )
    
    # 异步执行
    async for event in runner.run_async(
        user_id="user123",
        session_id=session.id,
        new_message=content
    ):
        if event.is_final_response():
            print(f"Agent: {event.content.parts[0].text}")
            
        # 处理实时事件
        if event.partial:
            print(f"Progress: {event.content.parts[0].text}")
```

## 测试与调试

### 本地开发和调试

```bash
# 启动开发UI
adk web

# 启动CLI交互
adk cli

# 运行测试
pytest tests/
```

### 单元测试示例

```python
import pytest
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types
from .agent import root_agent

@pytest.mark.asyncio
async def test_agent_basic_functionality():
    """测试Agent基本功能"""
    session_service = InMemorySessionService()
    runner = Runner(
        agent=root_agent,
        app_name="test_app",
        session_service=session_service
    )
    
    # 创建测试会话
    session = await session_service.create_session(
        app_name="test_app",
        user_id="test_user"
    )
    
    # 发送测试消息
    content = types.Content(
        role='user',
        parts=[types.Part(text="Test message")]
    )
    
    events = []
    async for event in runner.run_async(
        user_id="test_user",
        session_id=session.id,
        new_message=content
    ):
        events.append(event)
    
    # 验证结果
    final_events = [e for e in events if e.is_final_response()]
    assert len(final_events) > 0
    assert final_events[0].content.parts[0].text is not None

@pytest.mark.asyncio
async def test_sequential_workflow():
    """测试Sequential工作流"""
    from google.adk.agents import Agent, SequentialAgent
    
    # 创建测试工作流
    step1 = Agent(
        name="step1",
        model="gemini-2.0-flash",
        instruction="Extract numbers from input",
        output_key="numbers"
    )
    
    step2 = Agent(
        name="step2", 
        model="gemini-2.0-flash",
        instruction="Sum the numbers: {numbers}",
        output_key="result"
    )
    
    test_workflow = SequentialAgent(
        name="test_workflow",
        sub_agents=[step1, step2]
    )
    
    # 测试执行
    session_service = InMemorySessionService()
    runner = Runner(
        agent=test_workflow,
        app_name="test_workflow_app",
        session_service=session_service
    )
    
    session = await session_service.create_session(
        app_name="test_workflow_app",
        user_id="test_user"
    )
    
    content = types.Content(
        role='user',
        parts=[types.Part(text="Process these numbers: 1, 2, 3, 4, 5")]
    )
    
    final_result = None
    async for event in runner.run_async(
        user_id="test_user",
        session_id=session.id,
        new_message=content
    ):
        if event.is_final_response():
            final_result = event.content.parts[0].text
            
    assert final_result is not None
    # 可以进一步验证结果的正确性
```

### 集成测试和评估

```python
"""
评估Agent性能的示例
"""
from google.adk.evaluation import AgentEvaluator
import json

def create_evaluation_dataset():
    """创建评估数据集"""
    return [
        {
            "input": "Extract URLs from this text: Visit https://example.com and https://test.org",
            "expected_output": ["https://example.com", "https://test.org"],
            "metadata": {"task_type": "url_extraction"}
        },
        {
            "input": "Process the following data and return JSON format",
            "expected_output": {"status": "success", "data": []},
            "metadata": {"task_type": "data_processing"}
        }
    ]

async def evaluate_agent():
    """评估Agent性能"""
    evaluator = AgentEvaluator(
        agent=root_agent,
        evaluation_dataset=create_evaluation_dataset()
    )
    
    results = await evaluator.evaluate()
    
    print(f"Evaluation Results:")
    print(f"Success Rate: {results.success_rate:.2%}")
    print(f"Average Response Time: {results.avg_response_time:.2f}s")
    print(f"Failed Cases: {len(results.failed_cases)}")
    
    return results
```

**调试文档**: [https://google.github.io/adk-docs/get-started/quickstart/](https://google.github.io/adk-docs/get-started/quickstart/)

## 部署指南

### 本地开发部署

```bash
# 方式1: Web UI
adk web

# 方式2: CLI
adk cli  

# 方式3: API Server
adk api_server
```

### 容器化部署

创建`Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    pandoc \
    && rm -rf /var/lib/apt/lists/*

# 安装Python依赖
COPY requirements.txt .
RUN pip install -r requirements.txt

# 复制代码
COPY . .

# 暴露端口
EXPOSE 8000

# 启动API服务器
CMD ["python", "-m", "google.adk.api_server", "--agent", "agent.py"]
```

### Vertex AI部署

```python
# deployment/deploy.py
from vertexai.preview.reasoning_engines import AdkApp
import vertexai

# 初始化Vertex AI
vertexai.init(project="your-project", location="us-central1")

# 部署Agent
app = AdkApp(agent=root_agent)
remote_agent = app.deploy()

print(f"Deployed agent: {remote_agent.resource_name}")
```

### Cloud Run部署示例

```yaml
# cloudbuild.yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/my-agent:$BUILD_ID', '.']
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/my-agent:$BUILD_ID']
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'run'
      - 'deploy'
      - 'my-agent'
      - '--image=gcr.io/$PROJECT_ID/my-agent:$BUILD_ID'
      - '--platform=managed'
      - '--region=us-central1'
      - '--allow-unauthenticated'
```

**部署文档**: [https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/develop/adk](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/develop/adk)

## 最佳实践

### 1. Agent设计原则

- **单一职责**: 每个Agent专注一个明确的任务
- **状态管理**: 合理使用output_key在Agent间传递数据
- **错误处理**: 分层处理错误，提供有意义的错误消息
- **可测试性**: 设计时考虑单元测试和集成测试

### 2. 工具选择指南

- **简单逻辑**: 使用Function Tools
- **复杂集成**: 使用Built-in Tools或MCP
- **性能要求高**: 优先考虑Built-in Tools
- **第三方API**: 使用MCP或自定义Tools

### 3. 性能优化

- **并发处理**: 对独立任务使用ParallelAgent
- **状态缓存**: 合理使用会话状态避免重复计算
- **资源管理**: 及时释放外部资源连接
- **批处理**: 对大量相似任务使用批处理模式

### 4. 安全考虑

- **输入验证**: 在Tool中验证所有外部输入
- **权限控制**: 限制Agent的系统访问权限
- **数据隐私**: 避免在日志中记录敏感信息
- **错误信息**: 不要暴露系统内部实现细节

### 5. 监控和日志

```python
import logging

# 配置日志
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def monitored_tool(data: str) -> str:
    """带监控的工具函数"""
    logger.info(f"Processing data: {data[:100]}...")
    
    try:
        result = process_data(data)
        logger.info("Processing completed successfully")
        return result
    except Exception as e:
        logger.error(f"Processing failed: {str(e)}")
        raise

# 使用回调函数监控Agent执行
class MonitoringCallback:
    def on_agent_start(self, agent_name: str):
        logger.info(f"Agent {agent_name} started")
        
    def on_agent_complete(self, agent_name: str, result: str):
        logger.info(f"Agent {agent_name} completed: {result[:100]}...")
        
    def on_error(self, agent_name: str, error: Exception):
        logger.error(f"Agent {agent_name} failed: {str(error)}")

# 在Agent中使用回调
root_agent = Agent(
    name="monitored_agent",
    callbacks=[MonitoringCallback()]
)
```

### 6. 生产环境配置

```python
"""
生产环境配置示例
"""
import os
from google.adk.sessions import DatabaseSessionService
from google.adk.runners import Runner

def create_production_runner():
    """创建生产环境Runner"""
    
    # 使用数据库会话服务
    session_service = DatabaseSessionService(
        db_url=os.getenv("DATABASE_URL")
    )
    
    # 配置Runner
    runner = Runner(
        agent=root_agent,
        app_name=os.getenv("APP_NAME", "production_app"),
        session_service=session_service,
        # 生产环境配置
        max_concurrent_sessions=100,
        session_timeout=3600,  # 1小时
        enable_logging=True,
        log_level="INFO"
    )
    
    return runner
```

### 7. 错误处理最佳实践

```python
"""
完整的错误处理示例
"""
from google.adk.agents import BaseAgent
from google.adk.events import Event
from typing import AsyncGenerator
import logging

class ProductionAgent(BaseAgent):
    async def _run_async_impl(self, ctx) -> AsyncGenerator[Event, None]:
        try:
            # 输入验证
            user_input = ctx.session.state.get("user_input", "")
            if not user_input.strip():
                yield Event(
                    author=self.name,
                    content=types.Content(parts=[
                        types.Part(text="请提供有效的输入内容")
                    ])
                )
                return
            
            # 业务逻辑处理
            result = await self.process_business_logic(user_input)
            
            # 成功响应
            yield Event(
                author=self.name,
                content=types.Content(parts=[
                    types.Part(text=f"处理完成: {result}")
                ]),
                actions=EventActions(
                    state_delta={"last_result": result, "status": "success"}
                )
            )
            
        except ValidationError as e:
            # 输入验证错误
            yield Event(
                author=self.name,
                content=types.Content(parts=[
                    types.Part(text=f"输入格式错误: {str(e)}")
                ]),
                actions=EventActions(
                    state_delta={"status": "input_error", "error": str(e)}
                )
            )
            
        except NetworkError as e:
            # 网络错误
            yield Event(
                author=self.name,
                content=types.Content(parts=[
                    types.Part(text="网络连接出现问题，请稍后重试")
                ]),
                actions=EventActions(
                    state_delta={"status": "network_error", "retry_needed": True}
                )
            )
            
        except Exception as e:
            # 未预期错误
            logging.error(f"Unexpected error in {self.name}: {str(e)}")
            yield Event(
                author=self.name,
                content=types.Content(parts=[
                    types.Part(text="处理出现异常，请联系技术支持")
                ]),
                actions=EventActions(
                    state_delta={"status": "system_error", "error_id": str(uuid.uuid4())}
                )
            )
```

## 参考资料

### 官方文档
- **ADK首页**: [https://google.github.io/adk-docs/](https://google.github.io/adk-docs/)
- **Python SDK**: [https://github.com/google/adk-python](https://github.com/google/adk-python)
- **快速开始**: [https://google.github.io/adk-docs/get-started/quickstart/](https://google.github.io/adk-docs/get-started/quickstart/)
- **Agent概念**: [https://google.github.io/adk-docs/agents/](https://google.github.io/adk-docs/agents/)
- **工具系统**: [https://google.github.io/adk-docs/tools/](https://google.github.io/adk-docs/tools/)
- **会话管理**: [https://google.github.io/adk-docs/sessions/](https://google.github.io/adk-docs/sessions/)
- **事件机制**: [https://google.github.io/adk-docs/events/](https://google.github.io/adk-docs/events/)
- **Sequential Agent**: [https://google.github.io/adk-docs/agents/workflow-agents/sequential-agents/](https://google.github.io/adk-docs/agents/workflow-agents/sequential-agents/)
- **Parallel Agent**: [https://google.github.io/adk-docs/agents/workflow-agents/parallel-agents/](https://google.github.io/adk-docs/agents/workflow-agents/parallel-agents/)
- **自定义Agent**: [https://google.github.io/adk-docs/agents/custom-agents/](https://google.github.io/adk-docs/agents/custom-agents/)
- **多Agent系统**: [https://google.github.io/adk-docs/agents/multi-agents/](https://google.github.io/adk-docs/agents/multi-agents/)

### 示例代码
- **官方示例**: [https://github.com/google/adk-samples](https://github.com/google/adk-samples)
- **学术研究Agent**: [https://github.com/google/adk-samples/tree/main/python/agents/academic-research](https://github.com/google/adk-samples/tree/main/python/agents/academic-research)
- **数据科学Agent**: [https://github.com/google/adk-samples/tree/main/python/agents/data-science](https://github.com/google/adk-samples/tree/main/python/agents/data-science)
- **客服Agent**: [https://github.com/google/adk-samples/tree/main/python/agents/customer-service](https://github.com/google/adk-samples/tree/main/python/agents/customer-service)
- **Bug助手Agent**: [https://github.com/google/adk-samples/tree/main/python/agents/software-bug-assistant](https://github.com/google/adk-samples/tree/main/python/agents/software-bug-assistant)
- **旅行助手Agent**: [https://github.com/google/adk-samples/tree/main/python/agents/travel-concierge](https://github.com/google/adk-samples/tree/main/python/agents/travel-concierge)
- **FOMC研究Agent**: [https://github.com/google/adk-samples/tree/main/python/agents/fomc-research](https://github.com/google/adk-samples/tree/main/python/agents/fomc-research)

### 教程和博客
- **官方博客**: [https://developers.googleblog.com/en/agent-development-kit-easy-to-build-multi-agent-applications/](https://developers.googleblog.com/en/agent-development-kit-easy-to-build-multi-agent-applications/)
- **DataCamp教程**: [https://www.datacamp.com/tutorial/agent-development-kit-adk](https://www.datacamp.com/tutorial/agent-development-kit-adk)
- **完整指南**: [https://www.siddharthbharath.com/the-complete-guide-to-googles-agent-development-kit-adk/](https://www.siddharthbharath.com/the-complete-guide-to-googles-agent-development-kit-adk/)
- **Medium深度分析**: [https://medium.com/@d3xvn/exploring-googles-agent-development-kit-adk-71a27a609920](https://medium.com/@d3xvn/exploring-googles-agent-development-kit-adk-71a27a609920)
- **多Agent示例**: [https://medium.com/@imranburki.ib/multi-agent-example-using-googles-agent-development-kit-adk-500312361ebb](https://medium.com/@imranburki.ib/multi-agent-example-using-googles-agent-development-kit-adk-500312361ebb)

### 部署和集成
- **Vertex AI部署**: [https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/develop/adk](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/develop/adk)
- **会话管理**: [https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/sessions/manage-sessions-adk](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/sessions/manage-sessions-adk)
- **BigQuery集成**: [https://cloud.google.com/blog/products/ai-machine-learning/bigquery-meets-google-adk-and-mcp](https://cloud.google.com/blog/products/ai-machine-learning/bigquery-meets-google-adk-and-mcp)
- **KYC工作流**: [https://cloud.google.com/blog/products/ai-machine-learning/build-kyc-agentic-workflows-with-googles-adk](https://cloud.google.com/blog/products/ai-machine-learning/build-kyc-agentic-workflows-with-googles-adk)

### 工具和扩展
- **MCP协议**: [https://github.com/google/multitool](https://github.com/google/multitool)
- **ADK Web UI**: [https://github.com/google/adk-web](https://github.com/google/adk-web)
- **Java SDK**: [https://github.com/google/adk-java](https://github.com/google/adk-java)
- **社区项目集合**: [https://github.com/Sri-Krishna-V/awesome-adk-agents](https://github.com/Sri-Krishna-V/awesome-adk-agents)

---

这份开发指南涵盖了Google ADK的核心概念、项目结构要求、实用代码片段和最佳实践。通过遵循这些指导原则，开发者可以构建出高质量、可维护的Agent应用程序。

## 重要提醒

1. **root_agent变量命名不可配置** - 这是ADK框架的硬编码要求
2. **每个agent.py都需要包含__init__.py** - 确保Python包结构正确
3. **多Agent项目中每个模块都要有独立的root_agent** - 即使作为子Agent使用
4. **优先使用official Built-in Tools** - 性能和稳定性更好
5. **状态管理使用output_key** - 这是Agent间数据传递的标准方式
