
# Handling Context Window Limitations in LLMs


### **The Core Problem**

**Context Window Constraints:**

- Modern LLMs: 8K-200K tokens (but longer ≠ better)
- **Quadratic attention cost**: O(n²) computation complexity
- **Linear but massive KV cache growth**
- **Cost scales linearly** with token count
- **"Lost in the middle" phenomenon**: Information buried mid-context gets ignored

---

## **Solution 1: Chunking Strategies**

### **A. Fixed-Size Chunking**

- **Method**: Split by token count (e.g., 512 tokens/chunk, 50-token overlap)
- **When to use**: Any content type, need predictability
- **Trade-off**: May break semantic units mid-sentence

**Critical Insight**: Halving chunk size = 2× chunks to index, 2× embeddings, 2× vector search space → slower queries

### **B. Semantic Chunking**

- **Method**: Split by paragraphs, sections, headers, topic boundaries
- **When to use**: Need semantic coherence over size predictability
- **Trade-off**: Variable sizes, more implementation complexity

### **C. Hierarchical Chunking**

- **Method**: Multi-level structure
    - **L1**: Document summaries (100 tokens) → fast, broad search
    - **L2**: Section summaries (500 tokens) → narrow down
    - **L3**: Full sections (2000 tokens) → fetch details
- **Real-world benefit**: Cost-effective retrieval, only load details when needed
    

---

## **Solution 2: Retrieval-Augmented Generation (RAG)**

### **Core Architecture**

```
Large Knowledge Base (millions of docs)
    ↓ Query
Retrieve top-k relevant chunks (3-10)
    ↓ 
Fit in context (3-5K tokens)
    ↓
LLM generates response
```

### **When to Use RAG vs. Long-Context**

- **< 200K tokens** (~500 pages): Include everything in prompt
- **> 200K tokens**: Use RAG
- **Production reality**: Most systems use RAG even for smaller bases due to cost

### **Key Benefits**

- **Scalability**: Handle unlimited documents
- **Freshness**: Update knowledge base without retraining
- **Cost**: Only pay for relevant tokens (20× cheaper than long-context)
- **Quality**: Focused context > dumping everything

---

## **Solution 3: Compression Techniques**

### **A. Prompt Compression (LLMLingua)**

- **Example transformation**:
    - Before: "The product specification document indicates that the maximum print speed capability of the printer model A300 is rated at one hundred pages per minute." (25 tokens)
    - After: "Product spec: A300 max print speed 100 ppm" (8 tokens)
- **Compression ratio**: 70-80% reduction
    
- **Trade-offs**: Adds latency, may lose nuance, better for factual content

### **B. Summarisation Cascading**

- **Step 1**: Summarize 10-page sections → 1 page each
- **Step 2**: Summarize all summaries → final overview
- **Step 3**: Use final summary as context

---

## **Solution 4: Long-Context Models**

### **Model Comparison (2025)**

| Model       | Context Window | ~Pages |
| ----------- | -------------- | ------ |
| GPT-4 Turbo | 128K           | ~400   |
| Claude 4    | 200K           | ~500   |
| Gemini 2.5  | 1M             | ~3,000 |

### **When to Use**
1. Whole document analysis (contracts, papers)
2. Full codebase understanding
3. Multi-document comparison
4. Long conversation history

### **Cost Reality Check**
- **GPT-4 Turbo (100K context)**: $1.00/request
- **RAG (5K context)**: $0.05/request
- **Verdict**: 20× cost difference

### **Quality Caveat**
- "Lost in the middle" problem persists
- Models perform best with relevant info at **beginning or end**
- Extreme length may degrade performance

---

## **Solution 5: Sliding Window Approaches**

### **Fixed Window Strategy**

```
Document: 20K tokens | Context limit: 4K

Window 1: [0-4K] → Generate summary
Window 2: [4K-8K] → Continue from W1
Window 3: [8K-12K] → Continue
Final: Synthesize all outputs
```

### **Overlap Strategy**

- **Window**: 4K tokens
- **Overlap**: 500 tokens between windows
- **Benefit**: Preserve context across boundaries
- **Cost**: Process more total tokens

---

## **Solution 6: Memory Management (Conversational AI)**

### **Architecture**

```python
class ConversationMemory:
    short_term = []  # Last 10-20 full messages
    long_term = {}   # Summarized history beyond N
    
    def add_message(self, message):
        self.short_term.append(message)
        if len(self.short_term) > limit:
            # Summarize oldest → move to long_term
            
    def get_context(self, query):
        # Combine recent full + relevant historical summaries
```

### **Strategy**

- **Short-term**: Keep last 10-20 messages (full detail)
- **Long-term**: Compress older messages into summaries + key facts
- **Retrieval**: Pull relevant history based on current query

---

## **Solution 7: Context Pruning**

### **A. Relevance Filtering**
- Retrieve 50 chunks → score relevance → keep top 8 (threshold 0.7)

### **B. Recency Weighting** (time-sensitive domains)
- Last 24h: Keep all
- Last week: Keep 50%
- Older: Keep 10% most relevant

### **C. Redundancy Elimination**
- Use embedding similarity
- If similarity > 0.9 between chunks → keep only one

---

## **Solution 8: Intelligent Routing**

### **Multi-Model Strategy**

```
Query Analysis
    ↓
Simple factual → Short-context model (cheaper, faster)
Complex analysis → Long-context model (better quality)
Multi-doc reasoning → Specialized model
```

**Benefits**: Cost optimization, quality optimization, latency optimization

---

## **Practical Decision Framework**

### **Decision Tree**

```
1. Information < 200K tokens?
   YES → Use long-context model
   NO ↓

2. Information frequently changing?
   YES → RAG (easy updates)
   NO ↓

3. Need full document coherence?
   YES → Long-context model
   NO ↓

4. Cost primary concern?
   YES → RAG (20× cheaper)
   NO ↓

5. Latency critical?
   YES → Aggressive chunking + caching
   NO → Flexible approach
```

---

## **Testing Context Utilization**

### **RULER Test Approach**

```python
def test_context_usage():
    # Insert "canary" information at positions:
    # [early: 1K, middle: 25K, late: 49K]
    
    response = model.generate(
        context=context_with_canaries,
        query="Find all canary information"
    )
    
    # Check recovery rate by position
    # If only finds early/late → "lost in middle" confirmed
```

---

## **Production Best Practices**

### **1. Monitor Context Usage**

- Average context length/request
- % requests hitting context limit
- Cost per request by context size
- Quality degradation patterns

### **2. Implement Fallbacks**

```
Context exceeds limit:
├─ Fallback 1: Auto-summarize
├─ Fallback 2: Ask user to be more specific
└─ Fallback 3: Process in chunks with synthesis
```

### **3. User Experience**

- ❌ **Don't**: Silently truncate, cryptic errors
- ✅ **Do**: Inform if truncated, suggest refinement, offer chunked processing

---

## **Key Takeaways**

### **The Default Stack**

1. **RAG**: Most cost-effective, scalable (default choice for > 200K tokens)
2. **Long-context models**: When you truly need full document (expensive but necessary)
3. **Semantic chunking**: Balance between size and coherence
4. **Compression**: For verbose sources
5. **Memory systems**: For conversational AI

### **Critical Numbers to Remember**

- **20× cost difference**: RAG vs long-context
- **70-80% compression ratio**: Prompt compression
- **0.9 similarity threshold**: Redundancy elimination
- **10-20 messages**: Typical short-term memory limit
- **"Lost in the middle"**: Info buried mid-context performs worst

#### **In a Nutshell**

_"I'd implement a hybrid approach: RAG as the foundation for cost and scalability, with intelligent routing to long-context models only when document coherence is critical. I'd use semantic chunking with hierarchical retrieval, monitor context utilisation metrics in production, and implement graceful fallbacks for edge cases. The key is matching the solution to specific constraints: cost, latency, quality, and update frequency."_