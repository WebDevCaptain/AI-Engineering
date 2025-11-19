
## **What is RAG?**

**Mechanism:**

- Converts queries into embeddings → searches knowledge base → retrieves relevant documents → injects context into LLM prompt
- Model weights remain unchanged; knowledge is provided at inference time
- Think: "giving the model a textbook during the exam"

**When to use RAG:**

- **Dynamic/frequently updated information** (stock prices, news, product catalogs)
- **Proprietary/private data** (company docs, customer records, internal wikis)
- **Attribution requirements** (need to cite sources for answers)
- **Large knowledge bases** (millions of documents - cheaper than training)
- **Information-based failures** (model gives wrong facts, not wrong format)

**Real example:** Customer support chatbot accessing current product documentation and ticket history

---

## **What is Finetuning?**

**Mechanism:**

- Continues training on task-specific data → updates model weights → changes model's behavior/style/format
- Knowledge becomes embedded in parameters
- Think: "teaching the model a new skill or style"

**When to use Finetuning:**

- **Consistent output format** (always return JSON, specific structure)
- **Style/tone adaptation** (formal legal language, medical terminology)
- **Task-specific behavior** (classification, extraction, summarization patterns)
- **Reducing verbosity** (model is too chatty, finetuning makes it concise)
- **Proprietary workflows** (multi-step reasoning patterns unique to your domain)

**Real example:** Legal contract analysis system that must output standardized clause classifications

---

## **Key Trade-offs**

|Aspect|RAG|Finetuning|
|---|---|---|
|**Cost**|Low initial (just embeddings)|High (GPU training hours)|
|**Update speed**|Instant (update knowledge base)|Slow (retrain required)|
|**Accuracy on facts**|High (retrieves exact info)|Can hallucinate memorized facts|
|**Latency**|Higher (retrieval step)|Lower (single forward pass)|
|**Transparency**|High (can see retrieved docs)|Black box|

---

## **When to Combine Both** ⭐

**The power move:** RAG + Finetuning together

- Finetune for: Output format, reasoning style, domain language
- RAG for: Factual knowledge, current information, specific examples

**Concrete example:**

- Medical diagnosis assistant
- **Finetuned on:** Medical reasoning patterns, clinical note formatting, appropriate hedging language
- **RAG for:** Latest research papers, drug interaction databases, patient history

**Evidence:** Research shows 43% improvement in MMLU when combining both vs. either alone

---

## **Decision Framework** (Practical)

**Start with RAG if:**

- Knowledge changes frequently
- You need explainability
- Budget is limited
- Fast iteration required

**Choose Finetuning if:**

- Model's _behavior_ is wrong, not its knowledge
- You have 1000+ high-quality training examples
- Latency is critical
- Output format must be perfectly consistent

**Use both if:**

- You're building production systems at scale
- You need both behavior change AND dynamic knowledge
- You have the resources for both approaches

---

## Common Pitfall 🎯

_"Many teams over-index on finetuning when RAG would solve their problem faster and cheaper. If your LLM says 'I don't know about X' but answers correctly when you paste the info in context, that's a RAG problem, not a finetuning problem."_