# AI Chatbot Pipeline Documentation
## Overview Proposed Workflow
![Image](images/chatbot_pipeline.png)

## Detailed Pipeline
## 1. Chatbot Configuration

### Steps:
1. **Create Chatbot** → Generate unique ID
2. **Set Name & Purpose** → Define chatbot identity
3. **Configure Initial Prompt** → Set personality and constraints
4. **Save Configuration** → Store in database with dedicated vector namespace

### Initial Prompt Example:
```
You are a professional assistant. Answer only from provided documents.
Always cite sources. Be concise and accurate.
```

![Image](images/chatbot_config.png)

## 2. Document Processing

### 2.1 Upload & Extraction
- **Supported**: PDF (PDFPlumber), DOCX (python-docx), TXT
- **Extracts**: Text, tables, structure, metadata

### 2.2 Text Processing
- Clean text (remove special characters, fix encoding)
- Preserve document structure (headings, lists, tables)

### 2.3 Late-Chunking + semantic chunking
- **Chunk size**: 8192 tokens (Using jinai)

### 2.4 Metadata
Each chunk includes:
- Source document
- Page number
- Section info
- Position

![Image](images/process_doc_&_chunk.png)

## 3. Embedding & Storage

### 3.1 Custom Embedding Model
- Custome embedding model (greennode)
- Output: 1024 dimension vectors

### 3.2 Vector Database
**Options**: Milvus, Pinecone, or Qdrant
- Store embeddings with metadata
- Separate namespace per chatbot

![Image](images/embedding.png)

## 4. Query Processing

### 4.1 Input Processing
1. Receive user query
2. Preprocess 
3. Check auto-reply rules → Return if matched

### 4.2 Query Enhancement
- **Expansion**: Add synonyms and related terms
- **Decomposition**: Split multi-part queries
  ```
  "What's the price and delivery time?" 
  → ["What's the price?", "What's the delivery time?"]
  ```

![Image](images/query_processing.png)

## 5. Multi-RAG Retrieval

### 5.1 Hybrid Search (Per Sub-query)

**Hybrid searches**:
1. **Semantic Search**: Vector similarity
2. **Keyword Search**: BM25 algorithm

### 5.2 Weighted Reranking
```
final_score = (0.6 × semantic_score) + (0.4 × keyword_score)
```

### 5.3 Retrieval
- Get top 5-10 chunks per sub-query
- Final re-ranking and filtering
- Remove duplicates

### 5.4 Context Building
- Aggregate results from all sub-queries
- Deduplicate information
- Order logically

![Image](images/RAG.png)

## 6. Response Generation

### 6.1 Prompt Construction
```
System: [Chatbot configuration]
Context: [Retrieved chunks with sources]
Query: [User question]
Instructions: Answer only from context, cite sources
```

### 6.2 Gemini LLM
- Temperature: 0.3-0.5
- Generate response with citations

![Image](images/LLM_Gen.png)

## 7. Validation

### 7.1 Checks
1. **Content Verification**: Ensure claims match sources
2. **Scope Check**: Remove any out-of-document content
3. **Add Citations**: Format: `[Source: document.pdf, page X]`
4. **Safety Checks**: Filter inappropriate content

### 7.2 Regeneration
If validation fails → Modify and regenerate

![Image](images/validation.png)

## 8. Output

### 8.1 Response Delivery
- Formatted answer with citations
- Return to user via original channel

### 8.2 Logging
- Response time
- Query count  
- Error rate

![Image](images/monitor.png)

## Pipeline Flow Summary

```
User Query → Auto-Reply Check → Query Processing → Multi-RAG Search 
→ Context Building → Gemini Generation → Validation → Response
```


