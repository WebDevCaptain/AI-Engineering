
### Self-Supervision: The Scaling Breakthrough

- **Self-supervision is THE breakthrough** that enabled scaling. Before this, AI models needed humans to manually label data, which was expensive and slow. For example, labeling 1 million images cost $50,000+. To label complex things like medical scans would be astronomical.

- **How self-supervision actually works:** The data contains both the question AND the answer. Take the sentence "I love street food." - a language model learns by predicting each next word using the previous ones. So "<BOS/> I" predicts "love", then "<BOS/> I love" predicts "street", etc. One sentence automatically generates 6 training examples with no human labeling needed.
  
- **Why this matters:** Text is everywhere (books, Reddit, articles). Self-supervision let models train on massive amounts of this naturally available text, enabling the scale-up to LLMs.

---

### Foundation Models: Multimodal and General-Purpose

- **Multimodal breakthrough:** Foundation models can work with text, images, audio together - not just one type of data. Traditional AI research was siloed: NLP did only text, computer vision did only images, audio models did only speech.
  
- **Why "foundation"?** Two meanings: (1) these models are foundational to AI applications, and (2) they can be built upon for different needs. They're like a foundation you build houses on, not a finished house.
  
- **Task-specific → General-purpose shift:** Old way = build separate models for sentiment analysis, translation, spam detection. New way = one foundation model can do all of these out-of-the-box. A model trained for sentiment analysis couldn't do translation before. Now an LLM does both.

---

### Why AI Engineering Emerged as a Discipline

**Three factors created AI engineering as a discipline:**

1. **General-purpose capabilities:** Models can do tasks thought impossible before. Since AI can write like humans, it can automate anything requiring communication (basically everything).
   
2. **Readily available models:** You don't need to train models from scratch anymore. This makes AI accessible to way more people.
   
3. **Cost-effective scaling:** Foundation models made it economically viable to build AI applications because you're leveraging existing models instead of expensive custom training.

---
### Application Patterns and Enterprise Realities

- **8 proven categories exist:** Coding, image/video production, writing, education, conversational bots, information aggregation, data organization, workflow automation.
  
- **Enterprise reality:** Companies deploy internal-facing apps first (like knowledge management) before external-facing ones (customer support bots) because internal apps have lower risk - if it messes up, customers don't see it.
  
- **Application realities are messy:** Many apps deployed but companies often don't know if they're actually working. Evaluation is the biggest bottleneck to AI adoption.

---

### The Last Mile Challenge: Demo vs. Product

- **Critical insight: "It's easy to build a cool demo with foundation models. It's hard to create a profitable product."**
  
- **The "last mile challenge":** Foundation models' impressive base capabilities are misleading. What actually happens:
    - 0% → 60% performance = easy (maybe a weekend)
    - 60% → 100% = exceedingly difficult (months or years)
    - Example: LinkedIn hit 80% desired experience in 1 month, then needed 4 MORE months to reach 95%. Each 1% gain gets slower and more discouraging.


- **Three risk levels to evaluate:**
    1. Existential threat (competitors could make you obsolete) → must do AI in-house
       
    2. Profit/productivity opportunities (most companies) → consider buy vs build
       
    3. Strategic positioning (don't want to be left behind) → R&D exploration

---

### ML Engineering Principles That Endure

- **What stayed the same:**
    - Still need systematic experimentation
    - Still need to make models faster and cheaper
    - Still need feedback loops to improve with production data
    - Still need to solve actual business problems
    - Still need to map business metrics to ML metrics

- **The foundation hasn't changed, but new stuff is layered on top**

---

### The AI Engineering Stack: What Changed

**The workflow completely flipped:**

- **Traditional ML:** Data → Model → Product (product comes last)
  
- **AI Engineering:** Product → Data → Model (product comes first)
  
- **Why this matters:** You can now rapidly build product demos with existing models, get user feedback, iterate fast, and only invest in custom data/models if the product shows promise.


**Three major capability shifts:**

1. **AI Interface became important**
    - Traditional ML: interfaces less important
    - AI Engineering: critical for user interaction
    - New interfaces: chat, voice, browser extensions, plug-ins, AR/VR
2. **Prompt Engineering is now a thing**
    - Traditional ML: doesn't exist
    - AI Engineering: essential skill
    - You adapt models by giving them instructions and context, not by changing the model itself
3. **Evaluation became MORE important**
    - Traditional ML: important
    - AI Engineering: even more critical
    - Why: Foundation models are open-ended (infinite possible responses), not close-ended like old classification tasks where you could just check if output matches expected output


**Model development layer changes:**

- **ML knowledge went from required → nice-to-have** (many successful AI builders don't care about gradient descent)
  
- **Inference optimization became even more critical** because autoregressive models generate tokens sequentially (10ms per token = 1 second for 100 tokens, but users expect 100ms responses)
  
- **Dataset engineering shifted focus:** less about feature engineering, more about data quality, deduplication, tokenization, context retrieval

---

> Tip - Framework Over Hype: Navigating AI's Fast Pace


