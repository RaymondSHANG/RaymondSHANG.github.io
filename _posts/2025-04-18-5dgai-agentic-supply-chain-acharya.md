---
layout: post
title: "Supply Chain Acharya: A GenAI-Powered Digital Twin to Solve Retail Inventory Mysteries"
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
{{note}} [Kaggle Notebook for Supply Chain Acharya: A GenAI-Powered Digital Twin](<a href="https://www.kaggle.com/code/haripriyaram51/agentic-supply-chain-acharya-final" target="_blank" rel="noopener noreferrer">
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

### Method
Supply Chain Acharya is a Gen AI-powered assistant designed to uncover these root causes dynamically helping the store managers, replenishment planners rootcause and recommend on next steps. The functionality has been tested with the information using a Digital Twin of a retail supply chain network.

#### Digital Twin of a retail supply chain network
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

#### Supply Chain Acharya – An Agent Assistant for Intelligent Supply Chain Diagnosis

Based on the intended use cases and the database structure, we aimed to develop an AI agent system capable of:
- Understanding ambiguous human queries
- Automatically generating SQL to explore relevant supply chain data
- Identifying bottlenecks and root causes
- Presenting results in a structured, easily readable format

To achieve this, we require an interactive **Generative AI agent** capable of engaging with users to obtain a more precise definition of their questions. For this interactive process, we utilize **LangGraph** to manage the decision flow between the agent and a clarification node, effectively modeling the refinement of user inquiries.

#### Function Calling for External Data Integration to AI agents
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
We design our multi-agent AI system shown below
![LangGraph for Supply Chain Acharya](/img/in-post/lang_graph.png)

This **LangGraph** workflow orchestrates a **six-node** supply chain analysis pipeline powered by four specialized **AI agents**. The process begins when users enter through the **Welcome Node** that provides initial instructions before proceeding to the Human Node for interaction. Here, users submit their supply chain questions, which then route to the **Manager Agent** for validation. Using **Chain-of-Thought (CoT)** reasoning, the Manager Agent determines whether the input requires clarification (looping back to the **Human Node**) or contains sufficient detail to proceed downstream. This iterative refinement continues until the input meets quality thresholds or reaches the maximum allowed clarification attempts.

Once validated, the workflow progresses through three analytical stages:
- The **SQL Generator Agent** transforms the refined input into executable, schema-aware database queries.

- The **SQL Executor Agent** runs these queries against the supply chain database. The results were wrapped in **JSON format** and were sent to **Root Cause Analyzer Agent**

- The **Root Cause Analyzer Agent** gets the query results and the original questions in dictionary format. Then,this agent employs LLM + CoT reasoning to diagnose core issues,and present evidence-backed conclusions.


#### Agent-01: Manager Agent
Acts as the coordinator for user interactions.It validates user input by checking for missing or ambiguous entities (e.g., item, store, DC) to ensures every query is handled efficiently and passed to the right downstream node. If more information is required, it will loop back to human node to get more information.
 * Prompt for Manager Agent

```python
    """
            You are a supply chain assistant. Extract store references (by number or name) and item references 
            (by name or “item <id>”) from user questions.

            Extract RAW:
            - store_mentions: ["6", "Dallas store", ...]
            - item_mentions: ["milk", "item 5", "eggs", ...]

            Respond ONLY in this JSON:
            {
            "store_mentions": [str],
            "item_mentions": [str]
            }
    """
```

#### Agent-02: SQL Generation Agent
This agent translates validated user queries into executable, schema-aware SQL code. It uses dynamic prompting and metadata to understand the query structure. It returns a well-formed SQL query to fetch the requested insights.

This agent acts as the bridge between natural language and database execution.

 * Prompt for SQL Generation Agent

 ```python
    """
    You are an expert SQL Generator Agent for a retail supply chain database.
    STRICT SCHEMA: Only use the tables & columns listed below—no others.

    {schema_description}

    --- EXAMPLES ---

    1) In: "What is the current inventory of item_id=1 at store 5?"
    Out:
    [
    "SELECT EODQty FROM InventoryOH WHERE store_id=5 AND item_id=1 ORDER BY InventoryDt DESC LIMIT 1;"
    ]

    2) In: "Show me the last 7 days of sales for item 2 at store 3."
    Out:
    [
    "SELECT Sales_dt, Sales FROM StoreSales WHERE store_id=3 AND item_id=2 ORDER BY Sales_dt DESC LIMIT 7;"
    ]

    3) In: "Compare forecast vs actual for item 4 at store 2 for this month."
    Out:
    [
    "SELECT Forecast_sales_dt, Forecast FROM StoreFC WHERE store_id=2 AND item_id=4 AND Forecast_sales_dt BETWEEN DATE('now','start of month') AND DATE('now');",
    "SELECT Sales_dt, Sales FROM StoreSales WHERE store_id=2 AND item_id=4 AND Sales_dt BETWEEN DATE('now','start of month') AND DATE('now');"
    ]

    --- END EXAMPLES ---

    User question: {question}
    Store ID: {store_id}
    Item IDs: {items}

    Please return **only** a JSON array of SQL string(s), e.g.:

    [
    "SELECT * FROM InventoryOH WHERE store_id=5 AND item_id=1;"
    ]
    """
 ```

#### Agent-03: SQL Execution Agent
This agent executes the SQL queries, retrieves results from the Digital Twin database and formats the output for easy interpretation. 


 * Prompt for SQL Generation Agent

 ```python
    """
            You are the SQL code executor using the tool execute_query_tool.
            Your job is to execute the SQL query and return only the raw rows from the database.
            Do not explain the results or provide any additional commentary.

            Always return the raw result in the same format as shown above.
    """
 ```

#### Agent-04: Root Cause Analyzer
This analytical component transforms raw simulation data into actionable insights by diagnosing the underlying causes of supply chain disruptions. Using a structured Chain-of-Thought framework, it systematically: (1) isolates critical patterns from the results, (2) generates prioritized corrective recommendations, and (3) proposes strategic follow-up questions to deepen the investigation when warranted. By revealing the fundamental "why" behind operational breakdowns, the agent empowers decision-makers with evidence-based understanding - converting retrospective analysis into proactive resolution strategies while maintaining full interpretive transparency throughout the diagnostic process.


 * Prompt for Root Cause Analyzer

 ```python
    """
    You are a supply‑chain insights assistant.

    The user asked:
    "{user_q}"

    The following SQL queries were executed and returned results:
    {qr_text}

    Please provide in three concise bullet points:
    1. The single most important insight from these results.
    2. A recommended next step or action for the user.
    3. A follow‑up question the user could ask to dig deeper.

    Respond in plain language, without SQL code or technical jargon.
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
- Answer questions like: "Why did this item stock out on Day 10?"
- Trace inventory flows across the supply chain
- Help diagnose root causes like incorrect lead times or missed order windows
- Explain the logic behind replenishment quantities and forecast mismatches

## Demo of Supply Chain Acharya

Below showed a user case of how Supply Chain Acharya work.

![Supply Chain Acharya Demo](/img/in-post/demo_supplyChain.png)

## ✍️ Authors

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
