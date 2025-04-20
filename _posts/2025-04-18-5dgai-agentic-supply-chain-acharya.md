---
layout: post
title: "Supply Chain Acharya: A GenAI-Powered Expert Assistant to Solve Retail Supply Chain Challenges"
subtitle: "Gen AI Capstone 2025Q1 by Kaggle"
date: 2025-04-18 14:24:45
header-style: text
catalog: true
author: "Sonney George, Hari Priya Ramamoorthy, Yuan Shang, Toshal Warke (Alphabetical Order)"
tags: [Gen AI Intensive Course with Google, Kaggle, Generative AI, Few Shot Learning,  Agent, CoT, Function Calling, Langraph, Supply Chain,Simulation,Inventory Management, AIinLogistics, RetailTech]
---
{% include linksref.html %}

>Multi-Agent Minds, One Mission.

## Kaggle Notebook
The Kaggle Notebook for this blogpost (including code implementations and demos) could be found at:
{{note}} [Supply Chain Acharya: A GenAI-Powered Expert Assistant to Solve Retail Supply Chain Challenges](<a href="https://www.kaggle.com/code/haripriyaram51/agentic-supply-chain-acharya-final" target="_blank" rel="noopener noreferrer">
  https://www.kaggle.com/code/haripriyaram51/agentic-supply-chain-acharya-final
</a>) {{end}}

{{note}} Vedio for this project can be found here(<a href="(https://medium.com/@haripriyaram51/bee428f66764)" target="_blank" rel="noopener noreferrer">
  https://medium.com/@haripriyaram51/bee428f66764
</a>) {{end}}

{{note}} Other media presentations could be found here(<a href="https://medium.com/@haripriyaram51/bee428f66764" target="_blank" rel="noopener noreferrer">
  https://medium.com/@haripriyaram51/bee428f66764
</a>) {{end}}

### Highlights
This project leverages Generative AI to develop an intelligent assistant designed to enhance supply chain management. The system employs a LangGraph workflow to orchestrate decision-making and interactions between a primary Generative AI agent and a clarification node. To improve the accuracy of question reasoning and data retrieval, the project incorporates several advanced prompting techniques and tools:

- ✅ Agent System with LangGraph: LangGraph is used to define the workflow and communication pathways between a primary Generative AI agent, designed for core supply chain tasks, and a clarification node, which handles user interaction and ambiguity resolution. This structure enables a focused agent to perform its primary function while the clarification node manages the conversational flow.
- ✅ Few-Shot and Chain-of-Thought Prompting: These prompting strategies are employed to guide the primary LLM agent in generating more accurate and reasoned responses, particularly when dealing with complex supply chain queries. The primary agent uses a prompt optimized for its core tasks, leveraging Few-Shot and Chain-of-Thought prompting.
- ✅ Function Calling: This technique is utilized to retrieve relevant data schemas, database constraints, and query results from external data sources (supply chain database), ensuring the agents have access to necessary information.
- ✅ Structured Output: The primary agent and the clarification node exchange information in dict format within the LangGraph workflow, providing a standardized and structured way to share data and updates throughout the decision-making process. Additionally, the final response from agent is formated so that human could read easily.

By combining these technologies, the project aims to create a more robust and optimized approach to supply chain management, enabling more informed and efficient decision-making.

### Problem Overview: Why Are the Shelves Empty—or Overflowing?
Imagine walking into your local store looking for eggs, only to find the shelves bare. Or picture a store associate asking, "Why are we getting so many eggs we can’t sell?" These everyday questions reflect a deeper issue in the retail supply chain: inventory mismanagement. Whether it’s understocking that disappoints customers or overstocking that drives up waste and cost, the underlying problem is surprisingly complex.

These problems are not just operational—they’re strategic and systemic. From decisions about the physical supply network to individual item parameters affecting availability, the supply chain is governed by a vast, interdependent set of rules.

### Our Solution: Supply Chain Acharya
To tackle this challenge, our team developed Supply Chain Acharya, a GenAI-powered assistant built on top of a Digital Twin of a simplified retail supply chain. 

With this assistant, we could just tell it what we want, and it would:

- Understand our problem.
- Understand the data shema of available database.
- Search across multiple data table including store inventory,orders,forecast,sales, logistic lead times.
- Sharing information between analytic steps.
- Analyze and Root-cause and Recommend the best possible solution/next steps.
- Show results in a structured format like JSON or table for better understanding.

Supply Chain Acharya is a Gen AI-powered assistant designed to uncover these root causes dynamically helping the store managers, replenishment planners rootcause and recommend on next steps. The functionality has been tested with the information using a Digital Twin of a retail supply chain network.

### Database decription and generation: Digital Twin of a retail supply chain network
{{note}} [Kaggle Notebook for Digital Twin of a retail supply chain network](<a href="https://www.kaggle.com/code/sonneygeorge/scacharyavapr16v3" target="_blank" rel="noopener noreferrer">
  https://www.kaggle.com/code/sonneygeorge/scacharyavapr16v3
</a>) {{end}}

We created a 30-day simulation of demand and replenishment for a representative supply chain network comprising:
-    2 Distribution Centers (DCs)
-    10 Suppliers
-    30 Stores
-    20 Items (2 per supplier in each store)
-    Simulated lead times between vendors, DCs, and stores
-    Calendar-based Sales for each item in each store
-    Calendar-based Orders for each item in each store
-    Calendar-based logic for shipping and receiving 

To manage the resulting data complexity and ensure efficiency, we designed a database with 16 tables, normalized to 3rd Normal Form. This database infrastructure enables us to inject disruptions, model various replenishment strategies, and analyze the downstream effects of parameter changes.

Details of the supply chain network could be viewed below, and 
[Kaggle Notebook for Digital Twin of a retail supply chain network](https://www.kaggle.com/code/sonneygeorge/scacharyavapr16v3)

![supply network](/img/in-post/supply_network.png)
In the simulation, Suppliers ship items to DCs, which then distribute them to Stores based on demand. Daily, each store generates item sales based on a simulated normal distribution. Stores subsequently create orders based on these sales and forecasts, which are transmitted to Suppliers. Suppliers then prepare item shipments to stores via the DCs. The lead time for each order is simulated using a normal distribution.

### Supply Chain Acharya – An Agent Assistant for Intelligent Supply Chain Diagnosis

Based on the intended use cases and the database structure, we aimed to develop an AI agent system capable of:
- Understanding ambiguous human queries
- Automatically generating SQL to explore relevant supply chain data
- Identifying bottlenecks and root causes
- Presenting results in a structured, easily readable format

To achieve this, we require an interactive **Generative AI agent** capable of engaging with users to obtain a more precise definition of their questions. For this interactive process, we utilize **LangGraph** to manage the decision flow between the agent and a clarification node, effectively modeling the refinement of user inquiries.

### Function Calling for External Data Integration to AI agents
To enhance information extraction from our database, we implemented four functions designed to provide the LLM with database access capabilities:

- List Tables: Retrieves a list of all tables within the database.
- Describe Table: Provides a detailed schema description for a specified table.
- Execute Query: Executes a given SQL query and returns the results.

These functions enable the LLM to dynamically interact with the database, allowing for more informed and contextually relevant responses.

<ul id="profileTabs" class="nav nav-tabs">
    <li  class="active"><a class="noCrossRef" href="#list_tables" data-toggle="tab">list_tables</a></li>
    <li><a class="noCrossRef" href="#describe_table" data-toggle="tab">describe_table</a></li>
    <li><a class="noCrossRef" href="#execute_query" data-toggle="tab">execute_query</a></li>
</ul>

<div class="tab-content">
<div role="tabpanel" class="tab-pane active" id="list_tables" markdown="1">

```python
# SQL Tools
def list_tables(conn) -> list[str]:
    """
    Retrieve the names of all tables in the SQLite database.
    
    Args:
        conn: A SQLite connection object.
    
    Returns:
        List[str]: A list of table names.
    """
    print(' - DB CALL: list_tables()')
    cursor = conn.cursor()
    cursor.execute("SELECT name FROM sqlite_master WHERE type='table';")
    tables = cursor.fetchall()
    return [t[0] for t in tables]




```

</div>


<div role="tabpanel" class="tab-pane" id="describe_table" markdown="1">


```python

def describe_table(conn, table_name: str) -> list[tuple[str, str]]:
    """
    Look up the schema (column names and types) of the specified table.
    
    Args:
        conn: A SQLite connection object.
        table_name (str): The name of the table.
    
    Returns:
        List[Tuple[str, str]]: A list of (column_name, column_type).
    """
    print(f' - DB CALL: describe_table({table_name})')
    cursor = conn.cursor()
    cursor.execute(f"PRAGMA table_info({table_name});")
    schema = cursor.fetchall()
    return [(col[1], col[2]) for col in schema]
```
</div>


<div role="tabpanel" class="tab-pane" id="execute_query" markdown="1">


```python

def execute_query(conn, sql: str) -> list[list[str]]:
    """
    Execute an SQL statement and return the results.
    Args:
        conn: The SQLite connection object.
        sql (str): The SQL query string.
    Returns:
        List[List[str]]: The results as a list of rows.
    """
    print(f' - DB CALL: execute_query({sql})')
    cursor = conn.cursor()
    cursor.execute(sql)
    return cursor.fetchall()
```
</div>

</div>



### LangGraph for Supply Chain Acharya
We design our AI agent system using LangGraph shown below
![LangGraph for Supply Chain Acharya](/img/in-post/lang_graph.png)

This **LangGraph** workflow orchestrates a **four-node** supply chain analysis pipeline powered by a specialized **AI agents**. The process begins when users enter through the **input_node** that provides initial instructions before proceeding to the Human Node for interaction. Here, users submit their supply chain questions, which then route to the **supplychain_agent** for validation. Using **Chain-of-Thought (CoT)** and **few shots prompting** reasoning, the supplychain_agent determines whether the input requires clarification (looping to the **clarification_node**) or contains sufficient detail to get the results. This iterative refinement continues until the input meets quality thresholds.

After user clarify their questions, **supplychain_agent** will feature the the workflow progresses through four analytical stages:
- Interpret the user’s natural language question
- Explore the database schema using tools (not assumptions)
- Generate SQL queries to investigate all relevant angles (inventory, vendors, forecasts, shipments)
- Generate structured output with defined format


### Prompt of the Agent
The agent acts as the coordinator for user interactions. It validates user input by checking for missing or ambiguous entities (e.g., item, store, DC) to ensure every query is handled efficiently and directed to the appropriate downstream node. If further information is required for clarification, it initiates a loop back to the human node to gather the necessary details.

Furthermore, the prompt establishes key constraints and formatting guidelines:

* The agent must use provided tools (e.g., list_tables(), describe_table()) to explore the database schema and avoid asking the user for table or column names.
SQL queries should only be generated after the agent has understood the schema.
* Ambiguous terms should be resolved by querying the database for options and presenting them to the user for selection.
* The final response must adhere to a specified bullet-point format, clearly stating the root cause of the issue and providing actionable recommendations.

By providing this detailed set of instructions, the prompt aims to ensure consistent, accurate, and helpful responses from the agent, mimicking the expertise of a supply chain analyst.


 * Prompt for the Agent

```python
    """You are a helpful **supply chain root-cause analyst agent** with access to an SQL database for a retail store.

    Your job is to identify and explain **why a supply chain issue is happening** — such as:
    - Inventory stockouts or shortages
    - Vendor shipment delays
    - Sales forecast mismatches
    - Missed deliveries or receipts

    You will:
    - Interpret the user’s natural language question
    - Explore the database schema using tools (not assumptions)
    - Generate SQL queries to investigate all relevant angles (inventory, vendors, forecasts, shipments)

    ---

    STRICT RULES you MUST follow:

    1. **DO NOT ask the user for table or column names**  
    → Use `list_tables()` and `describe_table(table_name)` to explore the schema  
    
    2. **Only generate SQL AFTER** you've understood the schema through tool usage.

    3. **For ambiguous terms (like 'eggs', 'forecast', or 'Store X')**:  
    → Use the database to find possible options and ask the user to choose  
    → Phrase it like: "Which of the following items/stores do you mean?"  
    → Provide real entries from the table as “Options: [...]”


    Your Final Response Format (concise bullet points):

    1. **Root Cause**: What is likely causing the issue, with specific facts or numbers (e.g., stock levels, forecast gaps, missing shipments). Always use item names and store names, not IDs.

    2. **Recommendation**: What should the user do next to fix or investigate further?


    Do NOT:
    - Guess any schema
    - Output SQL queries or table structures
    - Ask the user for table/column names

    Do:
    - Think step-by-step using tool calls
    - Provide smart, plain-language insights

    Respond clearly and helpfully like a domain expert — no jargon.
    """
```


### 🛠️ Technology Stack & Generative AI Capabilities

| Component                  | Role                          | Key AI Capabilities                          |
|----------------------------|-------------------------------|---------------------------------------------|
| **Gemini Pro** (via LangChain) | Core reasoning engine         | • Function calling<br>• Few-shot prompting<br>• Dynamic context handling |
| **LangGraph**              | Stateful workflow orchestrator | • Agent collaboration<br>• Memory persistence<br>• Conditional routing |
| **OpenAI/Vertex AI**       | Structured output generation  | • JSON-mode completion<br>• Controlled generation<br>• Role-based prompts |
| **Pandas + SQLite**        | Data processing backbone      | • SQL-based analysis<br>• Tabular diagnostics<br>• Data validation |
| **Agentic Workflow**       | Task automation framework     | • Manager-agent delegation<br>• Self-correcting loops<br>• Context-aware rerouting |
| **Google Colab**           | Development environment       | • Reproducible experiments<br>• Notebook-as-interface<br>• Scalable prototyping |


### What Can Supply Chain Acharya Do?

-   Answer questions like: "Why did this item stock out on Day 10?"
-   Trace inventory flows across the supply chain
-   Help diagnose root causes like incorrect lead times or missed order windows
-   Explain the logic behind replenishment quantities and forecast mismatches

### Demo of Supply Chain Acharya

Below showed a user case of how Supply Chain Acharya work.

![Supply Chain Acharya Demo](/img/in-post/demo_supplyChain.png)

### ✍️ Authors

This project is developed by (Alphabetical Order):

*   **Dr. Sonney George**
    *   Kaggle: [@sonneygeorge](https://www.kaggle.com/sonneygeorge)
    *   LinkedIn: [Dr. Sonney George](https://www.linkedin.com/in/sonneygeorge/)
*   **Hari Priya Ramamoorthy**
    *   Kaggle: [@haripriyaram51](https://www.kaggle.com/haripriyaram51)
    *   LinkedIn: [Hari Priya Ramamoorthy](https://www.linkedin.com/in/haripriyaram51/)
*   **Dr.Yuan Shang**
    *   Kaggle: [@raymondyuanshang](https://www.kaggle.com/raymondyuanshang)
    *   LinkedIn: [Dr.Yuan Shang](https://www.linkedin.com/in/yuanshang2020/)
*   **Toshal Warke**
    *   Kaggle: [@toshall](https://www.kaggle.com/toshall)
    *   LinkedIn: [Toshal Warke](https://www.linkedin.com/in/toshal-warke/)

---
