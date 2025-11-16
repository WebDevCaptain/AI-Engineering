
### Behaviour-Driven Dataset Design

- Models learn by example, not by explanation. If you want your chatbot to give concise answers, your training data must contain concise answers. If you want it to think step-by-step (chain-of-thought), your data must show step-by-step reasoning.
  
- **For example**: A ChatGPT team noticed the bot was arrogant and gave unsolicited rewrites. Investigation revealed training data contained examples with unsolicited suggestions. 
  Solution: Remove those examples and add ones showing fact-checking WITHOUT unsolicited rewrites.
  
- **Why this matters**: You can't tell a model "be polite" in training—you must SHOW it what politeness looks like through consistent examples.
---

### Evolution of Data Roles

- Data work has evolved from being a side task to a full-time specialized role because of its complexity and importance.
  
- **Scale of change**: GPT-3 (2020) had 2 people working on data. GPT-4 (2023) had 80 people, plus external annotators. Even something as "simple" as ChatML format required 11 senior researchers.
  
- **Why dedicated roles**: Data compliance alone can be a full-time job. Dataset creators need to understand how models learn, what resources are available, and work closely with application developers.

---

### Training Phase Data Requirements

- Different training stages need fundamentally different data types:
    - **Pre-training**: Raw sequences of data (measured in tokens—trillions of them)
      
    - **Instruction finetuning**: (instruction, response) format pairs
      
    - **Preference finetuning**: (instruction, winning response, losing response) triplets
      
    - **Reward model**: ((instruction, response), score) format


- **Why this matters**: You can't use pre-training data for instruction finetuning or vice versa. The format must match what you're teaching the model to do.

---

### The Three Core Criteria: Quality, Coverage, Quantity

**Key Learning - The Cooking Analogy:**

- **Quality** = Quality of ingredients. Spoiled ingredients → bad food. Bad data → bad model.
  
- **Coverage** = Right mix of ingredients. Too much/little sugar ruins the dish. Wrong data mix ruins the model.
  
- **Quantity** = How many ingredients you need. Not enough → incomplete dish. Too much → waste.

**Critical insight**: These three must work together. You can't compensate for poor quality with more quantity.

---

### Quality Over Quantity Principle

- **Concrete proof**: A 1.3B parameter model trained on 7B tokens of high-quality code data outperformed much larger models on coding benchmarks.
  
- **Experiment: 7B model finetuned on 2,000 high-quality + diverse examples significantly outperformed the same model trained on 2,000 low-quality OR 2,000 non-diverse examples.
  
- **Why**: Noisy data teaches the model bad patterns. The model learns both the correct and incorrect behaviours, creating confusion.

---

### Data Diversity and Coverage

- **Coverage = Diversity**: Your data must cover the range of real-world problems users will have.
  
- **Practical examples**:
    - If some users write detailed instructions and others write short ones → include BOTH in training
    - If users make typos → include typos in training data
    - If app supports multiple programming languages → training must include all those languages

- **Meta's Llama 3 finding**: Performance gains came primarily from improvements in data quality and diversity, NOT architecture changes.

---

### Synthetic Data Generation

**Key Learning:**
- **Evolution**: Used to generate only simple things (names, addresses). Now generates doctor's notes, contracts, financial statements, images.
  
- **Why synthetic data helps**:
    - **Scale**: Generate data where real-world data is scarce (rare weather, accidents for self-driving cars)
    - **Targeted coverage**: Generate specific types of data to fill gaps (very short texts, very long texts, toxic content for detection models)
    - **Quality**: Sometimes AI generates better data than humans (e.g., complex math problems, tool usage data where AI understands AI behavior better)
    - **Privacy**: Generate data without exposing sensitive information


**Important**: Synthetic data supplements but doesn't completely replace human data. Best results come from mixing both.

---

### Evaluating Synthetic Data

**Key Learning:**
- **The challenge**: Evaluating AI-generated data is as tricky as evaluating other AI outputs.
  
- **Why people hesitate**: They'll only use synthetic data if they can reliably verify its quality. "Garbage in, garbage out" applies doubly to synthetic data.
  
- **Ultimate test**: Does it improve the model's real-world performance? That's the only measure that truly matters.
  
- **Risk of model collapse**: Training only on AI-generated data can cause models to progressively lose performance over generations. 
  
**Solution**: Always mix synthetic with real data.
  
---

### Limitations of Automation

**Key Learning - The Hierarchy of Difficulty:**

1. **Hard**: Annotating data (labeling examples)
2. **HARDER**: Creating annotation guidelines (defining what "good" looks like)
3. **Hard**: Automating data generation (getting AI to make data)
4. **HARDER**: Automating verification (checking if generated data is correct)


**What can't be automated:**

- Thinking through what data you actually want
- Creating clear annotation guidelines (LinkedIn reported this as their most challenging pipeline part)
- Paying attention to details
- Making strategic decisions about data mix


**Real consequence**: Teams often abandon careful annotation halfway, hoping models will "figure it out." This is risky for production applications.

---

### Annotation Guideline Complexity

**Key Learning - Why Guidelines Are So Hard:**

- Must explicitly define abstract concepts:
    - What does a "good" response look like?
    - Can a response be correct but unhelpful?
    - What's the difference between a score of 3 vs 4?
    - How do you score creativity? Factual accuracy? Conciseness?

**The annotation guideline paradox**: Guidelines are needed for both humans AND AI annotations, but creating them requires the exact clarity that you're trying to teach the model.

**Good news**: Annotation guidelines are the same as evaluation guidelines ( as we have learned in [4-AI-Evaluation-Methods](4-AI-Evaluation-Methods.md) ). Do this work once, use it twice.

---
### Creativity in Dataset Engineering

**Key Learning:**

- **No single "correct" approach**: Different teams use wildly different strategies to build datasets, and many work well.
  
- **Examples of creativity**:
    - Using dimensionality reduction for deduplication
    - Synthesizing data by creating templates then generating variations
    - Using AI to bootstrap better AI through carefully designed workflows
    - Creating adversarial examples to test specific model behaviours
    - Mixing multiple data sources through iterative refinement


**Why creativity matters**: The data landscape is more complex than the model landscape. You need creative solutions for:

- Acquiring data within budget constraints
- Balancing quality vs. quantity tradeoffs
- Designing data to teach specific complex behaviors (tool use, chain-of-thought)
- Verifying data quality when ground truth doesn't exist

**Key insight**: Dataset engineering is part science, part art. The "art" part—thinking creatively about what data to collect and how—can't be automated.

---

### Manual Inspection and Data Mistakes

**Manual inspection is irreplaceable:**

- Greg Brockman (OpenAI co-founder) quote: "Manual inspection of data has probably the highest value-to-prestige ratio of any activity in machine learning."
- Just 15 minutes of staring at data can reveal insights that save hours of debugging.
- Patterns emerge that you can turn into heuristics (e.g., annotations made in the second half of a session are lower quality due to fatigue).

**Data mistakes have cascading effects:**

- A simple formatting error (missing arrow "->", extra space) can cause perplexing model behaviors
- A float stored as integer can completely change model learning
- Duplicated data teaches wrong patterns (e.g., "all red items are expensive")
- Wrong chat template causes silent failures—model works but not as intended

**The data flywheel advantage:**

- Your application's user data is your most valuable source—it matches exactly what you care about
- If you can create a system where user interactions improve your data, which improves your product, which attracts more users, you gain a significant competitive advantage