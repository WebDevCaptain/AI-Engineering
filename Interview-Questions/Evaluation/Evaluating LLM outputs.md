
## How do you evaluate LLM outputs in production?

### **The Core Challenge**

LLM evaluation is fundamentally harder than traditional ML because there's no single correct answer, quality is subjective, and multiple valid responses exist. However, systematic evaluation is critical - you cannot rely on gut feeling or anecdotes.

### **My Three-Tiered Evaluation Approach**

#### **Tier 1: Exact/Automatic Evaluation (Fast, Cheap, Scalable)**

**1. Functional Correctness**

- For code generation or math: Execute the code and verify outputs against unit tests
- Example: Using HumanEval benchmark - does the generated code pass all test cases?
- Limitation: Only works when there's a definitive correct answer

**2. Similarity Metrics (When Reference Data Exists)**

- **BLEU/ROUGE**: Good for translation/summarization but limited
- **BERTScore**: Better semantic similarity using embeddings
- **Exact Match**: For classification tasks
- _Critical limitation_: Requires expensive reference data, and multiple valid responses may score poorly

**3. Log Probabilities (Often Overlooked but Powerful)**

- Measure model confidence: High logprob (95%) = confident, low (33%) = uncertain
- Compare logprobs across classes for classification tasks
- Calculate perplexity for fluency measurement
- Early detection of low-quality outputs before expensive evaluation

---

#### **Tier 2: AI as Judge (Most Common Production Method as of 2025)**

**Implementation Strategy:**

```
System Prompt: Evaluate based on:
1. Relevance to query
2. Factual accuracy  
3. Completeness
4. Clarity

Rate each 1-5 with justification
```

**Critical Considerations I Always Apply:**

**a) Judge Selection & Consistency**

- Use GPT-5/Claude 4.5 Sonnet (or Opus) for high-stakes evaluation
- Never mix judges - scores aren't comparable across different models
- Version-lock your judge to prevent drift

**b) Judge Drift Problem**

- AI judges improve with updates, making them unreliable as fixed benchmarks
- Solution: Always version-lock the specific model (e.g., gpt-4-0613)

**c) Cost Optimization Strategy**

- Run cheap classifiers on 100% of data for low-quality signals
- Run expensive AI judges (GPT-5 / Opus 4.1) on 1% sampled data for high-quality signals
- Balance confidence requirements with budget

**d) Prompt Engineering for Judges**

- Provide clear, unambiguous scoring rubrics with examples
- Use structured outputs (JSON) for consistent parsing
- Pairwise comparison often beats absolute scoring

---

#### **Tier 3: Human Evaluation (The North Star)**

**When I Use Human Evaluation:**

- Sensitive domains (medical, legal, financial)
- Final quality gate before production deployment
- Calibrating AI judges periodically
- Understanding edge cases and failure modes

**Practical Implementation:**

- **Smart Sampling**: Don't evaluate everything - too expensive
    - Example: 500 daily conversations (like LinkedIn does)
    - Oversample edge cases and failures
    - Stratify by difficulty: easy, medium, hard queries

- **Quality Control**:
    - Multiple annotators for inter-annotator agreement
    - Clear rubrics with good/bad examples
    - Regular calibration sessions

---

### **Production Architecture I Implement**

**1. Multi-Method Approach**

- Combine all three tiers based on risk and volume
- Cheap methods broadly, expensive methods selectively
- Continuously validate AI judges against human baselines

**2. Slice-Based Evaluation (Critical for Debugging)**

- Never rely on aggregate metrics alone
- Slice by: query length, topic, user segment, difficulty
- Avoids Simpson's Paradox where Model A beats B overall but loses on every subset
- Helps identify biases and improvement opportunities

**3. Task-Specific Metrics**

For **RAG Systems**:
- Context precision: % retrieved chunks actually relevant
- Context recall: % of relevant information retrieved
- Answer relevancy: Addresses the query?
- Answer faithfulness: Grounded in context?

For **Code Generation**:
- Pass@k: % where at least 1 of k generations passes tests
- Compilation rate
- Execution success rate

For **Classification**:

- Precision, Recall, F1-score
- Per-class performance analysis

**4. Continuous Monitoring Pipeline**

- Daily evaluation of random production samples
- Track metrics over time for degradation detection
- A/B testing framework for model/prompt changes
- Anomaly detection in usage patterns

**5. User Feedback Loop**

- **Implicit signals**: Time on page, conversation continuation, task completion
- **Explicit signals**: Thumbs up/down, ratings
- **Key practice**: Correlate user feedback with evaluation metrics to validate

---

### **Cost-Latency-Quality Tradeoff Management**

In production, I optimize using Pareto principles:

1. Filter by non-negotiable constraints first (e.g., latency < 2s)
2. Then optimize quality within that space
3. Use tiered evaluation: cheap on 100%, expensive on 1%

---

### **Application-Specific vs Benchmarks**

I don't rely solely on public benchmarks (MMLU, HumanEval):

- They saturate quickly as models improve
- May not reflect actual use cases
- **Instead**: Curate custom test sets from real production usage
- This evolves with the product and reflects actual user needs

---

### **Key Takeaway**

The best production evaluation is **layered and continuous**:

- Automatic metrics for broad, cheap coverage
- AI judges for scalable quality assessment
- Human evaluation for ground truth and calibration
- All feeding back to improve the system

**I invest in systematic evaluation infrastructure from day one** - it's not optional for production LLM systems.