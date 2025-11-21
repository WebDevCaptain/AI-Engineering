
# How would you optimize inference latency and cost for an LLM application?


## **I. Framework: The Optimization Triangle**

**Three Metrics in Tension**:

- **Latency**: Time to complete request (TTFT + TPOT × tokens)
- **Throughput**: Requests/second capacity
- **Cost**: $/1M tokens or $/request

**Key Insight**: You cannot optimize all three simultaneously. Start by clarifying which matters most for your use case.

**For Autoregressive Models**:

```
Total Latency = TTFT (prefill) + (num_tokens × TPOT (decode))
```

- **TTFT dominates** for short outputs
- **TPOT dominates** for long outputs

---

## **II. Model Selection + Quantization**

### **Right-Sizing First**

Most critical decision: smallest model that meets quality bar.

**Real Example** (1000 req/day, 500 tokens avg):

- GPT-4: $450/month, best quality, slowest
- Llama 2-13B (self-hosted): $83/month, good quality, 3x faster
- **82% cost reduction** with acceptable quality trade-off

### **Quantization Stack**

|Precision|Memory|Quality Loss|Decision|
|---|---|---|---|
|FP16|26 GB|~0%|**Production standard**|
|INT8|13 GB|1-2%|**Sweet spot** for most use cases|
|INT4|6.5 GB|3-5%|Aggressive optimization|

**Measured Impact** (Llama-7B):

```
FP32 baseline:     1.0x
+ torch.compile:   1.5x
+ INT8:            2.0x  ← Recommend starting here
+ INT4:            3.0x
```

---

## **III. Batching: The Throughput Multiplier**

### **Static vs. Continuous Batching**

**Static Batching**:

- Wait for N requests → process together
- 10 sequential requests (100ms each) = 1000ms
- Batched (10 requests) = 200ms → **5x throughput**

**Continuous Batching** (Modern approach):

- Don't wait for entire batch to finish
- As sequences complete, immediately add new requests
- Eliminates "batch bubbles"
- **vLLM, TensorRT-LLM use this**

**Trade-off**: Slight latency increase (waiting time) for massive throughput gain.

---

## **IV. KV Cache Optimization**

**Problem**: KV cache often becomes the bottleneck (grows linearly with sequence length).

### **Three Techniques**

**1. KV Cache Quantization**

- Store K,V in INT8 vs FP16
- **2x memory reduction**, minimal quality loss
- Enable longer contexts or larger batches

**2. PagedAttention (vLLM)**

- **Problem**: Contiguous memory allocation causes fragmentation
- **Solution**: Block-based memory management (like OS paging)
- **Impact**: 2x throughput improvement + memory sharing

**3. Selective Cache Eviction**

- Keep high-attention-score tokens, evict low-importance history
- Critical for very long contexts (100K+ tokens)

**When to Use**: If GPU memory exhausted before compute saturated.

---

## **V. Caching: The Hidden Performance Lever**

### **1. Prompt Caching** (Highest ROI for RAG)

**Scenario**: Multi-turn chat with fixed context

```
Turn 1: [1000-token context] + "What is X?"
Turn 2: [same context] + "What is Y?"
```

**Solution**: Cache prefill computation for repeated context

- **80%+ TTFT reduction** for cache hits
- Supported: Anthropic, OpenAI, Gemini
- **Critical** for RAG applications

### **2. Semantic Caching**

- Hash query embeddings → find similar past queries
- If similarity > threshold, return cached response
- **Trade-off**: Risk of incorrect cache hits vs. latency/cost savings

**Hit Rate Impact**:

- 20% hit rate = 20% cost reduction (for cached queries)
- Most valuable for FAQ-style workloads

---

## **VI. Speculative Decoding**

**Concept**: Small model drafts, large model verifies in parallel.

**Process**:
1. Small model generates k tokens fast
2. Large model verifies all k tokens in **one forward pass**
3. Accept correct predictions, continue from last correct

**Why It Works**:
- Verifying k tokens ≈ cost of generating 1 token (parallelizable)
- Small model ~70-80% accurate → most drafts accepted
- **2-3x speedup** with same quality

**Requirement**: Models must share vocabulary and be architecturally similar.

---

## **VII. Parallelism for Scale**

### **When to Use Each**

**Tensor Parallelism**:
- **When**: Model doesn't fit on single GPU (70B+)
- **How**: Split weights across GPUs, synchronize activations
- **Cost**: Communication overhead

**Pipeline Parallelism**:
- **When**: Very large models, multiple GPUs available
- **How**: Different layers on different GPUs
- **Challenge**: Pipeline bubbles reduce efficiency

**Replica Parallelism**:
- **When**: Need higher throughput, not memory-constrained
- **How**: Multiple model copies + load balancer
- **Benefit**: Lower latency per request (less load per replica)

**Insight**: Most production systems use **Tensor + Replica** hybrid.

---

## **VIII. Cost Framework**

### **Quick Calculation**

```python
cost_per_request = hardware_cost_per_hour / (
    (throughput_tokens_per_second * 3600) / avg_tokens_per_request
)

# Example: $2/hr GPU, 100 tok/s, 200 tok/req
# = $2 / (100*3600/200) = $0.0011 per request
```

### **Batch vs. Online APIs**

- **Online**: <100ms latency, full cost
- **Batch**: Hours delay, **50% cost reduction** (OpenAI, Gemini)
- **Decision**: Use batch for offline workloads (embeddings, synthetic data)

---

## **IX. Production Optimization Playbook**

### **Systematic Approach**

1. **Baseline Metrics**: Establish P95 latency, throughput, $/1M tokens
2. **Model + Quantization**: Right-size model, apply INT8 → **2x speedup**
3. **Batching**: Continuous batching → **3x throughput**
4. **KV Cache**: PagedAttention → **1.5x capacity**
5. **Prompt Caching**: For RAG contexts → **80% TTFT reduction**
6. **Monitor & Iterate**: Track P95/P99 metrics, optimize based on patterns

**Realistic Outcome**: **5-10x total cost reduction** in production.

---

## **X. Monitoring Dashboard**

**Critical Metrics** (track percentiles, not averages):

**Latency**:
- P50, P95, P99 TTFT
- P50, P95, P99 TPOT
- P99 matters most for user experience

**Throughput**:
- Tokens/second
- Batch utilization %
- GPU/CPU utilization

**Cost**:
- $/1M tokens
- $/request
- Hardware efficiency

**Quality**:
- Model accuracy/quality scores
- User feedback (thumbs up/down rate)

---

## **XI. Interview Response Framework**

### **When Asked**: _"How would you optimize inference for our LLM application?"_

**Step 1 - Clarify** (30 seconds):

> "First, I'd establish our primary constraint: Are we optimizing for latency, throughput, or cost? And what's our quality bar?"

**Step 2 - Strategy** (60 seconds):

> "I'd take a layered approach:
> 
> **Model Layer**: Right-size the model—often a 13B model with INT8 quantization gives 90% of GPT-4's quality at 10% of the cost. That's a 2x memory reduction with minimal quality loss.
> 
> **Serving Layer**: Implement continuous batching with PagedAttention. This typically gives 3-5x throughput improvement while managing KV cache efficiently—the actual bottleneck in most production systems.
> 
> **Caching Layer**: Prompt caching is critical, especially for RAG. If you're repeatedly processing the same context with different queries, you can reduce TTFT by 80%+."

**Step 3 - Example** (30 seconds):

> "For instance, in a RAG chatbot I'd use: Llama-13B-INT8 with vLLM for continuous batching and PagedAttention, prompt caching for the retrieval context, and semantic caching for common queries. This typically achieves 5-10x cost reduction while maintaining quality."

**Step 4 - Monitoring** (15 seconds):

> "I'd track P95 TTFT, tokens/second throughput, and cost per million tokens—then iterate based on real production patterns."

---

## **Key Differentiators**

1. **Lead with trade-offs**: Show you understand optimization is about choices, not universal solutions
2. **Use precise numbers**: "2x speedup with INT8" beats "faster performance"
3. **Reference modern tools**: vLLM, PagedAttention, TensorRT-LLM shows current expertise
4. **Think in systems**: Combine techniques strategically (batching + caching + quantization)
5. **Production-focused**: Emphasize monitoring, iteration, and realistic outcomes

> Optimization is multi-dimensional; success comes from systematically combining techniques based on your specific constraints and continuously measuring impact.