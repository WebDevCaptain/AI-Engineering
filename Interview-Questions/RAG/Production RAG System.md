
## Q. Design a production RAG system at scale

### **System Architecture Overview**

A production RAG system requires two distinct pipelines working in harmony:

### **Pipeline 1: The Indexing Pipeline (Offline Processing)**

**Purpose:** Prepare your knowledge base for efficient retrieval

**Components in Order:**

1. **Data Collection**
    - Fetch from multiple sources: documents, databases, APIs, websites
    - Handle diverse formats: PDF, DOCX, HTML, markdown, structured data
    - Example: Scraping company knowledge repositories, uploading technical manuals
2. **Preprocessing**
    - Clean raw text: remove noise, standardize formats
    - Fix inconsistencies, handle special characters
    - Remove or anonymize PII (Personally Identifiable Information)
    - Filter out toxic or low-quality content
3. **Chunking Strategy** (Critical Decision Point)
    - **Why chunk?** Naively retrieving whole documents creates arbitrarily long contexts
    - **Chunk size trade-offs:**
        - Too small → Loses context, increases retrieval calls
        - Too large → Exceeds context windows, reduces precision
    - **Strategies:**
        - Fixed-size chunking (e.g., 512 tokens with 50-token overlap)
        - Semantic chunking (split by paragraphs, sections, topics)
        - Hierarchical chunking (summaries → detailed chunks)
4. **Embedding Generation**
    - Convert text chunks into vector representations
    - Use pre-trained models: Sentence Transformers, OpenAI Ada, BERT
    - Each chunk becomes a high-dimensional vector (e.g., 768 or 1536 dimensions)
5. **Vector Storage**
    - Store embeddings in specialized vector databases
    - Index for fast similarity search
    - Maintain metadata: document ID, chunk position, timestamps

### **Pipeline 2: The Generation Pipeline (Online/Real-time)**

**Purpose:** Respond to user queries with retrieved context

**Flow:**

1. **Query Processing**
    - User submits query
    - Preprocess query (same cleaning as indexing)
    - Generate query embedding using same model as indexing
2. **Retrieval Phase**
    - **Vector Search:** Find top-k most similar chunks (typically k=3-10)
    - **Distance Metrics:** Cosine similarity, Euclidean distance, or dot product
    - **Hybrid Approach** (Best Practice):
        - Combine term-based (BM25) with vector search
        - BM25 excels at keyword matching (product codes, error messages)
        - Vector search captures semantic meaning
        - Retrieve top-50 candidates from each, merge results
3. **Reranking** (Performance Boost)
    - Use cross-encoder models (Cohere Rerank, specialized BERT)
    - More expensive but more accurate than initial retrieval
    - Rerank top-50 to get best 3-10 chunks
4. **Context Assembly**
    - Join retrieved chunks with user prompt
    - Add explicit instructions: "Answer based only on the provided context"
    - Format: System prompt + Retrieved context + User query
5. **LLM Generation**
    - Feed assembled prompt to generative model
    - Model generates response grounded in retrieved facts
6. **Post-processing**
    - Add source citations
    - Validate output format
    - Apply safety filters

### **Scalability Considerations**

**1. Vector Database Selection**

- **Pinecone**: Fully managed, easy to scale, good for startups
- **FAISS**: Facebook's library, fast, requires self-hosting
- **Qdrant**: Open-source, good for on-premise deployments
- **Weaviate/Milvus**: Full-featured, good for complex use cases

**2. Caching Strategy**

- **Query caching**: Store results for frequent queries
- **Embedding caching**: Cache embeddings for common queries
- **Hot-query identification**: Identify trending searches, pre-compute results
- **KV cache**: Reuse computed attention values

**3. Sharding and Partitioning**

- Shard documents by topic, date, or category
- Enable distributed retrieval across shards
- Parallel processing for multiple shards

**4. Retrieval Optimization**

- **Indexing algorithms**: HNSW (Hierarchical Navigable Small Worlds), IVF (Inverted File Index)
- **Quantization**: Reduce vector dimensions for faster search
- **Approximate Nearest Neighbor (ANN)**: Trade slight accuracy for massive speed gains

### **Monitoring and Evaluation**

**Key Metrics:**

1. **Latency Metrics**
    - Time to first token (TTFT)
    - End-to-end latency (P50, P95, P99)
    - Retrieval time vs. generation time breakdown
2. **Quality Metrics**
    - **Context Precision**: % of retrieved chunks that are relevant
    - **Context Recall**: % of relevant info that was retrieved
    - **Answer Relevancy**: How well the final answer addresses the query
    - **Faithfulness**: Whether answer is grounded in retrieved context (no hallucinations)
3. **Cost Metrics**
    - API calls per query
    - Token usage (input + output)
    - Vector search costs
    - Cache hit rates
4. **System Health**
    - Vector DB query performance
    - Cache effectiveness
    - Error rates and types
    - Throughput (queries per second)

### **Production Best Practices**

1. **Continuous Updates**
    - Implement pipeline to regularly update vector database
    - Handle document additions, modifications, deletions
    - Version control for embeddings
2. **Feedback Loops**
    - Collect user feedback (thumbs up/down, relevance ratings)
    - Use feedback to improve retrieval ranking
    - Identify gaps in knowledge base
3. **Fallback Strategies**
    - When confidence is low, admit uncertainty
    - Provide multiple sources when information conflicts
    - Route to human support for critical failures
4. **Security Considerations**
    - Access control for sensitive documents
    - PII detection and masking
    - Audit logging for all retrievals
    - Rate limiting to prevent abuse

### **Architecture Decision: When to Avoid RAG**

According to Anthropic's guidance:

- If your entire knowledge base is **< 200,000 tokens (~500 pages)**, you can include it directly in the prompt
- Long-context models (Claude 4.5 with 200K tokens, GPT-5 with ~200K - 400K tokens) may eliminate RAG need for smaller knowledge bases
- However, RAG still provides:
    - Cost savings (only retrieve relevant parts)
    - Ability to handle millions of documents
    - Dynamic knowledge updates without prompt changes