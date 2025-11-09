
### Context Window Problem

- **Context window = the amount of text a model can "see" at once**. Think of it like a person's short-term memory capacity

- Models have **hard limits** on how much information they can process in a single query (e.g., 128K tokens ≈ 400 pages)

- **Real-world problem**: Many applications need MORE information than fits in context
	- A codebase might be 10,000 files
	- Research requires reading 50+ books
	- Customer support needs access to years of documentation

> `If you just paste everything into the model's input, you either (a) hit the limit and fail, or (b) pay massive costs processing unnecessary information`
---

### RAG comes to rescue

**RAG tries to solve two problems:

1. **Technical limitation**: Gets around context window size
2. **Efficiency problem**: Finds and uses ONLY relevant information instead of dumping everything (efficiency)


**Why efficiency matters**:

- Every token you send to a model costs money
- Processing 50,000 tokens when you only need 500 wastes 99% of your budget
- More irrelevant information -> model gets confused ->> worse answers

> Instead of reading an entire encyclopedia to answer "What year did World War 2 start?", you open just the WW2 page. RAG does this automatically.


---

### RAG = Retrieve + Generate (and Augment)

**Step 1 - RETRIEVE**:

- System searches external storage (database, documents, internet)
- Finds the most relevant pieces of information for the user's question
- Returns only what's needed (maybe 5 paragraphs from 10,000 documents)

**Step 2 - GENERATE**:

- Takes the user's question + retrieved information
- Feeds both to the AI model
- Model generates answer using this ==**focused context**==


**Example flow**:

- User asks: "What's our refund policy for shoes?"
- Retrieve: System finds the "Shoe Refund Policy" document
- Generate: Model reads policy + question, answers: "30-day return window with receipt"


---

### Retriever

- **The retriever is the critical component** - if it grabs the wrong documents, the model generates wrong answers (garbage in = garbage out)

- **Two main retriever types**: 
  
	**Term-based (keyword matching)**:
    
    - Works like Ctrl+F search - finds exact word matches
    - Fast, cheap, easy to set up
    - Example: Search "machine learning" → finds docs with those exact words
    - Problem: Misses synonyms (won't find "AI" or "neural networks")


    **Embedding-based (semantic/meaning-based)**:
    
    - Converts text to numbers (vectors) that capture meaning
    - Finds documents with SIMILAR meaning, not just matching words
    - Example: Search "machine learning" → also finds "artificial intelligence", "neural networks", "deep learning"
    - Cost: Requires computing embeddings (slow, expensive), storing vectors (memory-heavy)

> Start with *term-based* for quick wins, upgrade to *embedding-based* for <u>better quality</u> when needed


---

### Vector Search & DB

- **Vector search = finding similar items in high-dimensional space**
    - Each document becomes a list of numbers (e.g., [0.2, -0.5, 0.8, ...])
    - Similar documents have similar numbers
    - "Similar" = close distance in this number space
- **Why it's everywhere**:
    - Google Search uses it to find relevant web pages
    - Netflix/Spotify use it to recommend content ("people who liked X also liked Y")
    - Face recognition uses it to match faces
    - RAG borrowed proven technology from these mature systems

---

### RAG ⊂ Agents (RAG is a subset of agents)

- **What this means**:
    - In RAG: Model has ONE tool → the retriever
    - In Agents: Model can have MANY tools → retriever + calculator + web browser + database + email + etc.
      
- **Why this matters for learning**:
    - If you understand RAG, you understand the basic agent pattern
    - Agent is just RAG with more tools added
    - RAG is the simplest, most common agent pattern

---

### RAG & Agents (similarity)

- **Two shared benefits**:
    1. **Bypass context limits**: Access unlimited external information
    2. **Stay current**: Pull fresh data instead of relying on stale training data
        - Training data might be from 2023
        - RAG/Agents can access today's news, latest docs, real-time databases

- **Agents go further**:
    - RAG = read-only (just fetches information)
    - Agents = read AND write (can take actions)
    - Examples: Send emails, book meetings, update databases, make purchases

---

### Agent

**Two components define any agent**:

1. **Environment** = where the agent operates
    - Web browsing agent → environment = internet
    - Coding agent → environment = codebase + terminal
    - Customer service agent → environment = ticketing system + knowledge base

2. **Tools** = actions the agent can perform
    - Web agent tools: navigate, click, scroll, search
    - Coding agent tools: read files, write code, run tests
    - Customer support tools: search docs, read tickets, send responses

> Agent capabilities = environment × tools

- Same model with different tools = completely different capabilities
- A model with only a calculator tool can't browse the web

---

### Agentic workflow

- **Agent = AI Brain + Tools** (the model doesn't execute, it decides)
- **The planning process**:
    1. **Receive task**: "Book me a flight to Tokyo under $1000"
    2. **Analyze**: What needs to happen? (search flights, compare prices, check dates)
    3. **Plan**: Sequence of steps to accomplish goal
    4. **Decide**: Which tool to use at each step
    5. **Execute**: Call the tools in order     

- **Example breakdown**:
    - Task: "Find cheapest hotel in Paris with 4+ stars"
    - Plan:
        - Use search_hotels(location="Paris", min_rating=4)
        - Use sort_by_price(results)
        - Use get_details(cheapest_hotel)
        - Generate response with recommendation

---

### Powerful agents -> Strong Model + Memory + Reflection

- **Multi-step tasks are hard** - each step can fail, models lose track
  
- **Three mechanisms help**:
    1. **Powerful model**: Stronger models = better planning
        - GPT-4 can plan 20-step tasks
        - GPT-3.5 struggles with 5-step tasks
          
    2. **Reflection** = agent checks its own work
        - After each step: "Did that work? Should I adjust?"
        - Example: Search returned no results → reflect → try different search terms
        - Catches mistakes before they compound
          
    3. **Memory system** = keeps track of progress
        - Short-term: Current conversation, recent steps
        - Long-term: Past conversations, learned facts, tool results
        - Prevents repeating work, remembers context
          
- **Real example**:
    - Task: "Research and write a report on climate change solutions"
    - Without memory: Agent might search same thing 5 times
    - With memory: Remembers what it already found, builds on it

---

### Capability ↑ = Danger ↑

- **Power-Risk Trade-off**: Capability ↑ = Danger ↑

- **Why more tools = more capability**:
    - 1 tool (retriever) → can answer questions
    - 5 tools (retriever, calculator, email, calendar, database) → can research, compute, communicate, schedule, store
    - 20 tools → can automate entire workflows
      

- **Why more automation 🤖 = more danger** ⛔️:
    - Read-only tools: Worst case = wrong answer
    - Write tools: Worst case = deleted database, sent spam to customers, charged credit cards

- **Real risks**:
    - Agent with database access could delete all records
    - Agent with email could send confidential data to wrong people
    - Agent with payment tools could make unauthorized purchases
  
> Every tool needs safeguards, limits, and human oversight for sensitive actions

---

### Memory & Efficiency

- **The memory problem**: Even WITH RAG/Agents, information accumulates
  
- **Why memory systems are needed**:
    - Conversation grows: 50 turns = too much context
    - Retrieved data piles up: Search 10 times = 10 chunks -> overload
    - Agent stores: tool outputs + plans + reflections + results = overflow

- **Memory system = smart filing cabinet**:
    - Decides what to keep in "active memory" (context window)
    - Decides what to store externally (long-term memory)
    - Retrieves from long-term when needed

- **Three-tier memory**:
    1. **Internal (model's training)**: Built-in knowledge, can't change
    2. **Short-term (context window)**: Current conversation, limited size
    3. **Long-term (external storage)**: Everything else, unlimited but slower to access

- **Analogy**:
    - Internal = what you learned in school (permanent but old)
    - Short-term = what you're thinking about right now (fast but limited)
    - Long-term = notes in your notebook (unlimited but need to flip through)

---

### RAG & Agents -> uses prompts for model adaptation

- **Prompt-based** = changing what you *TELL* the model, not changing the model

- **What this means**:
    - Model weights stay the same
    - You just give better inputs (retrieved context, tool results)
    - No retraining required
    - Fast to implement, easy to update
      
- **Limitation**:
    - Model is fundamentally the same
    - If model doesn't know how to code Python, RAG won't help
    - If model can't understand medical terms, retrieval won't fix it

- **Why finetune ??:**
    - Finetuning = actually changing the model's weights
    - Teaches model new skills RAG/Agents can't provide
    - More powerful but more complex
      
- **When to use what**:
    - RAG/Agents: Model is capable, just needs information
      
    - Finetuning: When Model needs to learn new capabilities
