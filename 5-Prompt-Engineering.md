
### **Foundation Model Capabilities & Limitations**

- Foundation models are **pre-trained on massive amounts of data** and contain broad capabilities (writing, reasoning, translation, coding, etc.)

- However, they don't automatically know YOUR specific task or desired output format
  
- **Unlike traditional ML models** that are trained for one specific task, foundation models need runtime instructions to activate the right "program" or capability

- Think of it like having a Swiss Army knife - it has many tools, but you need to explicitly choose which one to use

---

### Prompt Engineering Fundamentals

- Prompt engineering is fundamentally different from traditional ML because it **doesn't change the model's weights**
  
- It's the **easiest and most common model adaptation technique** - you should exhaust ***prompting possibilities*** before moving to expensive techniques like *finetuning*
  
- The model already has the capability; 
	- prompting is about **activation**, not training


**Why this matters:**
- No computational cost for training
  
- Instant iteration (change prompt, test immediately)
  
- But also no guarantee - the model must already have learned the capability during pre-training

---

### Model Robustness & Sensitivity

**Critical Understanding:**
- Models have different levels of **robustness to prompt perturbation**
  
- Small changes (writing "5" vs "five", adding newline, capitalization) can dramatically change weaker models' outputs
  
- **Robustness correlates with overall model capability** - stronger models understand that "5" and "five" mean the same thing


**Practical Implication:**
- This is WHY working with stronger models often saves time - less fiddling needed
  
- Weaker models require more careful prompt engineering
  
- You can measure robustness by randomly perturbing prompts and observing output changes

---

### **Human-AI Communication Principles**

> Here is a profound analogy: **just like human communication, AI communication has quality levels**

- Anyone can communicate, but effectiveness varies widely
  
- Clear instructions + examples + relevant context = better communication (with both humans and AI)
  
- Models have "quirks and biases" just like humans - you need to understand these to work effectively with them


**Key prerequisite:** 
	The model must be able to **follow instructions** (an instruction-following capability). If the model is bad at this, no amount of prompt crafting will help.

---

### Prompt Anatomy: System vs User Prompts


**System Prompt vs User Prompt:**
- **System prompt** = task description (what role/behavior you want)
- **User prompt** = the actual task/query


> They're concatenated into a single prompt using a **chat template** before feeding into the model. From the model's perspective, they're processed identically.


**Why system prompts seem to work better:**

1. **Position effect**: System prompts come FIRST in the final prompt, and models are better at understanding information at the beginning
   
2. **Training bias**: Models may have been post-trained to prioritize system prompts (OpenAI's "Instruction Hierarchy" training)


**Why templates matter critically:**

- Different models use different templates (Llama 2 vs Llama 3 have different formats)
  
- Even the SAME model provider can change templates between versions
  
- Small mistakes (extra newline, wrong special token) cause **silent failures** - model still responds but performs poorly
  
- Template errors are VERY common in practice

---

### In-Context Learning Mechanics

**Why In-Context Learning Works - The Revolutionary Discovery:**

**Historical Context:**

- **Before GPT-3 (2020)**: ML models could ONLY do what they were explicitly trained to do
- **After GPT-3**: Models could learn NEW behaviours from examples in the prompt, WITHOUT weight updates
- This felt like "magic" at the time - many researchers tried to understand why


**The Fundamental Mechanism:**

- GPT-3 was trained only for **next token prediction**
- Yet it could do translation, math, reading comprehension - tasks it wasn't explicitly trained for
- François Chollet's analogy: Foundation model = "library of many different programs"
    - Each program (haiku writer, limerick writer, translator) can be activated by the right prompt
    - Prompt engineering = finding the activation key


**Practical Implication - Continual Learning:**

- Example: Model trained on old JavaScript docs
- Without in-context learning: Must retrain the entire model
- With in-context learning: Just include new JavaScript docs in the prompt
- This prevents models from becoming outdated

**Zero-Shot vs Few-Shot Evolution:**

- **Early models (GPT-3)**: Few-shot >>> Zero-shot (needed many examples)
- **Modern models (GPT-4)**: Few-shot ≈ Zero-shot (examples help less)
- **Why?** Stronger models are better at instruction-following
- **Exception**: Domain-specific tasks with rare syntax (like Ibis dataframe API) still benefit greatly from examples

---


### Chain-of-Thought & Context Provision

**Core Discovery:**
- Introduced in Wei et al., 2022 (BEFORE ChatGPT existed)
- One of the first prompting techniques that works across different models
- Simply adding "think step by step" dramatically improves performance

**Why It Works:**
- Nudges model toward **systematic problem-solving** instead of jumping to conclusions
- Forces model to show its reasoning process
- **LinkedIn finding**: CoT also REDUCES hallucinations

**Practical Variations:**
1. **Zero-shot CoT**: Add "think step by step" or "explain your decision"
2. **Specified steps**: Tell model exactly what steps to follow
3. **One-shot CoT**: Show an example of step-by-step reasoning

**The Relevance/Context Principle:**
- Just like humans do better on exams with reference materials, models do better with sufficient context
- Without context, models rely on internal knowledge (which may be unreliable) → leads to hallucinations
- Providing context mitigates hallucinations by grounding responses in actual information

---

### Model Quirks & Positional Biases

> Models are NOT Perfect Reasoners


**Context Position Bias ("Lost in the Middle"):**
- Models understand information at BEGINNING and END of prompts better than middle
- This is why you should place important information strategically
- "Needle in a Haystack" tests reveal which parts of context models actually attend to
  

**Model-Specific Quirks:**
- Some models (GPT-4) prefer task description at beginning
- Others (Llama 3) perform better with task description at end
- You must experiment to find what works for YOUR model


**Template Sensitivity:**
- Small changes dramatically affect behavior
- This is why stronger models are better - they're more robust to variations

---


### The Security Paradox of Instruction-Following


> The VERY ABILITY that makes foundation models useful (following instructions) is what makes them vulnerable. This is an inherent tension, not a bug to be fixed.


**Why This Happens:**

- Models cannot differentiate between system instructions and user instructions
- Both are concatenated into one prompt and processed identically
- A model trained to follow instructions will follow ALL instructions - including malicious ones
- **Critical finding**: As models get BETTER at following instructions, they also get better at following malicious instructions


**Types of Attacks:**
1. **Prompt Extraction**: Stealing proprietary system prompts
   
2. **Jailbreaking**: Subverting safety features (getting model to explain bomb-making)
   
3. **Prompt Injection**: Injecting malicious commands (e.g., "Delete the order entry from the database")


**Attack Evolution:**
- **Early attacks**: Simple obfuscation ("vacine" instead of "vaccine", roleplaying as "DAN")
- **Modern attacks**: Automated AI-powered attacks (PAIR - one AI attacks another, requiring <20 queries to jailbreak)
- **Indirect injection**: Placing malicious instructions in tools/websites the model can access (much more powerful)


**Defense Approach:** 3 layered:--

1. **Model-level**: Train models to prioritize system prompts (OpenAI's "Instruction Hierarchy" - 63% robustness improvement)
2. **Prompt-level**: Duplicate system prompts, explicit warnings
3. **System-level**: Isolation, human approval for dangerous commands

---

### Adversarial Attacks & Defense Strategies

**Fundamental Reality:**

- As long as models CAN do impactful things, they CAN be exploited
- New attacks constantly emerge as defenses improve
  
- **Two key metrics needed**:
    - Violation rate (% of successful attacks)
    - False refusal rate (% of safe queries refused)
      
- A system that refuses ALL requests has 0% violation rate but is useless


**Why High-Stakes Adoption is Slow:**

- Security risks remain a significant roadblock
  
- No complete solution exists
  
- Risk compounds when models have access to real systems (databases, APIs)