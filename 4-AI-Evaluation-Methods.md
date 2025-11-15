
### **The Evaluation Pipeline Crisis**

**Foundation:** Evaluation isn't just a nice-to-have metric - it's the difference between shipping something useful versus shipping something that might actively harm your users or business. 

> Without evaluation, you can't answer "Is this working?" or "Did this change make things better or worse?" 

Many teams deploy AI applications and have no idea if they're helping or hurting user experience. 

> For eg, The used car dealership AI: they deployed a car value prediction model, users seemed to like it, but a year later they still had no idea if the predictions were accurate.

---

### **From Model Building to Model Selection**

**Foundation:** We're in an era where hundreds of capable foundation models exist (GPT-5, Claude, Llama, Mistral, etc.). You don't need to train models from scratch. The real skill is knowing which model fits YOUR specific use case. 

> A model that's great for creative writing might be terrible for medical diagnosis.

This shifts the engineer's role from "how do I train a model" to "how do I evaluate and choose among existing models."

---

### **Four Evaluation Criteria: Domain, Generation, Instructions, and Constraints**

There are four buckets of criteria you need to think about:

1. **Domain-specific capability** - Can it actually do the task? (e.g., write legal contracts, generate code, translate Spanish)
   
2. **Generation capability** - How good is the output quality? (Is it fluent? Coherent? Faithful to the source?)
   
3. **Instruction-following** - Does it follow format requirements? (correct length, structure, tone)
   
4. **Practical constraints** - Cost per request, latency, scalability


The key insight: Different criteria need different evaluation methods. You can't evaluate code generation the same way you evaluate essay writing.

---

### **Evaluating Capabilities: Domain Skills, Factual Consistency, and Safety**


**Domain-specific**: If you need a model to write Python code, you test it with code benchmarks using **functional correctness** - does the code actually run and produce the right output? But there's more: is the code efficient? Readable? Maintainable?

**Generation capabilities**: These come from traditional NLP - does the text sound natural (fluency)? Does it flow logically (coherence)? For translation, is it faithful to the original meaning?

**Factual consistency**: This is critical and tricky. The model might generate fluent, coherent text that's completely wrong. Example: GPT-3 confidently stating "All artificial intelligences currently follow the Three Laws of Robotics" (from science fiction, not reality). You evaluate this using **textual entailment** - can the generated statement be inferred from the source material?

**Safety**: Six categories of harmful content - inappropriate language, harmful recommendations, hate speech, violence, stereotypes, and political/religious bias. Different models have different political leanings (GPT-4 tends left-libertarian, Llama more authoritarian).

---

### **Traditional NLP Metrics: Fluency, Coherence, and Faithfulness**

**Foundation:** We're not inventing evaluation from scratch. Decades of NLP research gave us methods to measure:

- **Fluency**: Does this sound like a native speaker wrote it?
- **Coherence**: Does the whole piece have logical structure and flow?
- **Faithfulness**: For tasks like summarization/translation, does the output preserve the meaning of the original?

These traditional metrics still apply, but foundation models present NEW challenges because they're open-ended and can do so many different tasks.

---

### **Build vs. Buy: The Seven-Axis Decision Framework**

> The seven axes are:

1. **Data privacy**: APIs mean sending your data externally; self-hosting keeps it internal
   
2. **Data lineage**: Who owns the training data? Copyright issues?
   
3. **Performance**: Best models are often closed-source APIs
   
4. **Functionality**: APIs have better tooling; self-hosting gives you access to internal model states (logprobs) which are crucial for debugging
   
5. **Cost**: APIs charge per token; self-hosting requires engineering talent and time. Key insight: cost per token decreases dramatically at scale for self-hosted
   
6. **Control**: APIs subject you to rate limits, terms changes; self-hosting gives full control but full responsibility
   
7. **Edge deployment**: APIs need internet; some use cases require on-device models

The wisdom: **There's no universal answer**. It depends on your scale, privacy requirements, and engineering capacity. Many teams start with APIs (fast to prototype) and switch to self-hosting at scale.

---

### **Public Benchmarks as Filters, Not Solutions**

Think of public benchmarks (like MMLU, HumanEval, GSM-8K) as a **filter, not a solution**. If a model scores 30% on a math benchmark, it's probably bad at math. But if two models both score 90%, which is better for YOUR specific math tutoring app? The benchmark can't tell you.

Why? 
Because public benchmarks:-
- Test general capabilities, not your specific use case
- Use standardized data that may not reflect your users
- Can't capture all the nuances that matter to you (tone, style, domain expertise)

---

### **The Data Contamination Problem**

**Data contamination** is a massive problem. Models are trained on internet data. Benchmarks are published online. So models often see the test questions during training and just memorize answers.

_A satirical example_: A researcher deliberately trained a tiny 1M-parameter model ONLY on benchmark data. It achieved near-perfect scores and beat much larger models - but it was completely useless for anything else. It just memorized the answers.

This means: **High benchmark scores don't guarantee real-world usefulness**. A model might score 95% on legal bar exam questions not because it understands law, but because it memorized those specific questions.

---

### **Creating Your Private Leaderboard**

You need to think like a leaderboard creator:

1. **Choose relevant benchmarks** - If you're building a coding assistant, use code benchmarks, not poetry benchmarks
   
2. **Check correlation** - If two benchmarks are perfectly correlated (always rank models the same way), you only need one
   
3. **Decide how to aggregate** - Should all benchmarks count equally? Or should coding ability matter more than math for your coding assistant?
   
4. **Create your own evaluation data** - Use actual examples from your application domain

This is essentially creating a **private leaderboard** customized to your application.

---

### **Combining Multiple Evaluation Methods**

Foundation models are incredibly complex - they can write, reason, code, analyze, create. How do you reduce that to a single number?

You can't. It's like trying to judge a person's entire capability with their SAT score. It tells you something, but not everything.

**The solution**: Use **multiple evaluation methods** together:

- Exact match for simple tasks
- Functional correctness for code
- Semantic similarity for meaning
- AI judges for subjective quality
- Human evaluation for edge cases

Each method has biases and limitations. But **combining them** gives you a more complete picture. It's like using multiple diagnostic tests in medicine - each one gives you a different signal.

---

### **Evaluation as Continuous, Not One-Time**

Evaluation isn't a one-time activity - it's continuous:

- **During selection**: Which model should I use?
- **During development**: Did this prompt change improve things?
- **During deployment**: Is the model performing as expected?
- **In production**: Are users satisfied? Where are the failures?

The chapter provides a **4-step iterative workflow**:

1. Filter by hard constraints (privacy, licensing, budget)
2. Narrow using public benchmarks
3. Run your private evaluation pipeline
4. Monitor in production and iterate

This is evaluation-driven development - make evaluation criteria clear BEFORE you start building, not after.