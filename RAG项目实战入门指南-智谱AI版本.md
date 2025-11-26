# 🚀 RAG项目实战入门指南（智谱AI版本）

## 🎯 学习目标

通过本指南，您将：
1. ✅ 了解RAG项目的前置知识
2. ✅ 搭建完整的开发环境（使用智谱AI）
3. ✅ 实现第一个能运行的RAG系统
4. ✅ 理解RAG的核心组件
5. ✅ 掌握逐步扩展的方法

---

## 📋 前置知识自查表

### ✅ 必须掌握（您已经具备）

- [x] **Python基础**：变量、函数、类
- [x] **基本数据结构**：列表、字典
- [x] **文件操作**：读写文件

### 🟡 需要了解（可以边做边学）

- [ ] **向量数据库概念**：已学习理论，实践中会用到
- [ ] **API调用**：会用requests或SDK调用接口
- [ ] **异常处理**：try-except基本用法

### 🔵 可选（有更好，没有也行）

- [ ] **Docker**：方便部署向量数据库
- [ ] **异步编程**：async/await（可以提升性能）
- [ ] **Web框架**：Flask/FastAPI（如果要做API服务）

---

## 🎓 学习路径：三个阶段

### 阶段1：最简RAG（1-2天）⭐ 从这里开始

**目标：**跑通一个最简单的RAG系统

**核心功能：**
- 读取本地文档（txt/md）
- 使用智谱AI Embedding（embedding-2模型）
- 存储到本地（不用向量库）
- 简单检索+问答

**技术栈：**
```python
- Python 3.8+
- zhipuai库（智谱AI SDK）
- numpy（向量计算）
```

**难度：** ⭐ 适合入门

---

### 阶段2：标准RAG（3-5天）

**新增功能：**
- 接入向量数据库（Milvus/ChromaDB）
- 支持更多文档类型（PDF、Word）
- 优化检索效果

**技术栈：**
```python
- 阶段1 +
- pymilvus / chromadb（向量库）
- langchain（RAG框架）
- pypdf / docx（文档解析）
```

**难度：** ⭐⭐ 

---

### 阶段3：生产级RAG（1-2周）

**新增功能：**
- API服务化（FastAPI）
- 流式输出
- 多轮对话
- 监控和日志

**技术栈：**
```python
- 阶段2 +
- fastapi（Web框架）
- redis（缓存）
- docker（部署）
```

**难度：** ⭐⭐⭐

---

## 💡 建议：从阶段1开始！

先把最简单的版本跑通，理解核心原理，再逐步扩展。

---

## 🛠️ 阶段1：最简RAG实战

### 项目结构

```
simple-rag/
├── documents/          # 存放文档
│   ├── doc1.txt
│   └── doc2.txt
├── simple_rag.py      # 主程序
├── requirements.txt   # 依赖
├── .env               # API密钥配置
└── README.md
```

### 核心流程

```
1. 文档加载 → 分块
2. 生成Embedding → 存储
3. 用户提问 → 生成问题Embedding
4. 相似度搜索 → 找到相关文档
5. 文档+问题 → 发给LLM → 生成答案
```

让我们一步步实现！

---

## 📦 步骤1：环境配置

### 1.1 创建项目目录

```bash
mkdir simple-rag
cd simple-rag
mkdir documents
```

### 1.2 创建虚拟环境（推荐）

```bash
# Windows
python -m venv venv
venv\Scripts\activate

# Mac/Linux
python3 -m venv venv
source venv/bin/activate
```

### 1.3 安装依赖

创建 `requirements.txt`：

```txt
zhipuai>=2.0.0
numpy>=1.24.0
python-dotenv>=1.0.0
```

安装：

```bash
pip install -r requirements.txt
```

### 1.4 配置API密钥

**如何获取智谱AI API密钥：**
1. 访问 https://open.bigmodel.cn/
2. 注册/登录账号
3. 在控制台创建API密钥
4. 复制密钥

创建 `.env` 文件：

```bash
ZHIPUAI_API_KEY=your_api_key_here
```

**智谱AI的优势：**
- ✅ 国内访问速度快
- ✅ 价格相对便宜
- ✅ 支持中文效果好
- ✅ 有免费额度可用

---

## 💻 步骤2：完整代码实现

下面是一个完整的、可运行的最简RAG系统（智谱AI版本）：

### 2.1 创建 `simple_rag.py`

```python
"""
最简RAG系统 - 完整实现（智谱AI版本）
simple_rag.py
"""

import os
import numpy as np
from zhipuai import ZhipuAI
from typing import List, Tuple
import json

# ============================================
# 第1部分：文档处理
# ============================================

class DocumentLoader:
    """文档加载器"""
    
    def load_documents(self, folder_path: str) -> List[str]:
        """从文件夹加载所有txt文档"""
        documents = []
        
        for filename in os.listdir(folder_path):
            if filename.endswith('.txt'):
                filepath = os.path.join(folder_path, filename)
                with open(filepath, 'r', encoding='utf-8') as f:
                    content = f.read()
                    documents.append(content)
                print(f"✅ 加载文档: {filename}")
        
        return documents
    
    def split_text(self, text: str, chunk_size: int = 500) -> List[str]:
        """将文本分块
        
        Args:
            text: 原始文本
            chunk_size: 每块的字符数
        """
        chunks = []
        
        # 简单按字符数分块
        for i in range(0, len(text), chunk_size):
            chunk = text[i:i + chunk_size]
            chunks.append(chunk)
        
        return chunks


# ============================================
# 第2部分：向量存储（简单版，不用向量库）
# ============================================

class SimpleVectorStore:
    """简单的向量存储（内存存储）"""
    
    def __init__(self):
        self.texts = []      # 存储原始文本
        self.embeddings = [] # 存储对应的向量
        
    def add(self, text: str, embedding: List[float]):
        """添加文本和向量"""
        self.texts.append(text)
        self.embeddings.append(embedding)
    
    def search(self, query_embedding: List[float], top_k: int = 3) -> List[Tuple[str, float]]:
        """搜索最相似的文档
        
        Args:
            query_embedding: 查询向量
            top_k: 返回前k个结果
            
        Returns:
            [(文本, 相似度分数), ...]
        """
        if not self.embeddings:
            return []
        
        # 计算余弦相似度
        query_vec = np.array(query_embedding)
        
        similarities = []
        for i, emb in enumerate(self.embeddings):
            emb_vec = np.array(emb)
            
            # 余弦相似度公式
            similarity = np.dot(query_vec, emb_vec) / (
                np.linalg.norm(query_vec) * np.linalg.norm(emb_vec)
            )
            
            similarities.append((self.texts[i], float(similarity)))
        
        # 按相似度排序，返回top_k
        similarities.sort(key=lambda x: x[1], reverse=True)
        return similarities[:top_k]
    
    def save(self, filepath: str):
        """保存到文件"""
        data = {
            'texts': self.texts,
            'embeddings': self.embeddings
        }
        with open(filepath, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False)
        print(f"✅ 向量库已保存到: {filepath}")
    
    def load(self, filepath: str):
        """从文件加载"""
        with open(filepath, 'r', encoding='utf-8') as f:
            data = json.load(f)
        self.texts = data['texts']
        self.embeddings = data['embeddings']
        print(f"✅ 向量库已加载，共 {len(self.texts)} 条记录")


# ============================================
# 第3部分：RAG核心类
# ============================================

class SimpleRAG:
    """简单的RAG系统（智谱AI版本）"""
    
    def __init__(self, api_key: str):
        """初始化
        
        Args:
            api_key: 智谱AI API密钥
        """
        self.client = ZhipuAI(api_key=api_key)
        
        self.doc_loader = DocumentLoader()
        self.vector_store = SimpleVectorStore()
        
        print("✅ SimpleRAG 初始化完成（使用智谱AI）")
    
    def get_embedding(self, text: str) -> List[float]:
        """获取文本的Embedding向量"""
        try:
            response = self.client.embeddings.create(
                model="embedding-2",  # 智谱AI的embedding模型
                input=text
            )
            # 智谱AI返回格式：response.data[0].embedding
            return response.data[0].embedding
        except Exception as e:
            print(f"❌ 获取Embedding失败: {e}")
            raise
    
    def index_documents(self, folder_path: str):
        """索引文档（构建向量库）"""
        print(f"\n📚 开始索引文档...")
        
        # 1. 加载文档
        documents = self.doc_loader.load_documents(folder_path)
        print(f"✅ 共加载 {len(documents)} 个文档")
        
        # 2. 分块
        all_chunks = []
        for doc in documents:
            chunks = self.doc_loader.split_text(doc, chunk_size=500)
            all_chunks.extend(chunks)
        print(f"✅ 共分成 {len(all_chunks)} 个文本块")
        
        # 3. 生成Embedding并存储
        print("🔄 正在生成向量...")
        for i, chunk in enumerate(all_chunks):
            embedding = self.get_embedding(chunk)
            self.vector_store.add(chunk, embedding)
            
            if (i + 1) % 10 == 0:
                print(f"   进度: {i + 1}/{len(all_chunks)}")
        
        print("✅ 向量生成完成！")
    
    def query(self, question: str, top_k: int = 3) -> str:
        """查询问题
        
        Args:
            question: 用户问题
            top_k: 检索多少个相关文档
            
        Returns:
            LLM生成的答案
        """
        print(f"\n❓ 问题: {question}")
        
        # 1. 将问题转成向量
        print("🔄 正在检索相关文档...")
        question_embedding = self.get_embedding(question)
        
        # 2. 检索相似文档
        results = self.vector_store.search(question_embedding, top_k=top_k)
        
        if not results:
            return "❌ 没有找到相关文档"
        
        # 3. 打印检索结果
        print(f"✅ 找到 {len(results)} 个相关文档:")
        for i, (text, score) in enumerate(results):
            print(f"   {i+1}. 相似度: {score:.3f} - {text[:50]}...")
        
        # 4. 构建prompt
        context = "\n\n".join([text for text, score in results])
        
        prompt = f"""请基于以下文档内容回答问题。如果文档中没有相关信息，请说"文档中没有找到相关信息"。

文档内容：
{context}

问题：{question}

回答："""
        
        # 5. 调用LLM生成答案
        print("🔄 正在生成答案...")
        try:
            response = self.client.chat.completions.create(
                model="glm-4",  # 智谱AI的对话模型，也可以用 "glm-3-turbo"
                messages=[
                    {"role": "user", "content": prompt}
                ],
                temperature=0.7
            )
            
            answer = response.choices[0].message.content
            return answer
        except Exception as e:
            print(f"❌ 生成答案失败: {e}")
            return f"生成答案时出错: {str(e)}"
```

---

## 🎮 步骤3：使用示例

### 3.1 准备测试文档

创建 `documents/ai_intro.txt`：

```
人工智能（Artificial Intelligence, AI）是计算机科学的一个分支。
它企图了解智能的实质，并生产出一种新的能以人类智能相似的方式做出反应的智能机器。
该领域的研究包括机器人、语言识别、图像识别、自然语言处理和专家系统等。

机器学习是人工智能的核心，是使计算机具有智能的根本途径。
深度学习是机器学习的一个分支，它使用神经网络来学习数据的表示。
```

创建 `documents/rag_intro.txt`：

```
RAG（Retrieval Augmented Generation）是一种结合检索和生成的AI技术。
它通过检索相关文档来增强大语言模型的生成能力。

RAG的核心优势包括：
1. 可以使用最新的、私有的数据
2. 减少大模型的幻觉问题
3. 答案可以追溯来源
4. 不需要重新训练模型

RAG系统通常包含三个核心组件：文档检索器、向量数据库和大语言模型。
```

### 3.2 创建主程序 `main.py`

```python
from simple_rag import SimpleRAG
import os
from dotenv import load_dotenv

# 加载环境变量
load_dotenv()

# 从环境变量读取API密钥
api_key = os.getenv('ZHIPUAI_API_KEY')

if not api_key:
    print("❌ 请在 .env 文件中设置 ZHIPUAI_API_KEY")
    exit(1)

# 初始化RAG
rag = SimpleRAG(api_key=api_key)

# 索引文档
rag.index_documents('documents')

# 保存向量库（可选）
rag.vector_store.save('vector_store.json')

# 开始问答
while True:
    question = input("\n请输入问题（输入'quit'退出）: ")
    
    if question.lower() == 'quit':
        break
    
    answer = rag.query(question)
    print(f"\n💡 答案:\n{answer}\n")
    print("-" * 50)
```

### 3.3 运行

```bash
python main.py
```

---

## 🔍 代码详解

### 核心流程图

```
用户问题: "什么是RAG?"
    ↓
1. 转成向量 (get_embedding)
    使用智谱AI embedding-2模型
    [0.23, -0.45, 0.67, ...]
    ↓
2. 向量检索 (vector_store.search)
    找到最相似的3个文档块
    ↓
3. 构建Prompt
    "基于以下文档: [文档1][文档2][文档3]
     问题: 什么是RAG?"
    ↓
4. 发给LLM (client.chat.completions.create)
    使用智谱AI glm-4模型
    ↓
5. 返回答案
    "RAG是一种结合检索和生成的AI技术..."
```

### 关键代码解析

#### 1. 智谱AI客户端初始化

```python
from zhipuai import ZhipuAI

self.client = ZhipuAI(api_key=api_key)
```

**说明：**
- 使用 `zhipuai` 库，而不是 `openai`
- 只需要传入 `api_key`，不需要 `base_url`

#### 2. 获取Embedding向量

```python
response = self.client.embeddings.create(
    model="embedding-2",  # 智谱AI的embedding模型
    input=text
)
return response.data[0].embedding
```

**说明：**
- 模型名称：`embedding-2`（智谱AI的embedding模型）
- 返回格式与OpenAI相同：`response.data[0].embedding`

#### 3. 调用对话模型

```python
response = self.client.chat.completions.create(
    model="glm-4",  # 智谱AI的对话模型
    messages=[
        {"role": "user", "content": prompt}
    ],
    temperature=0.7
)
```

**说明：**
- 模型名称：`glm-4`（智谱AI最新模型）或 `glm-3-turbo`（更快更便宜）
- API调用方式与OpenAI完全兼容

#### 4. 余弦相似度计算

```python
similarity = np.dot(query_vec, emb_vec) / (
    np.linalg.norm(query_vec) * np.linalg.norm(emb_vec)
)
```

**含义：**
- `np.dot()`: 向量点积
- `np.linalg.norm()`: 向量长度
- 余弦相似度 = 点积 / (长度1 × 长度2)
- 范围：-1到1，越接近1越相似

---

## 🚨 常见问题和解决方案

### Q1: 智谱AI API调用失败怎么办？

**方案A：检查API密钥**

```python
# 确保API密钥正确
import os
from dotenv import load_dotenv
load_dotenv()

api_key = os.getenv('ZHIPUAI_API_KEY')
print(f"API密钥前5位: {api_key[:5]}...")  # 检查是否正确加载
```

**方案B：检查网络连接**

```python
# 智谱AI服务器在国内，如果访问失败可能是网络问题
# 可以尝试使用代理或检查防火墙设置
```

**方案C：查看错误信息**

```python
try:
    response = self.client.embeddings.create(...)
except Exception as e:
    print(f"错误详情: {e}")
    # 常见错误：
    # - API密钥无效
    # - 余额不足
    # - 请求频率过高
```

---

### Q2: API调用太慢怎么办？

**解决方案：批量处理**

```python
def get_embeddings_batch(self, texts: List[str]) -> List[List[float]]:
    """批量获取embedding（智谱AI版本）"""
    try:
        response = self.client.embeddings.create(
            model="embedding-2",
            input=texts  # 一次发送多个文本（智谱AI支持批量）
        )
        return [data.embedding for data in response.data]
    except Exception as e:
        print(f"❌ 批量获取Embedding失败: {e}")
        # 降级为单个处理
        return [self.get_embedding(text) for text in texts]
```

**优化索引文档方法：**

```python
def index_documents(self, folder_path: str):
    # ... 前面的代码 ...
    
    # 批量生成Embedding（更快）
    print("🔄 正在批量生成向量...")
    batch_size = 20  # 每批处理20个
    for i in range(0, len(all_chunks), batch_size):
        batch = all_chunks[i:i + batch_size]
        embeddings = self.get_embeddings_batch(batch)
        
        for chunk, embedding in zip(batch, embeddings):
            self.vector_store.add(chunk, embedding)
        
        print(f"   进度: {min(i + batch_size, len(all_chunks))}/{len(all_chunks)}")
```

---

### Q3: 文档太大怎么办？

**解决方案：智能分块**

```python
def split_text_smart(self, text: str, chunk_size: int = 500):
    """按段落智能分块"""
    paragraphs = text.split('\n\n')
    
    chunks = []
    current_chunk = ""
    
    for para in paragraphs:
        if len(current_chunk) + len(para) < chunk_size:
            current_chunk += para + "\n\n"
        else:
            if current_chunk:
                chunks.append(current_chunk.strip())
            current_chunk = para + "\n\n"
    
    if current_chunk:
        chunks.append(current_chunk.strip())
    
    return chunks
```

---

### Q4: 检索结果不准确？

**解决方案：调整参数**

```python
# 1. 增加检索数量
results = self.vector_store.search(question_embedding, top_k=5)  # 从3改到5

# 2. 添加相似度阈值
filtered_results = [
    (text, score) for text, score in results 
    if score > 0.7  # 只要相似度>0.7的
]

# 3. 调整chunk_size
# 如果chunk太小，可能丢失上下文
# 如果chunk太大，可能包含无关信息
chunks = self.doc_loader.split_text(doc, chunk_size=800)  # 从500改到800
```

---

### Q5: 智谱AI模型选择

**Embedding模型：**
- `embedding-2`：推荐使用，效果好
- `embedding-1`：旧版本，不推荐

**对话模型：**
- `glm-4`：最新最强模型，效果好但稍慢
- `glm-3-turbo`：速度快，价格便宜，适合简单任务
- `glm-4-flash`：平衡版本，速度快且效果好

**选择建议：**
- 开发测试：用 `glm-3-turbo`（便宜）
- 生产环境：用 `glm-4`（效果好）

---

## 🎯 练习任务

### 任务1：基础版（必做）⭐

**目标：**跑通上面的代码

**步骤：**
1. ✅ 安装依赖
2. ✅ 注册智谱AI账号并获取API密钥
3. ✅ 创建测试文档（至少2个txt文件）
4. ✅ 运行代码，成功问答
5. ✅ 测试不同的问题

**验收标准：**
- 能成功索引文档
- 能正确回答基于文档的问题
- 理解整个流程

---

### 任务2：改进版（推荐）⭐⭐

**目标：**添加新功能

**可选改进：**
1. ✅ 添加文档元数据（文件名、创建时间）
2. ✅ 在答案中显示引用来源
3. ✅ 支持多种文档格式（md、pdf）
4. ✅ 添加简单的Web界面（Streamlit）

**提示：引用来源实现**

```python
def query_with_source(self, question: str) -> Tuple[str, List[str]]:
    """查询并返回来源"""
    # ... 检索代码 ...
    
    # 记录来源
    sources = [f"文档片段{i+1}: {text[:100]}..." 
               for i, (text, score) in enumerate(results)]
    
    # 在prompt中要求标注来源
    prompt = f"""基于以下文档回答，并标注引用了哪些文档。

文档1: {results[0][0]}
文档2: {results[1][0]}
...

问题: {question}"""
    
    answer = # ... LLM调用 ...
    
    return answer, sources
```

---

### 任务3：向量库版（进阶）⭐⭐⭐

**目标：**接入真实向量数据库

**步骤：**
1. ✅ 安装ChromaDB（最简单的向量库）
   ```bash
   pip install chromadb
   ```

2. ✅ 替换SimpleVectorStore为ChromaDB

```python
import chromadb

class ChromaVectorStore:
    def __init__(self):
        self.client = chromadb.Client()
        self.collection = self.client.create_collection("documents")
    
    def add(self, text: str, embedding: List[float]):
        self.collection.add(
            embeddings=[embedding],
            documents=[text],
            ids=[f"doc_{len(self.collection.get()['ids'])}"]
        )
    
    def search(self, query_embedding: List[float], top_k: int = 3):
        results = self.collection.query(
            query_embeddings=[query_embedding],
            n_results=top_k
        )
        return list(zip(results['documents'][0], results['distances'][0]))
```

---

## 📚 推荐学习资源

### 官方文档

1. **智谱AI API文档**
   - https://open.bigmodel.cn/dev/api
   - 学习Embedding和Chat API的用法
   - 查看模型列表和价格
   - 查看示例代码

2. **ChromaDB文档**
   - https://docs.trychroma.com/
   - 最简单的向量数据库

3. **LangChain文档**
   - https://python.langchain.com/
   - RAG框架，阶段2会用到
   - 支持智谱AI集成

### 视频教程

1. **B站搜索：**
   - "RAG教程"
   - "大模型应用开发"
   - "智谱AI使用教程"

2. **YouTube搜索：**
   - "RAG tutorial"
   - "Build RAG from scratch"
   - "LangChain RAG"

### 开源项目参考

1. **Awesome-RAG**
   - https://github.com/Awesome-LLM-RAG/Awesome-RAG
   - RAG相关资源合集

2. **LangChain RAG示例**
   ```bash
   git clone https://github.com/langchain-ai/langchain
   cd langchain/templates/rag-*
   ```

---

## 🗺️ 后续学习路线

### 完成阶段1后，接下来学什么？

```
阶段1（当前）
  ↓
了解核心概念 ✅
  ↓
阶段2：标准RAG
  - 学习LangChain框架
  - 接入Milvus/ChromaDB
  - 文档解析（PDF、Word）
  - 优化检索效果
  ↓
阶段3：高级特性
  - 多轮对话
  - 流式输出
  - Agent能力
  ↓
阶段4：生产部署
  - API服务化
  - 性能优化
  - 监控和日志
```

### 推荐的学习顺序

1. **第1周：阶段1**
   - 跑通本文档的代码
   - 完成任务1和任务2
   - 理解RAG的核心原理

2. **第2周：阶段2**
   - 学习LangChain基础
   - 接入向量数据库
   - 处理更复杂的文档

3. **第3-4周：实战项目**
   - 选择一个真实场景（如个人知识库）
   - 完整实现一个RAG应用
   - 部署并实际使用

---

## 💡 关键要点总结

### ✅ RAG的核心只有5步

1. **文档加载** → 读取文档
2. **文档分块** → 切成小段
3. **生成向量** → Embedding（使用智谱AI embedding-2）
4. **向量检索** → 找相似文档
5. **LLM生成** → 基于文档回答（使用智谱AI glm-4）

### ✅ 最小可运行系统需要

- ✅ Python基础
- ✅ 智谱AI API密钥（注册即送免费额度）
- ✅ 200行代码
- ✅ 半天时间

### ✅ 不需要（初期）

- ❌ 复杂的向量数据库
- ❌ LangChain框架
- ❌ 生产级部署
- ❌ 分布式架构

### ✅ 学习建议

1. **先跑通，再优化**
   - 不要一开始就追求完美
   - 先实现最简版本
   - 理解原理后再改进

2. **动手最重要**
   - 看100遍不如写1遍
   - 遇到问题现查文档
   - 边做边学效率最高

3. **从简单文档开始**
   - 先用txt文件测试
   - 不要一开始就处理PDF、Word
   - 文档内容要少（几千字即可）

---

## 🎯 下一步行动清单

### 今天就可以做的（1-2小时）

- [ ] 创建项目目录
- [ ] 安装依赖（pip install zhipuai numpy python-dotenv）
- [ ] 注册智谱AI账号并获取API密钥
- [ ] 创建2-3个测试文档
- [ ] 复制本文档的代码到 `simple_rag.py`

### 本周完成（2-3天）

- [ ] 跑通完整流程
- [ ] 测试10个不同问题
- [ ] 理解每行代码的含义
- [ ] 完成任务1（基础版）
- [ ] 尝试任务2（改进版）

### 下周目标（1周）

- [ ] 学习LangChain基础
- [ ] 接入ChromaDB
- [ ] 支持PDF文档
- [ ] 做一个简单的Web界面

---

## ✅ 准备好开始了吗？

**现在就动手吧！** 🚀

记住：
- 遇到问题很正常，多查文档
- 从最简单的开始
- 每天进步一点点

**祝您学习顺利！** 💪

---

## 📝 完整代码文件清单

### `simple_rag.py` - 完整代码

```python
"""
最简RAG系统 - 完整实现（智谱AI版本）
simple_rag.py
"""

import os
import numpy as np
from zhipuai import ZhipuAI
from typing import List, Tuple
import json


class DocumentLoader:
    """文档加载器"""
    
    def load_documents(self, folder_path: str) -> List[str]:
        """从文件夹加载所有txt文档"""
        documents = []
        
        for filename in os.listdir(folder_path):
            if filename.endswith('.txt'):
                filepath = os.path.join(folder_path, filename)
                with open(filepath, 'r', encoding='utf-8') as f:
                    content = f.read()
                    documents.append(content)
                print(f"✅ 加载文档: {filename}")
        
        return documents
    
    def split_text(self, text: str, chunk_size: int = 500) -> List[str]:
        """将文本分块"""
        chunks = []
        for i in range(0, len(text), chunk_size):
            chunk = text[i:i + chunk_size]
            chunks.append(chunk)
        return chunks


class SimpleVectorStore:
    """简单的向量存储（内存存储）"""
    
    def __init__(self):
        self.texts = []
        self.embeddings = []
        
    def add(self, text: str, embedding: List[float]):
        """添加文本和向量"""
        self.texts.append(text)
        self.embeddings.append(embedding)
    
    def search(self, query_embedding: List[float], top_k: int = 3) -> List[Tuple[str, float]]:
        """搜索最相似的文档"""
        if not self.embeddings:
            return []
        
        query_vec = np.array(query_embedding)
        similarities = []
        
        for i, emb in enumerate(self.embeddings):
            emb_vec = np.array(emb)
            similarity = np.dot(query_vec, emb_vec) / (
                np.linalg.norm(query_vec) * np.linalg.norm(emb_vec)
            )
            similarities.append((self.texts[i], float(similarity)))
        
        similarities.sort(key=lambda x: x[1], reverse=True)
        return similarities[:top_k]
    
    def save(self, filepath: str):
        """保存到文件"""
        data = {'texts': self.texts, 'embeddings': self.embeddings}
        with open(filepath, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False)
        print(f"✅ 向量库已保存到: {filepath}")
    
    def load(self, filepath: str):
        """从文件加载"""
        with open(filepath, 'r', encoding='utf-8') as f:
            data = json.load(f)
        self.texts = data['texts']
        self.embeddings = data['embeddings']
        print(f"✅ 向量库已加载，共 {len(self.texts)} 条记录")


class SimpleRAG:
    """简单的RAG系统（智谱AI版本）"""
    
    def __init__(self, api_key: str):
        """初始化"""
        self.client = ZhipuAI(api_key=api_key)
        self.doc_loader = DocumentLoader()
        self.vector_store = SimpleVectorStore()
        print("✅ SimpleRAG 初始化完成（使用智谱AI）")
    
    def get_embedding(self, text: str) -> List[float]:
        """获取文本的Embedding向量"""
        try:
            response = self.client.embeddings.create(
                model="embedding-2",
                input=text
            )
            return response.data[0].embedding
        except Exception as e:
            print(f"❌ 获取Embedding失败: {e}")
            raise
    
    def index_documents(self, folder_path: str):
        """索引文档（构建向量库）"""
        print(f"\n📚 开始索引文档...")
        
        documents = self.doc_loader.load_documents(folder_path)
        print(f"✅ 共加载 {len(documents)} 个文档")
        
        all_chunks = []
        for doc in documents:
            chunks = self.doc_loader.split_text(doc, chunk_size=500)
            all_chunks.extend(chunks)
        print(f"✅ 共分成 {len(all_chunks)} 个文本块")
        
        print("🔄 正在生成向量...")
        for i, chunk in enumerate(all_chunks):
            embedding = self.get_embedding(chunk)
            self.vector_store.add(chunk, embedding)
            if (i + 1) % 10 == 0:
                print(f"   进度: {i + 1}/{len(all_chunks)}")
        
        print("✅ 向量生成完成！")
    
    def query(self, question: str, top_k: int = 3) -> str:
        """查询问题"""
        print(f"\n❓ 问题: {question}")
        
        print("🔄 正在检索相关文档...")
        question_embedding = self.get_embedding(question)
        
        results = self.vector_store.search(question_embedding, top_k=top_k)
        
        if not results:
            return "❌ 没有找到相关文档"
        
        print(f"✅ 找到 {len(results)} 个相关文档:")
        for i, (text, score) in enumerate(results):
            print(f"   {i+1}. 相似度: {score:.3f} - {text[:50]}...")
        
        context = "\n\n".join([text for text, score in results])
        
        prompt = f"""请基于以下文档内容回答问题。如果文档中没有相关信息，请说"文档中没有找到相关信息"。

文档内容：
{context}

问题：{question}

回答："""
        
        print("🔄 正在生成答案...")
        try:
            response = self.client.chat.completions.create(
                model="glm-4",
                messages=[{"role": "user", "content": prompt}],
                temperature=0.7
            )
            answer = response.choices[0].message.content
            return answer
        except Exception as e:
            print(f"❌ 生成答案失败: {e}")
            return f"生成答案时出错: {str(e)}"
```

### `main.py` - 主程序

```python
from simple_rag import SimpleRAG
import os
from dotenv import load_dotenv

# 加载环境变量
load_dotenv()

# 从环境变量读取API密钥
api_key = os.getenv('ZHIPUAI_API_KEY')

if not api_key:
    print("❌ 请在 .env 文件中设置 ZHIPUAI_API_KEY")
    exit(1)

# 初始化RAG
rag = SimpleRAG(api_key=api_key)

# 索引文档
rag.index_documents('documents')

# 保存向量库（可选）
rag.vector_store.save('vector_store.json')

# 开始问答
while True:
    question = input("\n请输入问题（输入'quit'退出）: ")
    
    if question.lower() == 'quit':
        break
    
    answer = rag.query(question)
    print(f"\n💡 答案:\n{answer}\n")
    print("-" * 50)
```

### `requirements.txt` - 依赖

```txt
zhipuai>=2.0.0
numpy>=1.24.0
python-dotenv>=1.0.0
```

### `.env` - 配置文件

```bash
ZHIPUAI_API_KEY=your_api_key_here
```

---

**文档创建时间：** 2025-11-19  
**作者：** AI Assistant  
**版本：** 1.0（智谱AI专用版）

