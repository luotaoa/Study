# 🤖 Agentic RAG 智能客服项目完整指南

## 从 0 到 1 构建生产级智能客服系统

---

## 📋 目录

1. [项目概述](#项目概述)
2. [技术架构](#技术架构)
3. [快速开始](#快速开始)
4. [实现方案](#实现方案)
5. [部署指南](#部署指南)
6. [最佳实践](#最佳实践)

---

## 项目概述

### 什么是 Agentic RAG？

**普通 RAG：**
```
用户提问 → 检索知识库 → 生成回答
```

**Agentic RAG（智能体增强）：**
```
用户提问
    ↓
Agent 分析决策
    ├─ 需要知识？ → 检索知识库
    ├─ 需要数据？ → 调用工具API
    ├─ 需要计算？ → 执行计算
    └─ 需要人工？ → 转接客服
    ↓
智能生成回答
```

### 核心能力

| 能力           | 说明                | 示例                     |
| -------------- | ------------------- | ------------------------ |
| 🔍 **知识检索** | RAG技术检索知识库   | FAQ、产品手册、政策文档  |
| 🔧 **工具调用** | 调用外部系统API     | 订单查询、库存检查、支付 |
| 🧠 **智能决策** | Agent自主判断和规划 | 意图识别、流程控制       |
| 💬 **多轮对话** | 记住上下文          | 信息收集、追问澄清       |
| 👤 **人工转接** | 复杂问题升级        | 投诉处理、特殊需求       |

---

## 技术架构

### 系统架构图

```
┌─────────────────────────────────────────────────┐
│            用户界面层                           │
│        (Web/微信/App/电话)                      │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────┴──────────────────────────────┐
│           Agent 智能决策层                      │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │意图识别 │  │工具选择 │  │流程控制 │        │
│  └─────────┘  └─────────┘  └─────────┘        │
└──────┬────────────┬────────────┬───────────────┘
       │            │            │
  ┌────┴───┐  ┌────┴───┐  ┌─────┴────┐
  │        │  │        │  │          │
┌─┴─┐  ┌──┴─┐┌┴─────┐┌─┴────────┐┌─┴───────┐
│RAG│  │工具││对话  ││业务系统  ││人工转接 │
│   │  │调用││记忆  ││API       ││         │
└───┘  └────┘└──────┘└──────────┘└─────────┘
  │      │      │        │
┌─┴──────┴──────┴────────┴──────────────────────┐
│            底层服务层                          │
│  • 向量数据库 (Milvus/Chroma)                 │
│  • 大语言模型 (GPT-4/GLM-4/Ollama)            │
│  • 缓存服务 (Redis)                           │
│  • 数据库 (MySQL/PostgreSQL)                  │
└───────────────────────────────────────────────┘
```

### 技术选型

#### 方案对比

| 技术栈        | 难度  | 成本 | 灵活性 | 推荐场景     |
| ------------- | ----- | ---- | ------ | ------------ |
| **自己实现**  | ⭐⭐⭐⭐⭐ | 💰    | 🔧🔧🔧🔧🔧  | 学习理解原理 |
| **LangChain** | ⭐⭐⭐   | 💰💰   | 🔧🔧🔧🔧   | 快速开发     |
| **LangGraph** | ⭐⭐⭐⭐  | 💰💰   | 🔧🔧🔧🔧🔧  | 复杂Agent    |
| **RAGFlow**   | ⭐     | 💰💰💰  | 🔧🔧     | 快速上线     |

---

## 快速开始

### 环境要求

```bash
Python >= 3.9
pip >= 21.0
```

### 安装依赖

#### 方案一：简单版（无需API）
```bash
pip install numpy
```

#### 方案二：标准版（需要LLM API）
```bash
pip install langchain langchain-openai chromadb python-dotenv
```

#### 方案三：高级版（LangGraph）
```bash
pip install langgraph langchain langchain-openai chromadb
```

### 配置 API Key

创建 `.env` 文件：
```env
# OpenAI API（付费）
OPENAI_API_KEY=sk-...
OPENAI_BASE_URL=https://api.openai.com/v1

# 或使用智谱AI（国内，有免费额度）
ZHIPU_API_KEY=...

# 或使用本地模型（免费）
# 使用 Ollama 运行本地模型
# 安装：curl -fsSL https://ollama.com/install.sh | sh
# 运行：ollama run llama2
```

---

## 实现方案

### 方案一：简单版（学习用）⭐⭐⭐⭐⭐

**特点：**
- ✅ 完全自己实现
- ✅ 无需外部API
- ✅ 代码简单易懂
- ✅ 适合学习原理

**核心代码结构：**

```python
class SimpleCustomerServiceAgent:
    def __init__(self):
        self.knowledge_base = SimpleKnowledgeBase()
        self.conversation_history = []
        self.tools = {
            "query_order": self.query_order,
            "check_product": self.check_product
        }
    
    def classify_intent(self, message):
        """简单的意图识别（关键词匹配）"""
        if "订单" in message:
            return "query_order"
        elif "产品" in message:
            return "product_inquiry"
        # ...
    
    def process(self, user_message):
        """处理用户消息"""
        intent = self.classify_intent(user_message)
        
        if intent == "query_order":
            result = self.tools["query_order"](user_message)
        elif intent == "product_inquiry":
            result = self.knowledge_base.search(user_message)
        
        return result
```

**完整代码：** 见 `Agentic-RAG智能客服完整实战.ipynb`

---

### 方案二：标准版（LangChain）⭐⭐⭐⭐

**特点：**
- ✅ 使用真实LLM
- ✅ 向量化知识库
- ✅ 语义检索
- ✅ 快速开发

**核心代码：**

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings
from langchain.llms import ChatOpenAI

class LangChainCustomerService:
    def __init__(self, api_key):
        # 1. 初始化LLM
        self.llm = ChatOpenAI(api_key=api_key, model="gpt-4")
        
        # 2. 创建向量知识库
        texts = self.load_documents()
        splitter = RecursiveCharacterTextSplitter(
            chunk_size=500,
            chunk_overlap=50
        )
        chunks = splitter.split_documents(texts)
        
        self.vectorstore = Chroma.from_documents(
            documents=chunks,
            embedding=OpenAIEmbeddings(api_key=api_key)
        )
        
        # 3. 定义工具
        self.tools = self.create_tools()
    
    def create_tools(self):
        """创建Agent可用的工具"""
        return [
            {
                "name": "search_knowledge",
                "description": "搜索公司知识库",
                "function": self.search_knowledge
            },
            {
                "name": "query_order",
                "description": "查询订单状态",
                "function": self.query_order
            }
        ]
    
    def process(self, user_message):
        """处理用户消息"""
        # 使用LLM进行意图理解和决策
        prompt = f"""
        用户消息：{user_message}
        
        可用工具：{self.tools}
        
        请分析用户意图，选择合适的工具，生成回答。
        """
        
        response = self.llm.predict(prompt)
        return response
```

---

### 方案三：高级版（LangGraph）⭐⭐⭐⭐⭐

**特点：**
- ✅ 复杂工作流
- ✅ 条件分支
- ✅ 循环支持
- ✅ 状态管理
- ✅ 可视化流程

**工作流设计：**

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class AgentState(TypedDict):
    messages: list
    user_query: str
    intent: str
    tool_result: str
    final_answer: str

# 定义节点
def intent_classifier(state): ...
def tool_selector(state): ...
def tool_executor(state): ...
def knowledge_retriever(state): ...
def answer_generator(state): ...

# 构建图
workflow = StateGraph(AgentState)

workflow.add_node("intent_classifier", intent_classifier)
workflow.add_node("tool_selector", tool_selector)
workflow.add_node("tool_executor", tool_executor)
workflow.add_node("answer_generator", answer_generator)

# 设置流程
workflow.set_entry_point("intent_classifier")
workflow.add_edge("intent_classifier", "tool_selector")
workflow.add_conditional_edges(
    "tool_selector",
    should_use_tool,  # 路由函数
    {
        "use_tool": "tool_executor",
        "use_knowledge": "knowledge_retriever"
    }
)
workflow.add_edge("tool_executor", "answer_generator")
workflow.add_edge("answer_generator", END)

agent = workflow.compile()
```

---

## 部署指南

### 1. 本地开发环境

```bash
# 创建项目目录
mkdir agentic-rag-customer-service
cd agentic-rag-customer-service

# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 安装依赖
pip install -r requirements.txt

# 配置环境变量
cp .env.example .env
# 编辑 .env 填入API keys

# 运行
python main.py
```

### 2. Docker 部署

**创建 Dockerfile：**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "main.py"]
```

**docker-compose.yml：**
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    volumes:
      - ./data:/app/data
      - ./logs:/app/logs
  
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
  
  postgres:
    image: postgres:14
    environment:
      POSTGRES_DB: customer_service
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

**启动：**
```bash
docker-compose up -d
```

### 3. 生产环境部署

#### 方案A：云服务（推荐）

**阿里云/腾讯云：**
```bash
# 1. 购买云服务器（ECS）
# 2. 配置安全组（开放端口）
# 3. 安装Docker
# 4. 部署应用
# 5. 配置域名和SSL证书
```

#### 方案B：Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: customer-service
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: your-registry/customer-service:latest
        env:
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: api-keys
              key: openai
```

---

## 最佳实践

### 1. 知识库管理

#### 文档组织
```
knowledge_base/
├── products/          # 产品说明
│   ├── iphone.txt
│   └── macbook.txt
├── policies/          # 公司政策
│   ├── return.txt
│   └── shipping.txt
└── faq/              # 常见问题
    └── general.txt
```

#### 文档更新流程
1. **内容审核**：确保准确性
2. **版本控制**：使用Git管理
3. **自动重建**：文档更新后自动重建向量索引
4. **A/B测试**：新版本先小流量测试

### 2. 对话质量优化

#### Prompt 工程

**意图识别 Prompt：**
```python
INTENT_PROMPT = """
分析用户消息，识别意图。

用户消息：{user_message}
历史对话：{history}

可能的意图：
1. query_order - 查询订单
2. product_inquiry - 产品咨询
3. complaint - 投诉
4. return_refund - 退换货

只返回意图类型。
"""
```

**回答生成 Prompt：**
```python
ANSWER_PROMPT = """
你是一个专业的客服助手。

用户问题：{question}
检索到的信息：{context}
工具调用结果：{tool_result}

要求：
1. 语气友好专业
2. 回答准确简洁
3. 如果信息不足，询问用户
4. 复杂问题建议转人工

生成回答：
"""
```

### 3. 性能优化

#### 缓存策略
```python
import redis

class CacheManager:
    def __init__(self):
        self.redis = redis.Redis(host='localhost')
    
    def get_answer(self, question):
        # 检查缓存
        cached = self.redis.get(f"qa:{question}")
        if cached:
            return cached
        
        # 生成新答案
        answer = self.agent.process(question)
        
        # 存入缓存（1小时）
        self.redis.setex(f"qa:{question}", 3600, answer)
        
        return answer
```

#### 异步处理
```python
import asyncio

async def process_message_async(message):
    # 并行执行多个操作
    intent_task = asyncio.create_task(classify_intent(message))
    context_task = asyncio.create_task(search_knowledge(message))
    
    intent, context = await asyncio.gather(intent_task, context_task)
    
    return generate_answer(intent, context)
```

### 4. 监控和日志

#### 关键指标

| 指标         | 说明           | 目标值    |
| ------------ | -------------- | --------- |
| **响应时间** | 平均处理时长   | < 2秒     |
| **解决率**   | 无需转人工比例 | > 70%     |
| **满意度**   | 用户评分       | > 4.0/5.0 |
| **错误率**   | 回答错误比例   | < 5%      |

#### 日志记录
```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('logs/customer_service.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)

def process_message(message):
    logger.info(f"收到消息: {message}")
    
    try:
        result = agent.process(message)
        logger.info(f"回答: {result}")
        return result
    except Exception as e:
        logger.error(f"处理失败: {e}")
        return "抱歉，系统出错了"
```

### 5. 安全考虑

#### API Key 保护
```python
# ❌ 错误：硬编码
api_key = "sk-1234567890"

# ✅ 正确：环境变量
import os
api_key = os.getenv("OPENAI_API_KEY")
```

#### 输入验证
```python
def validate_input(message):
    # 1. 长度限制
    if len(message) > 1000:
        return False, "消息过长"
    
    # 2. 敏感词过滤
    sensitive_words = ["敏感词1", "敏感词2"]
    if any(word in message for word in sensitive_words):
        return False, "包含敏感内容"
    
    # 3. SQL注入防护
    if "'" in message or "--" in message:
        return False, "非法字符"
    
    return True, ""
```

#### 频率限制
```python
from functools import wraps
import time

def rate_limit(max_calls, time_window):
    calls = {}
    
    def decorator(func):
        @wraps(func)
        def wrapper(user_id, *args, **kwargs):
            now = time.time()
            
            if user_id not in calls:
                calls[user_id] = []
            
            # 清理过期记录
            calls[user_id] = [t for t in calls[user_id] 
                              if now - t < time_window]
            
            if len(calls[user_id]) >= max_calls:
                raise Exception("请求过于频繁")
            
            calls[user_id].append(now)
            return func(user_id, *args, **kwargs)
        
        return wrapper
    return decorator

@rate_limit(max_calls=10, time_window=60)  # 每分钟最多10次
def process_user_message(user_id, message):
    return agent.process(message)
```

---

## 成本分析

### API 调用成本

#### GPT-4（OpenAI）
| 项目             | 价格            |
| ---------------- | --------------- |
| 输入             | $0.03/1K tokens |
| 输出             | $0.06/1K tokens |
| **每次对话估算** | **$0.01-0.05**  |

#### GLM-4（智谱AI）
| 项目             | 价格           |
| ---------------- | -------------- |
| 输入             | ¥0.1/1K tokens |
| 输出             | ¥0.1/1K tokens |
| **每次对话估算** | **¥0.02-0.1**  |

#### Ollama（本地免费）
- **成本**：0元（仅硬件成本）
- **配置要求**：16GB+ 内存
- **模型**：Llama2, Mistral等

### 月成本估算

假设日均1000次对话：

| 方案             | 月成本     | 说明               |
| ---------------- | ---------- | ------------------ |
| **OpenAI GPT-4** | ¥1000-3000 | 高质量，国际化     |
| **智谱AI GLM-4** | ¥600-3000  | 中文优化，国内稳定 |
| **Ollama本地**   | ¥0         | 需要服务器         |

---

## 常见问题

### Q1: 如何提高Agent的准确性？

**答：**
1. **优化 Prompt**：清晰的指令和示例
2. **增强知识库**：高质量、结构化的文档
3. **多轮验证**：Agent自我检查答案
4. **用户反馈**：收集并训练

### Q2: 如何处理复杂的多轮对话？

**答：**
```python
class ConversationManager:
    def __init__(self):
        self.sessions = {}  # {session_id: history}
    
    def add_message(self, session_id, role, content):
        if session_id not in self.sessions:
            self.sessions[session_id] = []
        
        self.sessions[session_id].append({
            "role": role,
            "content": content
        })
        
        # 保留最近10轮对话
        self.sessions[session_id] = self.sessions[session_id][-20:]
```

### Q3: 什么时候该转人工客服？

**答：触发条件**
1. 用户明确要求人工
2. Agent连续3次无法回答
3. 涉及敏感操作（退款、投诉）
4. 情感识别：用户愤怒、不满

```python
def should_transfer_human(state):
    # 1. 用户明确要求
    if "人工" in state["user_message"]:
        return True
    
    # 2. 连续失败
    if state["failed_attempts"] >= 3:
        return True
    
    # 3. 敏感操作
    if state["intent"] in ["complaint", "refund"]:
        return True
    
    return False
```

### Q4: 如何评估系统效果？

**答：关键指标**

```python
class Evaluator:
    def calculate_metrics(self, conversations):
        metrics = {
            "resolution_rate": 0,  # 解决率
            "avg_response_time": 0,  # 响应时间
            "user_satisfaction": 0,  # 满意度
            "human_transfer_rate": 0  # 转人工率
        }
        
        for conv in conversations:
            # 计算各项指标
            pass
        
        return metrics
```

---

## 后续优化方向

### 短期（1-2周）
- [ ] 接入真实LLM API
- [ ] 构建向量知识库
- [ ] 添加更多工具
- [ ] 优化Prompt

### 中期（1-2月）
- [ ] 使用LangGraph重构
- [ ] 添加A/B测试
- [ ] 实现用户反馈系统
- [ ] 性能优化（缓存、异步）

### 长期（3-6月）
- [ ] 微调专属模型
- [ ] 多语言支持
- [ ] 语音接入
- [ ] 情感分析

---

## 参考资源

### 文档
- [LangChain 官方文档](https://python.langchain.com/)
- [LangGraph 文档](https://langchain-ai.github.io/langgraph/)
- [OpenAI API 文档](https://platform.openai.com/docs)

### 开源项目
- [RAGFlow](https://github.com/infiniflow/ragflow)
- [Dify](https://github.com/langgenius/dify)
- [FastGPT](https://github.com/labring/FastGPT)

### 学习资源
- 📝 本项目配套 Jupyter Notebook：`Agentic-RAG智能客服完整实战.ipynb`
- 📖 前置知识：`Transformer-LangChain-LangGraph-RAGFlow完整对比.ipynb`

---

## 总结

### 🎯 核心要点

1. **Agentic RAG = RAG + Agent能力**
   - RAG：知识检索
   - Agent：智能决策、工具调用

2. **循序渐进**
   - 先简单实现理解原理
   - 再使用框架提升能力
   - 最后优化性能和体验

3. **实战为王**
   - 理论再多不如动手做
   - 从小项目开始
   - 不断迭代改进

### 🚀 行动建议

**本周：**
1. 运行简单版代码，理解核心流程
2. 设计自己的业务场景
3. 准备知识库文档

**下周：**
1. 接入真实LLM API
2. 构建向量知识库
3. 添加2-3个业务工具

**持续：**
1. 收集用户反馈
2. 优化Prompt和流程
3. 监控指标并改进

---

## 联系与反馈

如果有任何问题或建议，欢迎：
- 📧 提Issue讨论
- 💬 加入技术交流群
- 🌟 Star支持项目

**祝你构建出色的智能客服系统！🎉**

