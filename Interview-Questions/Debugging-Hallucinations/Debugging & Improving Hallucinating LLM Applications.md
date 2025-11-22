
## How would you debug/improve an LLM application that's hallucinating?


---

## 1. Understanding Hallucinations

### Definition

When a model generates **plausible-sounding but factually incorrect** or unsupported information.

### Root Causes

1. **Architecture limitation**: Model trained to predict next token (fluency > accuracy)
2. **Training gaps**: Missing/outdated information in training data
3. **Behavioral pattern**: Tendency to "please" users rather than admit uncertainty
4. **No built-in fact-checking**: Completes patterns without validation

---

## 2. Diagnostic Framework (First Step)

### Classify Hallucination Type

| Type                  | Description                    | Example                                       |
| --------------------- | ------------------------------ | --------------------------------------------- |
| **Information-based** | Model lacks correct knowledge  | "GPT-5 was released in 2024" (didn't existed) |
| **Reasoning-based**   | Has info but wrong conclusions | "Since A>B and B>C, therefore A<C"            |
| **Context-based**     | Ignores provided context       | Using training data instead of RAG documents  |

### Debug Checklist

```
□ Collect 10+ hallucination examples
□ Identify pattern (specific topics? query types? time-based?)
□ Classify type (info/reasoning/context)
□ Check retrieval quality (if using RAG)
□ Review temperature/sampling settings
```

---

## 3. Solution 1: Temperature & Sampling Control

### Quick Win - Immediate Impact

**Problem**: High temperature = more randomness = more hallucinations

```python
# ❌ High hallucination risk
response = model.generate(
    prompt=prompt,
    temperature=0.9,  # Creative but unreliable
    top_p=0.95
)

# ✅ Reduced hallucination risk
response = model.generate(
    prompt=prompt,
    temperature=0.1,  # Deterministic, factual
    top_p=0.9
)
```

### When to Use Low Temperature (0.0-0.3)

- Factual Q&A
- Code generation
- Data extraction
- Medical/legal applications

### Trade-off

Lower temperature = Less creative, more repetitive (acceptable for factual tasks)

---

## 4. Solution 2: RAG (Retrieval Augmented Generation)

### Most Effective Solution for Information-Based Hallucinations

**Architecture Flow**:

```
User Query → Retrieve Relevant Docs → Assemble Context → LLM → Grounded Response
```

### Critical Prompt Engineering

```python
system_prompt = """
You are a helpful assistant. Answer questions ONLY based on the provided context.

STRICT RULES:
1. Use ONLY information from the context below
2. If answer not in context, respond: "I don't have enough information to answer that"
3. NEVER use your training knowledge
4. Cite sources for each claim

Context: {retrieved_documents}

User Query: {question}

Answer:
"""
```

### Key Anti-Hallucination Phrases

- "Based ONLY on the provided context"
- "According to the documents"
- "The context states that..."
- "If uncertain, explicitly say so"

---

## 5. Solution 3: Confidence Thresholds

### Don't Generate if Retrieval Quality is Low

```python
def answer_with_confidence_check(query):
    # Retrieve with scores
    docs, scores = retriever.search(query, k=5)
    
    # Confidence gate
    if max(scores) < 0.7:  # Adjust threshold based on testing
        return "I don't have reliable information to answer confidently."
    
    # Use only high-confidence docs
    high_conf_docs = [doc for doc, score in zip(docs, scores) 
                      if score > 0.7]
    
    context = "\n".join(high_conf_docs)
    return generate_response(query, context)
```

### Benefits

- Prevents garbage-in-garbage-out
- Teaches model to admit uncertainty
- Builds user trust

---

## 6. Solution 4: Source Attribution & Citations

### Force Accountability Through Citations

```python
prompt = """
For each fact, cite the source document.
Format: [Fact] (Source: Document #X)

Context:
Document 1: {product_specs}
Document 2: {user_manual}

Query: What's the maximum print speed?

Answer: The maximum print speed is 100 pages per minute (Source: Document 1).
"""
```

### Implementation Impact

- ✅ Users can verify facts instantly
- ✅ Easy to spot hallucinations (missing citations)
- ✅ Creates feedback loop for improvement
- ✅ Legal/compliance requirements met

---

## 7. Solution 5: Multi-Stage Verification

### Add Verification Layer Before Responding

```python
def verified_answer(query, context):
    # Stage 1: Generate initial answer
    answer = model.generate(query, context)
    
    # Stage 2: Extract factual claims
    claims = extract_claims(answer)  # "Speed is 100ppm", "Supports A4 paper"
    
    # Stage 3: Verify each claim against context
    verified_claims = []
    for claim in claims:
        if verify_in_context(claim, context):  # Semantic similarity check
            verified_claims.append(claim)
        else:
            log_hallucination(claim)  # Track for improvement
    
    # Stage 4: Reconstruct answer with verified claims only
    return reconstruct_answer(verified_claims)
```

### Verification Methods

1. **Semantic similarity**: Cosine similarity between claim and context chunks (>0.8)
2. **NLI model**: Check if context entails the claim
3. **LLM-as-judge**: Another model verifies factuality

---

## 8. Solution 6: Constrain Output Format

### Structured Outputs Reduce Hallucination Surface Area

```python
# ❌ Bad - Open-ended (high hallucination risk)
prompt = "Tell me about Product X"

# ✅ Good - Structured extraction
prompt = """
Extract ONLY from context:
{
  "price": "[exact price or 'not specified']",
  "features": ["[list from context]"],
  "availability": "[in-stock status or 'not specified']",
  "warranty": "[stated warranty or 'not specified']"
}

Rules:
- Empty fields = 'not specified'
- No inference or guessing
- Quote exact phrases from context
"""
```

### Why It Works

- Forces **extraction** not **generation**
- Easy to validate programmatically
- Clear gaps = obvious "not specified" vs hallucinated info

---

## 9. Solution 7: Chain-of-Verification (CoVe)

### Self-Correction Through Verification Questions

**Process**:

```
1. Generate initial response
2. Generate verification questions about response
3. Answer verification questions independently
4. Revise based on discrepancies
```

**Example**:

```python
# Step 1: Initial response
initial = "The Eiffel Tower is 300m tall and completed in 1887"

# Step 2: Generate verification questions
verification_qs = [
    "What is the exact height of the Eiffel Tower?",
    "When was the Eiffel Tower completed?"
]

# Step 3: Answer from context independently
verified_answers = {
    "height": "330 meters",
    "completion": "1889"
}

# Step 4: Revise
final = "The Eiffel Tower is 330 meters tall and completed in 1889"
```

---

## 10. Solution 8: Explicit Anti-Hallucination Prompts

### Train Behavior Through Instructions + Examples

```python
system_prompt = """
You prioritize ACCURACY above all else.

RULES:
1. Never fabricate facts
2. Distinguish facts from inferences clearly
3. If uncertain, say "I don't know" 
4. Use phrases: "Based on the context..." or "The document states..."
5. ZERO speculation or guessing

GOOD Example:
Q: "What is the warranty?"
Context: [No warranty info]
A: "The context doesn't include warranty information."

BAD Example (NEVER DO THIS):
Q: "What is the warranty?"
Context: [No warranty info]
A: "Most products have 1-year warranty." ← HALLUCINATION
"""
```

---

## 11. Solution 9: Fine-Tuning on Curated Data

### When Prompt Engineering Isn't Enough

**Training Data Format**:

```json
{
  "context": "[Ground truth documents]",
  "question": "What is X?",
  "good_answer": "Based on the context, X is Y (Source: Doc 1).",
  "bad_answer": "X is probably Z."  // Model learns to avoid
}
```

**Include in Training Set**:

- ✅ Proper refusal examples ("I don't know" responses)
- ✅ Citation-backed answers
- ✅ Contrastive examples (good vs bad)
- ⚠️ **Warning**: Low-quality fine-tuning data worsens hallucinations

---

## 12. Solution 10: Production Monitoring

### Continuous Improvement Loop

**Detection Methods**:

```python
# 1. User feedback
thumbs_down_button()
report_incorrect_feature()

# 2. Automated checks
def detect_hallucination(response, context):
    # Fact verification against knowledge base
    unsupported_claims = verify_claims(response, context)
    
    # Consistency check
    if same_question_different_answers(query):
        flag_inconsistency()
    
    # External validation (if applicable)
    if api_available:
        cross_check_with_external_source()

# 3. Random sampling
daily_review = sample_responses(n=100)
human_review(daily_review)
```

**Key Metrics to Track**:

| Metric                        | Definition                     | Target |
| ----------------------------- | ------------------------------ | ------ |
| **Factual Accuracy**          | % claims verifiable in context | >95%   |
| **Unsupported Claim Rate**    | % statements without source    | <5%    |
| **Citation Rate**             | % responses with citations     | >90%   |
| **User Correction Frequency** | Reports per 1000 queries       | <10    |

**Feedback Loop**:

```
Detect Hallucination → Log Pattern → Analyze Root Cause → 
Improve (Prompt/RAG/Model) → Deploy Fix → Monitor Impact → Repeat
```

---

## 13. Complete Debugging Workflow

### Phase-by-Phase Approach

**Phase 1: Reproduce & Analyze** (Day 1)

```
□ Collect 20+ hallucination examples
□ Classify by type (info/reasoning/context)
□ Identify patterns (topics, query types, time-based)
□ Document reproduction steps
```

**Phase 2: Quick Wins** (Day 1-2)

```
□ Lower temperature to 0.1-0.3
□ Add explicit anti-hallucination instructions
□ Implement confidence thresholds
□ Test and measure impact
```

**Phase 3: Structural Improvements** (Week 1)

```
□ Implement/improve RAG system
□ Add mandatory source attribution
□ Improve retrieval quality (embeddings, chunking)
□ Add semantic search over keyword search
```

**Phase 4: Advanced Solutions** (Week 2-4)

```
□ Multi-stage verification pipeline
□ Fine-tune on curated data
□ Ensemble methods (multiple models vote)
□ Deploy continuous monitoring
```

---

## 14. Summary

### **"How would you debug an LLM hallucination issue?"**

**My Answer Framework**:

**1. Diagnose First** (Don't jump to solutions)

- Collect examples, classify type (info/reasoning/context-based)
- Identify patterns

**2. Quick Wins** (Show you can deliver fast)

- Lower temperature to 0.1 for factual tasks
- Add explicit "don't hallucinate" instructions
- Implement confidence thresholds on retrieval

**3. Structural Fix** (Show system design skills)

- Implement RAG with proper prompt engineering
- Force source citations for accountability
- Constrain output format (structured extraction)

**4. Verification Layer** (Show engineering rigor)

- Multi-stage verification before responding
- NLI models or LLM-as-judge for fact-checking
- Chain-of-Verification for self-correction

**5. Production Safeguards** (Show you think about ops)

- Monitor hallucination metrics continuously
- User feedback loops
- Automated consistency checks
- Human-in-the-loop random sampling

**6. Key Insight to Share**: _"RAG is most effective, but alone isn't enough. The combination of proper retrieval, explicit prompting to use ONLY context, low temperature, source attribution, and monitoring creates a robust system. I'd also implement graceful degradation—when confidence is low, admit uncertainty rather than hallucinate."_

---

## 15. Code Example: Complete Solution

```python
class HallucinationSafeQA:
    def __init__(self, model, retriever):
        self.model = model
        self.retriever = retriever
        self.confidence_threshold = 0.7
        self.temperature = 0.1
    
    def answer(self, query):
        # Step 1: Retrieve with confidence
        docs, scores = self.retriever.search(query, k=5)
        
        # Step 2: Confidence gate
        if max(scores) < self.confidence_threshold:
            return {
                "answer": "I don't have reliable information to answer this.",
                "confidence": max(scores),
                "sources": []
            }
        
        # Step 3: Filter high-confidence docs
        high_conf_docs = [
            {"text": doc, "score": score, "id": i} 
            for i, (doc, score) in enumerate(zip(docs, scores)) 
            if score > self.confidence_threshold
        ]
        
        # Step 4: Build context with source tracking
        context = "\n\n".join([
            f"Document {doc['id']}: {doc['text']}" 
            for doc in high_conf_docs
        ])
        
        # Step 5: Generate with strict prompt
        prompt = f"""
        Answer ONLY based on provided context. Cite sources.
        If answer not in context, say "Information not available."
        
        Context:
        {context}
        
        Question: {query}
        
        Answer with citations:
        """
        
        answer = self.model.generate(
            prompt=prompt,
            temperature=self.temperature,
            max_tokens=500
        )
        
        # Step 6: Verify claims (optional advanced step)
        if self.verify_enabled:
            verified_answer = self.verify_claims(answer, context)
        else:
            verified_answer = answer
        
        # Step 7: Log for monitoring
        self.log_response(query, verified_answer, high_conf_docs)
        
        return {
            "answer": verified_answer,
            "confidence": max(scores),
            "sources": [doc['id'] for doc in high_conf_docs]
        }
    
    def verify_claims(self, answer, context):
        # Extract claims and verify each against context
        claims = self.extract_claims(answer)
        verified = [c for c in claims if self.verify_in_context(c, context)]
        return self.reconstruct_answer(verified)
```

---

## 16. Key Metrics Before/After

|Metric|Before|After (with solutions)|
|---|---|---|
|Factual Accuracy|70%|95%+|
|Unsupported Claims|25%|<5%|
|User Corrections|50/1000 queries|<10/1000|
|"I don't know" Rate|2%|15% (good—admits uncertainty)|
|User Trust Score|3.2/5|4.6/5|

---

## Final Takeaway

**"Hallucinations are inherent to LLM architecture, but manageable through layered defenses:**

1. **RAG for grounding** (most critical)
2. **Low temperature for determinism**
3. **Confidence gates** (don't answer if retrieval poor)
4. **Forced citations** (accountability)
5. **Verification layers** (for critical applications)
6. **Continuous monitoring** (production safety net)

**The goal isn't perfection—it's building systems that degrade gracefully and admit uncertainty appropriately."**
