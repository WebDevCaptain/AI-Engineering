
### Interdisciplinary Nature of Finetuning

Finetuning sits at the intersection of multiple domains. You need to understand:

- **Transfer learning** (classical ML): The idea that a pre-trained model's knowledge can be adapted to new tasks
  
- **PEFT (Parameter-Efficient Fine-Tuning)**: Modern innovation addressing the resource problem of training massive models
  
- **Low-rank factorization**: Mathematical principle that you can approximate large matrices with smaller ones (this is why LoRA works)
  
- **Memory constraints**: Why you can't just "retrain everything" with billion-parameter models
  
- **Hyperparameter tuning**: Practical skills to make finetuning actually work

---

### Framework Accessibility vs Strategic Complexity

The mechanical act of finetuning is **accessible** - you don't need to write training loops from scratch. Frameworks like Hugging Face PEFT, Axolotl, and LitGPT abstract away complexity.

**However** - and this is crucial - knowing _when_, _what_, and _how_ to finetune requires deeper understanding. The technical barrier is low, but the strategic barrier is high.

---

### Decision Framework: When to Finetune

**Finetuning is not always the answer.** Critical decision framework:

**Finetune when:**

- Your model has **behavioral problems** (wrong format, wrong style, doesn't follow instructions)
- You need specific output structures (JSON, SQL, domain-specific formats)
- You want to teach the model a particular "way of speaking" or responding


**Don't finetune when:**

- Your model has **information problems** (missing facts, outdated data) → Use RAG instead
- Prompt engineering hasn't been fully explored yet
- You don't have quality training data
- The cost/complexity isn't justified by improvement

This is foundational because many people jump to finetuning when simpler solutions would work.

---

### Form vs Facts: RAG vs Finetuning

**The golden rule: "Finetuning is for FORM, RAG is for FACTS"**

- **RAG (Retrieval-Augmented Generation)**: Gives your model _external knowledge_ it can reference. Solves: "The model doesn't know X" or "The model's knowledge is outdated"
  
- **Finetuning**: Changes how the model _behaves_ - its style, format, instruction-following. Solves: "The model knows the answer but expresses it wrong" or "The model doesn't follow the format I need"
  

> Research showed RAG + base model often outperforms RAG + finetuned model for current events questions. This proves finetuning isn't about adding information.

---

### Historical Evolution of Finetuning

**Historical context matters:** Originally, "finetuning" meant retraining every parameter in the model. This was feasible when models had millions of parameters (BERT: 110M).

**Transfer learning workflow:**

1. Pre-train on massive general data
2. Finetune all weights on task-specific data
3. Deploy

This established the paradigm that pre-trained models could be adapted, which was revolutionary for NLP.

---

### The Scaling Crisis and Memory Constraints

**The scaling crisis:** When models went from millions to billions of parameters (GPT-3: 175B), full finetuning became impossible for most:

**Why memory explodes during training:**

- You need to store: model weights + activations + **gradients** + **optimizer states**
- For each trainable parameter with Adam optimizer, you need: 1 weight + 1 gradient + 2 optimizer states = **4 values**
- Example: 13B model with Adam = 26GB (weights) + 78GB (gradients/optimizer) = **104GB minimum**
- Most consumer GPUs: 12-24GB

This created a **democratization problem**: Only big companies could adapt large models. PEFT emerged to solve this.

---

### PEFT: Core Concept and Motivation

**Core insight of PEFT:** You don't need to update _all_ parameters to adapt a model effectively.

**The surprising finding:** Models finetuned with only **0.01% of trainable parameters** (millions instead of billions) can match or exceed full finetuning performance.

**Why this works (hypothesis):**

- The model already learned general knowledge during pre-training
- Task adaptation requires updating only a small "subspace" of the model
- Most parameters can stay frozen - they're doing their general-purpose job fine

**PEFT methods:**

- **Adapters**: Add small trainable modules, freeze original weights
- **LoRA**: Add low-rank matrices that update existing weights
- **Soft prompts**: Add trainable tokens to the input

All share goal: minimal trainable parameters = minimal memory.

---

### **Quantization and Precision Reduction**

**Quantization = precision reduction**

**Foundational concept:**

- Normally, each parameter/gradient is stored in 32-bit (FP32) or 16-bit (FP16) format
- Quantization stores them in 8-bit, 4-bit, or even lower
- **Memory savings:** 32-bit → 4-bit = **87.5% reduction**

**Example - QLoRA:**

- Stores base model weights in **4 bits** (instead of 16)
- Keeps LoRA adapters in 16 bits for training stability
- Result: Can finetune 13B model on consumer GPU (24GB)

**Key trade-off:** Lower precision = less memory but potential accuracy loss. However, research shows careful quantization maintains performance surprisingly well.

---

### **LoRA: Mechanism and Mathematics**

**Why LoRA dominates:** It solves multiple problems simultaneously:

**How LoRA works (conceptual):**

1. Take a large weight matrix W (the original model weights)
2. Instead of updating W directly, create two small matrices A and B
3. Their product (A × B) approximates the update to W
4. Only train A and B (tiny compared to W)
5. After training, merge A×B back into W - no inference slowdown


**Mathematical foundation - Low-rank factorization:**

- A 1000×1000 matrix has 1 million parameters
- Factor it into 1000×10 and 10×1000 = only 20,000 parameters
- The "10" is the **rank** - lower rank = more compression


**Why it's practical:**

- You're not adding new layers (unlike original adapters) - no inference latency
- Drastically fewer parameters to train
- Can be merged back seamlessly

---

### **LoRA: Modularity and Serving**

**LoRA's modularity advantage:--**

**Multi-model serving:**
- Load one base model in memory
- Swap different LoRA adapters for different tasks
- Example: One 7B base model + 100 different LoRA adapters (6MB each)
- Instead of hosting 100 separate 7B models (14GB each)


**Data efficiency:**
- Full finetuning might need 50K-1M examples for good results
- LoRA can work well with 1K-10K examples
- Why: You're learning a smaller, more focused transformation


**Sharing and reuse:**
- LoRA adapters can be shared like pre-trained models
- Communities build adapter collections for specific domains (coding, medical, etc.)
- You can download and use others' adapters without retraining

---

### **Model Merging: Combining Capabilities**

**Model merging = combining capabilities without combining resources**

**Core concept:** You can mathematically combine the weights of multiple models to create a single model that:

- Has capabilities of all constituent models
- Uses less memory than running them separately
- Can potentially perform better than any individual model

**Key use cases:**

1. **Multi-task learning without catastrophic forgetting:**
    - Traditional problem: Train model on Task A, then Task B → forgets Task A
    - Merging solution: Train separate models on A and B in parallel → merge them
    - Result: One model that does both tasks

2. **On-device deployment:**
    - Problem: Phone has limited memory, need 3 different models
    - Solution: Merge 3 models into 1 → fits in device memory
    - Enables privacy (data never leaves device)

3. **Federated learning:**
    - Deploy same model to 1000 devices
    - Each learns from local data
    - Periodically merge all models
    - Everyone benefits from collective learning


**Main approaches:**

- **Linear combination (averaging)**: Add/average model weights
- **Task arithmetic**: Add or subtract capabilities (can remove biases!)
- **SLERP**: Merge along curved path (spherical interpolation)
- **Layer stacking**: Take layers from different models, stack them


**Why it works:** Models finetuned from the same base learn "deltas" (changes). These deltas can be combined because they're in the same parameter space.

---

### **The Data Quality Bottleneck** in Finetuning

**The real bottleneck: data quality, not compute**

**Why instruction data is hard:**

1. **Format requirements:** Need (instruction, response) pairs that are:
    - Factually correct
    - Well-formatted
    - Representative of actual use cases
    - Diverse enough to prevent overfitting

2. **Quality over quantity:**
    - 100 high-quality examples > 10,000 low-quality ones
    - Bad data teaches bad behaviors (can worsen hallucinations)
    - Domain expertise often required (medical, legal, technical)

3. **Expensive to create:**
    - Human annotation: $2-20 per example
    - Expert annotation: $50-200 per example
    - Takes time to build quality datasets

**The irony:**

- Technical barrier to finetuning: Solved by LoRA/PEFT
- Data barrier: Still a major challenge
- This shifts bottleneck from "can we afford GPUs?" to "can we get good data?"

---

#### Lessons learned

1. **Decision-making**: When to finetune vs when to use alternatives (RAG, prompting)
2. **Resource awareness**: Why memory is the constraint and how different techniques address it
3. **Method selection**: Understanding trade-offs between full finetuning, PEFT, and specific approaches like LoRA
4. **Practical constraints**: Recognizing that data quality, not just compute, determines success
5. **Modern paradigm shift**: From "train everything" to "train minimally but strategically"

This discussion moves you from _"I need to finetune my model"_ to _"Do I need to finetune? If yes, what's the most efficient approach given my constraints?"_