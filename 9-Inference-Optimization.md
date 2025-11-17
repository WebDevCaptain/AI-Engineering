
### Production Viability of Models

- A model isn't useful in production just because it's accurate - it must also be economically viable and responsive
  
- Real-world example: A next-day stock price prediction model that takes 2 days to compute is useless, no matter how accurate
  
- This establishes the core motivation: optimization isn't optional, it's what makes models production-ready

---
### Measurement and Metrics Framework

- You can't optimize what you don't measure - metrics are your diagnostic tools
  
- Three categories of metrics matter:
    1. **Latency**: User experience (how fast responses appear)
    2. **Throughput**: Cost efficiency (how many requests you can process)
    3. **Utilization**: Hardware efficiency (how well you're using your expensive GPUs)

---
### Latency Decomposition (TTFT vs TPOT)

- **Why this split matters:** Language models have two distinct phases during inference
    - **Prefill phase** (input processing) → determines TTFT
    - **Decode phase** (token generation) → determines TPOT

- These phases have completely different computational characteristics:
    - Prefill is **compute-bound** (limited by how many operations your chip can do)
    - Decode is **memory bandwidth-bound** (limited by how fast you can move data)

- Understanding this split is crucial because different optimizations target different phases

---

### Prefill vs Decode Phases

- **Prefill = parallel processing**: Your prompt "The cat sat on the" can process all 5 tokens simultaneously—speed limited by computational power
- **Decode = sequential generation**: Must generate "mat" before "." -one token at a time, must load entire model weights into memory for each token
- This sequential constraint is THE fundamental bottleneck-each token generation requires transferring the entire model's parameters from memory to compute units

**Critical insight**: A single output token has the same latency impact as 100 input tokens because of this sequential bottleneck

---

### Latency-Throughput Trade-offs

- **Batching increases throughput BUT increases latency:** Processing 10 requests together is more efficient but makes each request wait
  
- Think of it like a bus vs individual cars:
    - Bus (batching) = moves more people per trip (throughput ↑) but people wait for the bus (latency ↑)
    - Individual cars = immediate departure (latency ↓) but fewer people moved overall (throughput ↓)

- LinkedIn AI team found they could double or triple throughput by accepting longer TTFT/TPOT
  
- This is why the metric "goodput" exists: requests per second that actually meet your latency requirements (SLO)

---

### Hardware Bottlenecks and Specifications

- Hardware has three critical specs that determine your bottleneck:
    1. **FLOP/s**: How many operations per second (for compute-bound work)
    2. **Memory bandwidth**: How fast data moves (for bandwidth-bound work)
    3. **Memory capacity**: How much data you can store

- Different models stress different specs:
    - Image generation (Stable Diffusion) = compute-bound → needs high FLOP/s
    - Autoregressive LLMs = memory bandwidth-bound → needs high bandwidth

- Optimization must match your bottleneck: Adding more FLOP/s won't help if you're bandwidth-bound

---

### Model-Level vs Service-Level Optimization

- There are two fundamentally different approaches to optimization:
    - **Model-level**: Modify the model (like crafting better arrows) → CAN affect quality
    - **Service-level**: Change how you serve the model (like improving the shooting process) → Shouldn't affect quality


**Model-level techniques include:**

- **Quantization**: Reduce precision (32-bit → 16-bit → 8-bit) = halve memory footprint
    - Most popular because it's easy and extremely effective
    - But we're near the limit (can't go below 1 bit)
      
- **Distillation**: Train a small model to mimic a large one
  
- **Important caveat**: Different inference providers using the same Llama model can show different benchmark scores because their optimization techniques alter model behavior slightly

---

### Attention Mechanism Bottlenecks

- **Why attention is a bottleneck**:
    - To generate each new token, you need the key and value vectors for ALL previous tokens
    - This creates the **KV cache** (key-value cache) that stores these vectors
    - KV cache size grows with: batch size × sequence length × number of layers × model dimension


**Real-world scale**: For a 500B+ parameter model with batch 512 and context 2048, the KV cache = 3 TB (3× the model weights!)


**Three approaches to optimize attention:**

1. **Redesign the mechanism**: Local windowed attention, multi-query attention, cross-layer attention
    - Example: Character.AI achieved 20× KV cache reduction, making memory no longer a bottleneck
      
2. **Optimize KV cache management**: PagedAttention (used by vLLM) divides cache into blocks, reduces fragmentation
   
3. **Write custom kernels**: FlashAttention fuses operations together, hardware-aware optimization

---

### Autoregressive Decoding Bottleneck

- **The core problem**: Sequential dependency—you MUST generate token N before token N+1
- **Why this is expensive**:
    - Generating 100 tokens sequentially takes 10 seconds if each token is 100ms
    - Output tokens cost 2-4× more than input tokens in API pricing


**Two approaches to overcome this:**

1. **Speculative Decoding** (use a draft model):
    - Fast draft model generates K tokens → Target model verifies in parallel
    - Key insight: Verification is parallelizable (fast), generation is sequential (slow)
    - Effectively converts decode's computational profile into prefill's profile
    - If all K draft tokens accepted, you get K+1 tokens in one iteration

2. **Parallel Decoding** (break sequential dependency):
    - Generate multiple future tokens simultaneously without knowing previous tokens
    - Works because existing context often sufficient to predict next few tokens
    - Example: Given "the cat sits", you can predict "the" comes after the next word, even without knowing if it's "on the" or "under the"

---

### Batching and Parallelism Strategies

**Batching evolution:**

- **Static batching**: Wait until batch is full (like a bus that waits for every seat) → first request delayed
  
- **Dynamic batching**: Process when batch full OR timeout (like a scheduled bus) → controls latency but may waste compute
  
- **Continuous batching**: Return completed responses immediately, fill slots with new requests → maximizes efficiency, minimizes latency for short requests


**Parallelism strategies:**

- **Replica parallelism**: Multiple copies of the model → handle more concurrent requests (straightforward but costs more)
  
- **Tensor parallelism**: Split operators across devices → enables serving models too large for one machine AND reduces latency
  
- **Pipeline parallelism**: Split model into stages across devices → enables large models but increases latency (avoid for inference)

---

### LLM-Specific Service Optimizations (Prefill/Decode Decoupling & Prompt Caching)

**Prefill/Decode Decoupling:**

- **The problem**: Prefill (compute-bound) and decode (bandwidth-bound) compete for resources on the same GPU
  
- **The solution**: Use separate GPU instances for prefill vs decode
  
- **Why it works**: Each phase can now optimize for its specific bottleneck without interference
  
- **Practical**: Ratio of 2:1 to 4:1 (prefill:decode) if long inputs; 1:2 to 1:1 if short inputs


**Prompt Caching:**

- **The problem**: Many prompts share overlapping segments (especially system prompts)
  
- **The solution**: Cache overlapping segments, process them only once
  
- **Impact example**: 1,000-token system prompt × 1 million daily requests = processing 1 billion repetitive tokens without caching
  
- **Real results** (Anthropic):
    - 100K-token cached prompt → 79% latency reduction, 90% cost reduction
    - Multi-turn convo → 75% latency reduction, 53% cost reduction

---

### Workload-Dependent Optimization Selection

**Context length matters:**
- **Long contexts** → KV caching and attention optimization are CRITICAL
- **Short contexts** → KV caching less important, focus elsewhere


**Prompt patterns matter:**
- **Long, overlapping prompts** (like system prompts) → Prompt caching crucial
- **Multi-turn conversations** → Prompt caching for message history
- **Unique prompts** → Prompt caching provides little benefit


**Performance priorities matter:**
- **Low latency critical** → Scale up replica parallelism (costs more but faster per request)
- **Cost critical** → Aggressive quantization, continuous batching, prompt caching
- **Large models** → Must use tensor or pipeline parallelism to fit on hardware

---

### High-Impact Universal Techniques

These four are universally valuable across use cases:

1. **Quantization**: Works well across models, easy to implement, near-universal applicability
   
2. **Tensor parallelism**: Two benefits—enables serving models too large for one machine AND reduces latency
   
3. **Replica parallelism**: Straightforward implementation, scales throughput linearly
   
4. **Attention optimization**: Significant acceleration for transformer models (which is most modern LLMs)

**The practical takeaway**: Start with these four before exploring more complex techniques. They give you the most leverage for the effort.

---

### Bottleneck-Driven Optimization Framework

Inference optimization is fundamentally about matching solutions to bottlenecks. Framework:

1. Identify your bottleneck (compute vs bandwidth vs memory capacity)
2. Understand your workload characteristics (context length, prompt patterns, latency requirements)
3. Select techniques that address YOUR specific bottleneck and workload
4. Accept trade-offs consciously (speed vs cost vs quality)