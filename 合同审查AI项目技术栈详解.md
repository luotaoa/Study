# 合同审查AI项目技术栈详解

## 📋 目录

1. [项目功能概述](#项目功能概述)
2. [核心技术栈](#核心技术栈)
3. [文档解析技术](#文档解析技术)
4. [NLP与信息抽取](#nlp与信息抽取)
5. [大模型应用](#大模型应用)
6. [合同对比技术](#合同对比技术)
7. [完整架构设计](#完整架构设计)
8. [开源项目推荐](#开源项目推荐)

---

## 🎯 项目功能概述

### 合同审查系统通常包含

```
核心功能：
├─ 📄 文档上传（PDF、Word、图片）
├─ 🔍 关键信息提取
│   ├─ 合同主体（甲方、乙方）
│   ├─ 金额条款
│   ├─ 日期条款（签订日期、生效日期、到期日期）
│   ├─ 权利义务条款
│   └─ 违约责任条款
├─ ⚠️ 风险点识别
│   ├─ 法律风险（不符合法律法规）
│   ├─ 商业风险（条款对己方不利）
│   ├─ 遗漏风险（缺少必要条款）
│   └─ 歧义风险（表述不清）
├─ 📊 合同对比
│   ├─ 两版本对比（红线标注）
│   ├─ 与标准模板对比
│   └─ 差异点分析
├─ 💡 修改建议
│   └─ 给出具体的修改意见
└─ 📈 审查报告生成
```

---

## 🛠️ 核心技术栈

### 技术架构全景图

```
┌─────────────────────────────────────────┐
│          前端层                         │
│  React/Vue + Ant Design                 │
│  + PDF.js/Viewer（文档预览）            │
│  + Diff库（对比展示）                   │
└──────────────┬──────────────────────────┘
               ↓
┌─────────────────────────────────────────┐
│          API层                          │
│  FastAPI/Django/Node.js                 │
└──────────────┬──────────────────────────┘
               ↓
┌─────────────────────────────────────────┐
│       文档处理层                        │
│  ├─ PDF解析：PyPDF2/pdfplumber         │
│  ├─ Word解析：python-docx               │
│  ├─ OCR：PaddleOCR/Tesseract           │
│  └─ 表格提取：Camelot/Tabula           │
└──────────────┬──────────────────────────┘
               ↓
┌─────────────────────────────────────────┐
│       NLP处理层                         │
│  ├─ 命名实体识别（NER）                 │
│  ├─ 关系抽取（RE）                      │
│  ├─ 文本分类                            │
│  └─ 相似度计算                          │
└──────────────┬──────────────────────────┘
               ↓
┌─────────────────────────────────────────┐
│       大模型层                          │
│  ├─ GPT-4/Claude（理解+生成）          │
│  ├─ Qwen-72B（开源方案）               │
│  └─ 法律专用模型（LaWGPT/智海）        │
└──────────────┬──────────────────────────┘
               ↓
┌─────────────────────────────────────────┐
│       知识库层（RAG）                   │
│  ├─ 向量数据库：Milvus/Chroma          │
│  ├─ 法律法规库                          │
│  ├─ 判例库                              │
│  └─ 企业合同模板库                      │
└──────────────┬──────────────────────────┘
               ↓
┌─────────────────────────────────────────┐
│       存储层                            │
│  ├─ 文档存储：MinIO/OSS                │
│  ├─ 数据库：PostgreSQL/MongoDB         │
│  └─ 缓存：Redis                         │
└─────────────────────────────────────────┘
```

---

## 📄 第一层：文档解析技术

### 1. PDF解析

**问题**：合同通常是扫描件或PDF格式

```python
# 方案1：纯数字PDF（可复制文字的PDF）
import pdfplumber

def extract_pdf_text(pdf_path):
    """提取PDF文字内容"""
    text = ""
    tables = []
    
    with pdfplumber.open(pdf_path) as pdf:
        for page in pdf.pages:
            # 提取文字
            text += page.extract_text()
            
            # 提取表格
            page_tables = page.extract_tables()
            if page_tables:
                tables.extend(page_tables)
    
    return text, tables

# 使用
text, tables = extract_pdf_text("合同.pdf")
print(f"提取文字：{len(text)}字")
print(f"提取表格：{len(tables)}个")
```

```python
# 方案2：扫描件PDF（需要OCR）
from pdf2image import convert_from_path
from paddleocr import PaddleOCR
import cv2

class ScanPDFParser:
    def __init__(self):
        # 初始化OCR引擎
        self.ocr = PaddleOCR(
            use_angle_cls=True,  # 文字方向检测
            lang='ch',           # 中文
            use_gpu=True         # 使用GPU加速
        )
    
    def parse_scan_pdf(self, pdf_path):
        """解析扫描版PDF"""
        # 1. PDF转图片
        images = convert_from_path(pdf_path, dpi=300)
        
        all_text = ""
        for i, image in enumerate(images):
            print(f"处理第{i+1}页...")
            
            # 2. 图片预处理（提高OCR准确率）
            processed = self.preprocess_image(image)
            
            # 3. OCR识别
            result = self.ocr.ocr(processed, cls=True)
            
            # 4. 提取文字
            page_text = self.extract_text_from_ocr(result)
            all_text += f"\n第{i+1}页\n{page_text}\n"
        
        return all_text
    
    def preprocess_image(self, image):
        """图片预处理"""
        import numpy as np
        
        # PIL转OpenCV
        img = cv2.cvtColor(np.array(image), cv2.COLOR_RGB2BGR)
        
        # 灰度化
        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        
        # 二值化（提高对比度）
        _, binary = cv2.threshold(
            gray, 0, 255, 
            cv2.THRESH_BINARY + cv2.THRESH_OTSU
        )
        
        # 去噪
        denoised = cv2.fastNlMeansDenoising(binary)
        
        return denoised
    
    def extract_text_from_ocr(self, ocr_result):
        """从OCR结果提取文字"""
        text = ""
        for line in ocr_result:
            for word_info in line:
                text += word_info[1][0] + " "  # word_info[1][0]是识别的文字
            text += "\n"
        return text

# 使用
parser = ScanPDFParser()
text = parser.parse_scan_pdf("扫描版合同.pdf")
```

### 2. Word文档解析

```python
from docx import Document

def extract_word_content(docx_path):
    """提取Word文档内容"""
    doc = Document(docx_path)
    
    # 提取段落
    paragraphs = [p.text for p in doc.paragraphs if p.text.strip()]
    
    # 提取表格
    tables = []
    for table in doc.tables:
        table_data = []
        for row in table.rows:
            row_data = [cell.text for cell in row.cells]
            table_data.append(row_data)
        tables.append(table_data)
    
    return {
        'text': '\n'.join(paragraphs),
        'tables': tables
    }

# 使用
content = extract_word_content("合同.docx")
```

### 3. 通用文档解析框架

**推荐：Unstructured** - 一站式文档解析

```python
from unstructured.partition.auto import partition

def parse_any_document(file_path):
    """解析任何格式的文档"""
    # 自动识别格式并解析
    elements = partition(filename=file_path)
    
    # 提取不同类型的内容
    result = {
        'title': [],
        'text': [],
        'tables': [],
        'lists': []
    }
    
    for element in elements:
        element_type = type(element).__name__
        
        if element_type == 'Title':
            result['title'].append(str(element))
        elif element_type == 'NarrativeText':
            result['text'].append(str(element))
        elif element_type == 'Table':
            result['tables'].append(str(element))
        elif element_type == 'ListItem':
            result['lists'].append(str(element))
    
    return result

# 支持的格式
# PDF、Word、Excel、PPT、HTML、图片等
content = parse_any_document("合同.pdf")
```

---

## 🧠 第二层：NLP与信息抽取

### 1. 命名实体识别（NER）- 提取关键信息

**目标**：从合同中提取甲方、乙方、金额、日期等

```python
from transformers import pipeline

class ContractNER:
    def __init__(self):
        # 使用法律领域预训练模型
        self.ner = pipeline(
            "ner", 
            model="law-ner-model",  # 法律领域NER模型
            aggregation_strategy="simple"
        )
    
    def extract_entities(self, text):
        """提取合同实体"""
        # NER识别
        entities = self.ner(text)
        
        # 分类整理
        result = {
            '甲方': [],
            '乙方': [],
            '金额': [],
            '日期': [],
            '地点': []
        }
        
        for entity in entities:
            entity_type = entity['entity_group']
            entity_text = entity['word']
            
            if entity_type == 'PARTY_A':
                result['甲方'].append(entity_text)
            elif entity_type == 'PARTY_B':
                result['乙方'].append(entity_text)
            elif entity_type == 'MONEY':
                result['金额'].append(entity_text)
            elif entity_type == 'DATE':
                result['日期'].append(entity_text)
            elif entity_type == 'LOCATION':
                result['地点'].append(entity_text)
        
        return result

# 使用
ner = ContractNER()
entities = ner.extract_entities(contract_text)
print(entities)
# 输出：
# {
#   '甲方': ['北京XX科技有限公司'],
#   '乙方': ['上海XX贸易有限公司'],
#   '金额': ['100万元', '10万元'],
#   '日期': ['2024年1月1日', '2025年12月31日'],
#   '地点': ['北京市朝阳区']
# }
```

### 2. 条款分类

**目标**：将合同分成不同的条款类型

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch

class ClauseClassifier:
    def __init__(self):
        self.tokenizer = AutoTokenizer.from_pretrained("legal-bert")
        self.model = AutoModelForSequenceClassification.from_pretrained(
            "legal-clause-classifier"
        )
        
        self.labels = [
            '合同主体',
            '合同标的',
            '价款支付',
            '履行期限',
            '违约责任',
            '争议解决',
            '其他'
        ]
    
    def classify_clause(self, clause_text):
        """分类单个条款"""
        inputs = self.tokenizer(
            clause_text, 
            return_tensors="pt",
            truncation=True,
            max_length=512
        )
        
        with torch.no_grad():
            outputs = self.model(**inputs)
        
        probs = torch.softmax(outputs.logits, dim=-1)
        label_id = torch.argmax(probs, dim=-1).item()
        confidence = probs[0][label_id].item()
        
        return {
            'label': self.labels[label_id],
            'confidence': confidence
        }
    
    def split_and_classify(self, contract_text):
        """拆分并分类所有条款"""
        # 1. 按条款编号拆分
        clauses = self.split_clauses(contract_text)
        
        # 2. 分类每个条款
        results = []
        for clause in clauses:
            classification = self.classify_clause(clause['text'])
            results.append({
                'clause_number': clause['number'],
                'text': clause['text'],
                'type': classification['label'],
                'confidence': classification['confidence']
            })
        
        return results
    
    def split_clauses(self, text):
        """按条款编号拆分"""
        import re
        
        # 匹配"第X条"、"第X章"等
        pattern = r'第[一二三四五六七八九十\d]+条'
        matches = list(re.finditer(pattern, text))
        
        clauses = []
        for i, match in enumerate(matches):
            start = match.start()
            end = matches[i+1].start() if i+1 < len(matches) else len(text)
            
            clauses.append({
                'number': match.group(),
                'text': text[start:end].strip()
            })
        
        return clauses

# 使用
classifier = ClauseClassifier()
classified_clauses = classifier.split_and_classify(contract_text)

for clause in classified_clauses:
    print(f"{clause['clause_number']}: {clause['type']} (置信度:{clause['confidence']:.2f})")
```

---

## 🤖 第三层：大模型应用

### 1. 使用GPT-4进行合同审查

```python
from openai import OpenAI

class ContractReviewer:
    def __init__(self):
        self.client = OpenAI(api_key="your_key")
    
    def review_contract(self, contract_text):
        """完整审查合同"""
        prompt = f"""
你是一位专业的法律顾问，请审查以下合同，并从以下几个维度给出意见：

合同内容：
{contract_text}

请按以下格式输出：

1. 【合同概要】
   - 合同类型：
   - 合同双方：
   - 主要内容：
   - 合同金额：
   - 履行期限：

2. 【风险点识别】
   列出所有潜在风险，每个风险包括：
   - 风险类型（法律风险/商业风险/遗漏风险）
   - 风险等级（高/中/低）
   - 具体描述
   - 涉及条款

3. 【条款建议】
   对每个有问题的条款给出修改建议

4. 【缺失条款】
   列出应该有但没有的条款

5. 【总体评价】
   对合同的总体评价和建议
"""
        
        response = self.client.chat.completions.create(
            model="gpt-4-turbo",
            messages=[
                {"role": "system", "content": "你是专业的法律顾问"},
                {"role": "user", "content": prompt}
            ],
            temperature=0.3  # 降低随机性，更严谨
        )
        
        return response.choices[0].message.content
    
    def analyze_risk_clause(self, clause_text):
        """针对单个条款进行风险分析"""
        prompt = f"""
分析以下合同条款的风险：

条款内容：
{clause_text}

请分析：
1. 是否存在歧义或模糊表述？
2. 是否对某一方明显不利？
3. 是否符合法律法规？
4. 是否存在执行困难？
5. 建议如何修改？
"""
        
        response = self.client.chat.completions.create(
            model="gpt-4-turbo",
            messages=[{"role": "user", "content": prompt}],
            temperature=0.3
        )
        
        return response.choices[0].message.content

# 使用
reviewer = ContractReviewer()
report = reviewer.review_contract(contract_text)
print(report)
```

### 2. 使用RAG增强审查准确性

```python
from langchain.document_loaders import DirectoryLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.chains import RetrievalQA
from langchain.llms import OpenAI

class RAGContractReviewer:
    def __init__(self):
        # 1. 加载法律知识库
        self.knowledge_base = self.build_knowledge_base()
        
    def build_knowledge_base(self):
        """构建法律知识库"""
        # 加载法律文档
        loader = DirectoryLoader(
            './legal_docs/',  # 法律法规、判例、模板
            glob="**/*.txt"
        )
        documents = loader.load()
        
        # 切分
        splitter = RecursiveCharacterTextSplitter(
            chunk_size=1000,
            chunk_overlap=100
        )
        chunks = splitter.split_documents(documents)
        
        # 向量化并存储
        vectorstore = Chroma.from_documents(
            chunks,
            OpenAIEmbeddings()
        )
        
        return vectorstore
    
    def review_with_rag(self, contract_text, clause_type):
        """基于RAG审查合同"""
        # 1. 检索相关法律条文
        query = f"关于{clause_type}的法律规定和注意事项"
        relevant_docs = self.knowledge_base.similarity_search(query, k=5)
        
        # 2. 构建Prompt
        context = "\n\n".join([doc.page_content for doc in relevant_docs])
        
        prompt = f"""
参考以下法律规定：
{context}

请审查以下合同条款：
{contract_text}

请指出：
1. 是否符合法律规定
2. 是否存在风险
3. 如何改进
"""
        
        # 3. 大模型分析
        client = OpenAI(api_key="your_key")
        response = client.chat.completions.create(
            model="gpt-4-turbo",
            messages=[{"role": "user", "content": prompt}]
        )
        
        return {
            'analysis': response.choices[0].message.content,
            'references': [doc.metadata for doc in relevant_docs]
        }

# 使用
rag_reviewer = RAGContractReviewer()
result = rag_reviewer.review_with_rag(
    contract_text="第五条 违约责任...",
    clause_type="违约责任"
)

print(result['analysis'])
print("\n参考依据：", result['references'])
```

---

## 📊 第四层：合同对比技术

### 1. 文本层面对比（Diff算法）

```python
import difflib
from difflib import SequenceMatcher

class ContractComparator:
    def compare_contracts(self, old_text, new_text):
        """对比两个合同版本"""
        # 按行对比
        old_lines = old_text.splitlines()
        new_lines = new_text.splitlines()
        
        # 生成Diff
        diff = difflib.unified_diff(
            old_lines, 
            new_lines,
            lineterm='',
            n=3  # 上下文行数
        )
        
        # 解析Diff结果
        changes = self.parse_diff(diff)
        
        return changes
    
    def parse_diff(self, diff):
        """解析Diff结果"""
        changes = {
            'added': [],      # 新增内容
            'deleted': [],    # 删除内容
            'modified': []    # 修改内容
        }
        
        for line in diff:
            if line.startswith('+') and not line.startswith('+++'):
                changes['added'].append(line[1:])
            elif line.startswith('-') and not line.startswith('---'):
                changes['deleted'].append(line[1:])
        
        return changes
    
    def generate_html_diff(self, old_text, new_text):
        """生成HTML格式的对比报告"""
        differ = difflib.HtmlDiff()
        html = differ.make_file(
            old_text.splitlines(),
            new_text.splitlines(),
            fromdesc='原版本',
            todesc='新版本',
            context=True,
            numlines=5
        )
        
        return html

# 使用
comparator = ContractComparator()

old_contract = "第一条 甲方应在2024年1月1日前支付100万元..."
new_contract = "第一条 甲方应在2024年3月1日前支付120万元..."

changes = comparator.compare_contracts(old_contract, new_contract)
print("新增：", changes['added'])
print("删除：", changes['deleted'])

# 生成HTML对比报告
html = comparator.generate_html_diff(old_contract, new_contract)
with open('diff_report.html', 'w', encoding='utf-8') as f:
    f.write(html)
```

### 2. 语义层面对比（基于Embedding）

```python
from sentence_transformers import SentenceTransformer, util
import numpy as np

class SemanticComparator:
    def __init__(self):
        # 使用法律领域的句子编码模型
        self.model = SentenceTransformer('law-bert-base')
    
    def compare_clauses_semantic(self, old_clauses, new_clauses):
        """语义层面对比条款"""
        # 1. 计算所有条款的向量
        old_embeddings = self.model.encode(old_clauses)
        new_embeddings = self.model.encode(new_clauses)
        
        # 2. 计算相似度矩阵
        similarity_matrix = util.cos_sim(old_embeddings, new_embeddings)
        
        # 3. 找出变化的条款
        changes = []
        
        for i, old_clause in enumerate(old_clauses):
            max_sim = similarity_matrix[i].max().item()
            most_similar_idx = similarity_matrix[i].argmax().item()
            
            if max_sim < 0.9:  # 相似度阈值
                changes.append({
                    'old_clause': old_clause,
                    'new_clause': new_clauses[most_similar_idx],
                    'similarity': max_sim,
                    'change_type': self.classify_change(max_sim)
                })
        
        return changes
    
    def classify_change(self, similarity):
        """分类变化程度"""
        if similarity < 0.5:
            return '重大修改'
        elif similarity < 0.8:
            return '中等修改'
        else:
            return '轻微修改'
    
    def highlight_differences(self, old_text, new_text):
        """高亮差异部分"""
        # 用大模型分析具体差异
        from openai import OpenAI
        client = OpenAI()
        
        prompt = f"""
对比以下两段文字，指出具体的差异点：

原文：
{old_text}

新文：
{new_text}

请列出所有修改的地方，并说明修改的影响。
"""
        
        response = client.chat.completions.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}]
        )
        
        return response.choices[0].message.content

# 使用
comparator = SemanticComparator()

old_clauses = [
    "第一条 甲方应在合同生效后30日内支付100万元",
    "第二条 乙方应在收到款项后开始供货"
]

new_clauses = [
    "第一条 甲方应在合同签订后一个月内支付人民币壹佰万元整",
    "第二条 乙方应在确认到账后3个工作日内发货"
]

changes = comparator.compare_clauses_semantic(old_clauses, new_clauses)
for change in changes:
    print(f"相似度: {change['similarity']:.2f}")
    print(f"变化类型: {change['change_type']}")
    print(f"原条款: {change['old_clause']}")
    print(f"新条款: {change['new_clause']}")
    print("---")
```

### 3. 结构化对比（关键信息对比）

```python
class StructuredComparator:
    def __init__(self):
        self.ner = ContractNER()  # 前面定义的NER
    
    def compare_key_info(self, old_contract, new_contract):
        """对比关键信息"""
        # 1. 提取两个版本的关键信息
        old_info = self.ner.extract_entities(old_contract)
        new_info = self.ner.extract_entities(new_contract)
        
        # 2. 对比
        comparison = {}
        
        for key in old_info.keys():
            old_values = set(old_info[key])
            new_values = set(new_info[key])
            
            comparison[key] = {
                '删除': list(old_values - new_values),
                '新增': list(new_values - old_values),
                '保持': list(old_values & new_values)
            }
        
        return comparison
    
    def generate_comparison_report(self, comparison):
        """生成对比报告"""
        report = "# 合同对比报告\n\n"
        
        for entity_type, changes in comparison.items():
            if changes['删除'] or changes['新增']:
                report += f"## {entity_type}\n\n"
                
                if changes['删除']:
                    report += f"❌ 删除: {', '.join(changes['删除'])}\n\n"
                
                if changes['新增']:
                    report += f"✅ 新增: {', '.join(changes['新增'])}\n\n"
                
                if changes['保持']:
                    report += f"➡️ 保持: {', '.join(changes['保持'])}\n\n"
        
        return report

# 使用
structured = StructuredComparator()
comparison = structured.compare_key_info(old_contract, new_contract)
report = structured.generate_comparison_report(comparison)
print(report)

# 输出：
# # 合同对比报告
# 
# ## 金额
# ❌ 删除: 100万元
# ✅ 新增: 120万元
# 
# ## 日期
# ❌ 删除: 2024年1月1日
# ✅ 新增: 2024年3月1日
```

---

## 🏗️ 完整系统架构

### 系统组件图

```python
# main.py - 主入口
from fastapi import FastAPI, UploadFile, File
from fastapi.responses import JSONResponse
import uvicorn

app = FastAPI(title="合同审查系统")

# 初始化各个模块
document_parser = DocumentParser()      # 文档解析
contract_ner = ContractNER()           # 实体识别
clause_classifier = ClauseClassifier()  # 条款分类
reviewer = RAGContractReviewer()       # AI审查
comparator = ContractComparator()      # 合同对比

@app.post("/api/upload")
async def upload_contract(file: UploadFile = File(...)):
    """上传合同文件"""
    # 1. 保存文件
    file_path = f"uploads/{file.filename}"
    with open(file_path, "wb") as f:
        content = await file.read()
        f.write(content)
    
    # 2. 解析文档
    if file.filename.endswith('.pdf'):
        text = document_parser.parse_pdf(file_path)
    elif file.filename.endswith('.docx'):
        text = document_parser.parse_word(file_path)
    else:
        return JSONResponse({"error": "不支持的文件格式"}, status_code=400)
    
    # 3. 保存到数据库
    contract_id = save_to_db(file.filename, text)
    
    return {
        "contract_id": contract_id,
        "filename": file.filename,
        "text_length": len(text)
    }

@app.post("/api/review/{contract_id}")
async def review_contract(contract_id: str):
    """审查合同"""
    # 1. 从数据库获取合同
    contract = get_contract_from_db(contract_id)
    
    # 2. 提取实体
    entities = contract_ner.extract_entities(contract['text'])
    
    # 3. 分类条款
    clauses = clause_classifier.split_and_classify(contract['text'])
    
    # 4. AI审查
    review_result = reviewer.review_contract(contract['text'])
    
    # 5. 风险评分
    risk_score = calculate_risk_score(review_result)
    
    return {
        "contract_id": contract_id,
        "entities": entities,
        "clauses": clauses,
        "review": review_result,
        "risk_score": risk_score
    }

@app.post("/api/compare")
async def compare_contracts(old_id: str, new_id: str):
    """对比两个合同"""
    # 1. 获取合同
    old_contract = get_contract_from_db(old_id)
    new_contract = get_contract_from_db(new_id)
    
    # 2. 文本对比
    text_diff = comparator.compare_contracts(
        old_contract['text'],
        new_contract['text']
    )
    
    # 3. 语义对比
    semantic_diff = SemanticComparator().compare_clauses_semantic(
        old_contract['clauses'],
        new_contract['clauses']
    )
    
    # 4. 结构化对比
    key_info_diff = StructuredComparator().compare_key_info(
        old_contract['text'],
        new_contract['text']
    )
    
    return {
        "text_diff": text_diff,
        "semantic_diff": semantic_diff,
        "key_info_diff": key_info_diff
    }

@app.get("/api/report/{contract_id}")
async def generate_report(contract_id: str):
    """生成审查报告"""
    # 生成PDF报告
    pdf_path = generate_pdf_report(contract_id)
    
    return FileResponse(pdf_path)

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

---

## 📦 开源项目推荐

### 1. 文档解析

```bash
# Unstructured - 万能文档解析
pip install unstructured
pip install "unstructured[pdf]"

# PyMuPDF (fitz) - 高性能PDF解析
pip install PyMuPDF

# pdfplumber - PDF表格提取
pip install pdfplumber

# python-docx - Word文档
pip install python-docx
```

### 2. OCR识别

```bash
# PaddleOCR - 最强开源OCR（支持中文）⭐
pip install paddleocr

# Tesseract - 经典OCR
pip install pytesseract

# EasyOCR - 简单易用
pip install easyocr
```

### 3. NLP处理

```bash
# Transformers - Hugging Face
pip install transformers

# spaCy - NLP工具包
pip install spacy
python -m spacy download zh_core_web_sm

# LTP - 哈工大语言技术平台（中文）
pip install ltp
```

### 4. 大模型

```bash
# LangChain - LLM应用框架
pip install langchain langchain-openai

# LlamaIndex - RAG框架
pip install llama-index

# OpenAI
pip install openai
```

### 5. 向量数据库

```bash
# Chroma - 轻量级向量数据库
pip install chromadb

# Milvus - 生产级向量数据库
pip install pymilvus

# Faiss - Facebook的向量搜索
pip install faiss-cpu  # 或 faiss-gpu
```

### 6. 文本对比

```bash
# difflib - Python内置

# python-Levenshtein - 编辑距离
pip install python-Levenshtein

# diff-match-patch - Google的Diff算法
pip install diff-match-patch
```

---

## 🎯 完整技术栈清单

### 后端技术栈

| 分类         | 技术                 | 用途       | 必选 |
| ------------ | -------------------- | ---------- | ---- |
| **Web框架**  | FastAPI/Django       | API服务    | ✅    |
| **文档解析** | PyPDF2/pdfplumber    | PDF解析    | ✅    |
|              | python-docx          | Word解析   | ✅    |
|              | Unstructured         | 通用解析   | ⭐    |
| **OCR**      | PaddleOCR            | 扫描件识别 | ✅    |
| **NLP**      | Transformers         | 预训练模型 | ✅    |
|              | spaCy                | NLP工具    | ⭐    |
| **大模型**   | OpenAI API           | GPT-4      | ✅    |
|              | Qwen                 | 开源方案   | ⭐    |
| **RAG**      | LangChain            | RAG框架    | ✅    |
|              | Chroma/Milvus        | 向量数据库 | ✅    |
| **数据库**   | PostgreSQL           | 关系数据库 | ✅    |
|              | MongoDB              | 文档数据库 | ⭐    |
|              | Redis                | 缓存       | ✅    |
| **对比**     | difflib              | 文本对比   | ✅    |
|              | SentenceTransformers | 语义对比   | ⭐    |

### 前端技术栈

| 分类         | 技术              | 用途       |
| ------------ | ----------------- | ---------- |
| **框架**     | React/Vue         | UI框架     |
| **组件库**   | Ant Design        | UI组件     |
| **PDF预览**  | PDF.js            | 文档预览   |
| **Diff展示** | react-diff-viewer | 对比展示   |
| **编辑器**   | Monaco Editor     | 文本编辑   |
| **图表**     | ECharts           | 数据可视化 |

---

## 💡 实战案例

### 案例1：完整的合同审查流程

```python
class ContractReviewSystem:
    def __init__(self):
        self.parser = DocumentParser()
        self.ner = ContractNER()
        self.reviewer = RAGContractReviewer()
    
    def full_review(self, file_path):
        """完整审查流程"""
        print("1️⃣ 解析文档...")
        text = self.parser.parse_pdf(file_path)
        
        print("2️⃣ 提取关键信息...")
        entities = self.ner.extract_entities(text)
        
        print("3️⃣ AI审查...")
        review = self.reviewer.review_contract(text)
        
        print("4️⃣ 生成报告...")
        report = self.generate_report(text, entities, review)
        
        return report
    
    def generate_report(self, text, entities, review):
        """生成完整报告"""
        return {
            'summary': {
                'parties': entities['甲方'] + entities['乙方'],
                'amount': entities['金额'],
                'dates': entities['日期']
            },
            'review': review,
            'recommendations': self.extract_recommendations(review)
        }

# 使用
system = ContractReviewSystem()
report = system.full_review("合同.pdf")
print(json.dumps(report, ensure_ascii=False, indent=2))
```

---

## 📚 学习资源

### 推荐项目

1. **LegalBERT** - 法律领域BERT模型
   - https://github.com/thunlp/LegalBERT

2. **LaWGPT** - 中文法律大模型
   - https://github.com/pengxiao-song/LaWGPT

3. **智海-录问** - 法律对话模型
   - https://github.com/zhihaiLLM/wisdomInterrogatory

### 论文

- "Legal Judgment Prediction" (法律判决预测)
- "Contract Understanding" (合同理解)
- "Legal Information Extraction" (法律信息抽取)

---

## 🚀 快速开始

### 最小可用版本（MVP）

```python
# 1. 安装依赖
"""
pip install pdfplumber
pip install openai
pip install fastapi uvicorn
"""

# 2. 最简实现
from fastapi import FastAPI, UploadFile
import pdfplumber
from openai import OpenAI

app = FastAPI()
client = OpenAI(api_key="your_key")

@app.post("/review")
async def review(file: UploadFile):
    # 1. 提取PDF文字
    with pdfplumber.open(file.file) as pdf:
        text = "\n".join([p.extract_text() for p in pdf.pages])
    
    # 2. GPT-4审查
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{
            "role": "user",
            "content": f"请审查以下合同，指出风险点：\n\n{text}"
        }]
    )
    
    return {"review": response.choices[0].message.content}

# 3. 运行
# uvicorn main:app --reload
```

只需50行代码，就能实现基础的合同审查功能！

---

## 总结

合同审查AI项目的核心技术栈：

```
📄 文档层：PyPDF2/pdfplumber + PaddleOCR
🧠 NLP层：Transformers + spaCy + 实体识别
🤖 AI层：GPT-4/Qwen + RAG (LangChain + Chroma)
📊 对比层：difflib + SentenceTransformers
🌐 服务层：FastAPI + PostgreSQL + Redis
🎨 前端层：React + Ant Design + PDF.js
```

**建议学习路径**：
1. 先用OpenAI API实现基础版本
2. 再加入文档解析和NER
3. 然后加入RAG提高准确率
4. 最后优化性能和用户体验

有问题随时问我！🚀

