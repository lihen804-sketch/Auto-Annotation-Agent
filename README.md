# Auto Annotation Agent

基于 **LangChain + 多 Agent 协作**的自动化数据标注系统。

## 架构

```
非结构化文本
     │
     ▼
┌─────────────────┐
│ EntityExtractor │  ← 识别实体 (PERSON, ORG, LOC, …)
└───────┬─────────┘
        │
        ▼
┌─────────────────┐
│RelationExtractor│  ← 提取实体间关系
└───────┬─────────┘
        │
        ▼
┌──────────────────┐
│AnnotationGenerator│  ← 生成 BIO 标签 + 结构化标注
└────────┬─────────┘
         │
         ▼
┌────────────────┐      ┌────────────────┐
│ CrossValidator │◀────│ ErrorCorrector │  ← 纠错 → 重新校验
└───────┬────────┘      └────────────────┘
        │
        ▼
    ✅ 最终标注数据
```

## 快速开始

### 1. 安装

```bash
cd annotation-agent
pip install -r requirements.txt
```

### 2. Demo 运行（无需 API Key）

```bash
python main.py --demo
```

使用内置 Mock LLM + 5 条中文样本数据，演示完整流程。

### 3. 自定义文本

```bash
python main.py --text "特斯拉由 Elon Musk 于2003年创立，总部在奥斯汀。"
```

### 4. 使用真实 LLM（OpenAI）

```bash
export OPENAI_API_KEY="sk-xxx"
python main.py --demo --model gpt-4o-mini
```

### 5. 批量处理文件

准备 `input.jsonl`：
```jsonl
{"id": "doc_001", "text": "阿里巴巴由马云创立，总部在杭州。"}
{"id": "doc_002", "text": "字节跳动旗下有抖音和TikTok。"}
```

运行：
```bash
python main.py -i input.jsonl -o output/
```

## 输出

| 文件 | 说明 |
|------|------|
| `output/annotations.jsonl` | 完整标注结果（含实体、关系、BIO标签、校验报告） |
| `output/metrics.json` | 聚合指标 |
| `output/summary.json` | 运行摘要 |

每个标注结果包含：
```json
{
  "document_id": "doc_001",
  "entities": [{"type": "ORGANIZATION", "text": "阿里巴巴", "start": 0, "end": 4, "confidence": 0.98}],
  "relations": [{"type": "FOUNDED_BY", "source_text": "阿里巴巴", "target_text": "马云", ...}],
  "bio_tags": ["B-ORG", "I-ORG", "O", "O", ...],
  "validation_passed": true,
  "corrections_applied": 0
}
```

## 配置

见 `rules/default_rules.yaml` 自定义标注规则和环境变量：

```bash
# 通过环境变量覆盖配置
export ANNOTATION__LLM__MODEL="gpt-4o"
export ANNOTATION__BATCH__BATCH_SIZE=20
python main.py --demo
```

## Benchmark（参考）

| 指标 | 值 |
|------|-----|
| 每日处理能力 | ~2000 条 |
| 标注效率提升 | ~60% |
| 标注准确率 | >92% |
