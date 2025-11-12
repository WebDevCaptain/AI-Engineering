
### Training Data: The Model's Entire World

- **Models learn patterns from data, not "knowledge" in the human sense** - If Vietnamese isn't in the training data, the model literally cannot understand or generate Vietnamese, no matter how large it is
  
- **Training data = the model's entire world** - A model has no concept of anything outside what it saw during training
  
- **Garbage in, garbage out** - Common Crawl (the most used dataset) contains clickbait, misinformation, and low-quality content, which directly affects what the model learns to generate


**Why it matters:** You can't fix a data problem with a bigger model. GPT-4 performs worse on math problems in Bengali than English simply because there's less Bengali math content in training data.

---

### Why Generic Models Fail: Language & Domain Gaps


- **Language representation is unequal** - High-resource languages (English) dominate training data, making models naturally better at English
  
- **Domain-specific knowledge requires domain-specific data** - Generic models trained on internet text won't understand medical imaging or protein structures (that's why AlphaFold needed specific protein sequence data)
  
- **Publicly available ≠ what you need** - Most public datasets are English text; specialized domains require curated proprietary data


**Why it matters:** Understanding this explains why Claude/ChatGPT can write code but struggles with specialized medical diagnosis - the internet has lots of code tutorials but limited specialized medical literature.

---

### Transformer's Breakthrough: Attention & Parallel Processing


- **The transformer solved two critical problems from older architectures (RNNs):**
    1. **Sequential processing bottleneck** - Old models had to process "I love eating pizza" word by word, waiting for each to finish. Transformers process all input words in parallel
    2. **Context limitation** - Old models summarized the entire input into one final state (like reading a book then only keeping the last page). Transformers can reference ANY previous word through the attention mechanism


**The Attention Mechanism (the key innovation):**

- Think of it as a **search system within the model**
- When generating "The capital of France is ___", the model can "attend" to (look back at) the words "capital" and "France" to generate "Paris"
- **Query = what I'm looking for**, **Key = indexed items**, **Value = actual content**
- This is why transformers can handle long-range dependencies (referencing something mentioned 1000 words ago)


**Why it matters:** This explains why GPT can maintain context in long conversations - it's not "remembering" like humans do, it's computing attention scores to relevant past tokens every single time it generates.

---

### The Scaling Trinity: Parameters, Tokens, and Compute Cost

**Parameters (the model's "brain size"):**

- Each parameter is a single number that gets adjusted during training
- More parameters = more capacity to learn complex patterns
- Think of it like having more neurons in a brain
- Example: 7B parameters means 7 billion adjustable numbers
  

**Training Tokens (the model's "education"):**

- A token ≈ ¾ of a word (0.75x of a word)
- More training tokens = the model has "read" more text
- **Critical insight:** You can't just train longer on the same data - Llama 3 used 15 TRILLION tokens (that's 450 million books worth)
  

**FLOPs (the cost):**

- Floating Point Operations = actual computational work done
- Training GPT-3 required 3.14 × 10²³ operations
- With 256 high-end GPUs at 70% efficiency: ~$4M and 8 months
- **This is why only big companies train foundation models**


> Understanding this trinity explains why you can't just "train a better ChatGPT" - it's not just about having the code, it's about having hundreds of millions of dollars and months of GPU time.

---

### Chinchilla's Law: Optimal Model-Data Balance


**The Chinchilla Law (the key formula):**

- For optimal performance: **Training tokens should be ~20× the number of parameters**
- A 3B parameter model needs ~60B training tokens
- You should scale both equally: double parameters → double training data

**The practical implication:**

- Earlier models (GPT-3) were over-parameterized and under-trained
- They used too many parameters for the amount of data
- Chinchilla showed you can get better performance with a smaller model trained on more data

**The real-world trade-off:**

- Chinchilla-optimal = best performance during training
- But smaller models are cheaper and faster to run in production
- **This is why Llama chose "suboptimal" smaller models** - they're easier to deploy and cheaper to serve to users

> This explains the industry shift: It's not just about making bigger models anymore, it's about finding the right balance between model size, training data, and real-world usability.

---

### The Pre-training Problem: From Internet Chaos to Rogue Model

**What is self-supervision:**

- The model learns by predicting the next word: "The cat sat on the ___" → predict "mat"
- No human tells it what's right or wrong, it just learns patterns
- **Problem:** It learns to complete text like a webpage, not have a conversation

**Why raw pre-trained models are "rogue":**

- If you ask "How to make pizza?", it might continue with "for 50 people? on a budget? with vegan cheese?" instead of answering
- It absorbed internet toxicity - racism, misinformation, harmful content
- It has no concept of being "helpful" or "safe"

**The fundamental mismatch:**

- Pre-training optimizes: "predict next token accurately"
- Users want: "give me helpful, safe, accurate responses"
- **These are fundamentally different objectives**

> This is why you can't just use GPT-3's raw pre-trained model - it would be like releasing an untrained, unsocialised AI that mimics the worst of the internet. Post-training is not optional, it's essential.

---

### Post-training: Teaching Conversation (SFT) and Values (RLHF)

**Supervised Finetuning (SFT) - Teaching conversation:**

- Show the model examples: "How to make pizza?" → (proper helpful answer)
- This is behaviour cloning - demonstrating how to behave
- **Critical insight:** You need high-quality human labelers who can think critically, not just crowd workers
- Cost: $25 per response example for quality labeling


**Preference Finetuning (RLHF/Direct Preference Optimization) - Teaching values:**

- Train a "reward model" that scores responses: Response A vs Response B - which is better?
- Then train the main model to generate responses that get high scores
- **The impossible challenge:** Human preferences are diverse and contradictory - how should AI respond to questions about abortion, gun control, etc.?


**The key philosophical problem:**
- Assumes "universal human preference" exists (it doesn't)
- Assumes you can capture it in math (very difficult)
- **This is why AI alignment is an unsolved problem**


**The resource split:**
- Pre-training: 98% of compute (InstructGPT)
- Post-training: 2% of compute
- **Post-training "unlocks" capabilities, doesn't create them**


> This explains why ChatGPT sometimes refuses requests or gives different answers to similar questions - the post-training is trying to align with human preferences, but those preferences are inherently contradictory and context-dependent.

---

### Probabilistic Sampling: The Source of Creativity and Hallucinations

**How models actually generate text:**
1. For "My favorite color is ___", the model computes probabilities: red (30%), green (50%), blue (15%), purple (5%)
2. It doesn't pick the highest - it **samples** according to these probabilities
3. This means: same question → potentially different answers every time
4. **This is fundamentally different from traditional software**


**Temperature (the creativity knob):**

- Temperature = 0: Always picks highest probability (boring but consistent)
- Temperature = 0.7: Balanced creativity
- Temperature = 2: Very random and creative (potentially incoherent)
- **Technical:** Divides logits before computing probabilities, redistributing likelihood to rare words


**Why probabilistic matters:**

- **Creativity:** Enables novel combinations and ideas because low-probability paths are explored
- **Inconsistency:** Ask "what's 2+2?" twice → might get different levels of explanation detail
- **Hallucination:** If "US presidents are aliens" appears in training data with 0.1% probability, it can still be generated


**The two hallucination hypotheses:**

1. **Self-delusion:** Model can't distinguish its own generated text from facts
    - Generates "Shreyash is a software architect" (wrong)
    - Next sentence conditioned on this, builds more false facts
    - Snowballing effect
2. **Mismatched knowledge:** During training, human labelers write responses using knowledge the model doesn't have
    - Model learns to "make things up" to match human responses
    - Essentially trained to hallucinate


> This explains why you can't make AI "perfectly consistent" or "never hallucinate" - the probabilistic sampling is core to how neural networks generate text. You can only mitigate, not eliminate. This is why building reliable AI systems requires architectural choices (verification, multiple samples, constrained outputs) rather than expecting deterministic behavior.
