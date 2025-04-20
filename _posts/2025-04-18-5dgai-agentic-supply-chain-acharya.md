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
{{note}} [Kaggle Notebook for Supply Chain Acharya: A GenAI-Powered Digital Twin](<a href="https://www.kaggle.com/code/toshall/agentic-supply-chain-acharya" target="_blank" rel="noopener noreferrer">
  https://www.kaggle.com/code/toshall/agentic-supply-chain-acharya
</a>) {{end}}

### Highlights
🎯 We built a multi-agent system powered by LangGraph and Gemini Pro that can:

-   Interpret natural language supply chain issues
-   Dynamically generate and execute SQL queries
-   Analyze and explain root causes of failures
-   Loop back for clarification if data is insufficient

🧩 Gen AI Capabilities Used

✅ Function Calling (for SQL schema-aware generation)

✅ Agents (Manager, SQL Generator, Executor, Clarifier, Analyzer)

✅ Stateful Workflow via LangGraph (loopback for clarification)

✅ Structured Output / JSON Mode

✅ Few-shot Prompting using Chain-of-Thought (CoT)

## Problem Overview: Why Are the Shelves Empty—or Overflowing?
Imagine walking into your local store looking for eggs, only to find the shelves bare. Or picture a store associate asking, "Why are we getting so many eggs we can’t sell?" These everyday questions reflect a deeper issue in the retail supply chain: inventory mismanagement. Whether it’s understocking that disappoints customers or overstocking that drives up waste and cost, the underlying problem is surprisingly complex.

These problems are not just operational—they’re strategic and systemic. From decisions about the physical supply network to individual item parameters affecting availability, the supply chain is governed by a vast, interdependent set of rules.

## Our Solution: Supply Chain Acharya
To tackle this challenge, our team developed Supply Chain Acharya, a GenAI-powered assistant built on top of a Digital Twin of a simplified retail supply chain. 

With this assistant, we could just tell it what we want, and it would:

- Understand our problem.
- Search across multiple databases store inventory,orders,forecast,sales, logistic lead times.
- Analyze and Root-cause and Recommend the best possible solution/next steps.
- Show results in a structured format like JSON or table

## Method
Supply Chain Acharya is a Gen AI-powered assistant designed to uncover these root causes dynamically helping the store managers, replenishment planners rootcause and recommend on next steps. The functionality has been tested with the information using a Digital Twin of a retail supply chain network.

### Digital Twin of a retail supply chain network
{{note}} [Kaggle Notebook for Digital Twin of a retail supply chain network](<a href="https://www.kaggle.com/code/sonneygeorge/scacharyavapr16v3" target="_blank" rel="noopener noreferrer">
  https://www.kaggle.com/code/sonneygeorge/scacharyavapr16v3
</a>) {{end}}

We created a 30-day simulation of demand and replenishment for a small but representative network:
-    2 Distribution Centers (DCs)
-    10 Suppliers
-    30 Stores
-    20 Items (2 per supplier in each store)
-    Simulated lead times between vendors, DCs, and stores
-    Calendar-based logic for shipping and receiving 

This setup allows us to inject disturbances, model replenishment strategies, and understand the downstream effects of seemingly small parameter changes.

Details of the supply chain network could be viewed below, and 
[Kaggle Notebook for Digital Twin of a retail supply chain network](https://www.kaggle.com/code/sonneygeorge/scacharyavapr16v3)

![supply network](/img/in-post/supply_network.png)
The Suppliers send items to DCs, and DCs distribute items to Stores on demand. Everyday, each store will generate sales for each item based on normal distribution and simultation. Then, the stores create orders based on sales and forecasts. The orders were sent to Suppliers, which will prepare items shipments to each store through DCs. The lead time of each order varies and were simulated through normal distribution.

### Supply Chain Acharya – A Multi-Agent System for Intelligent Supply Chain Diagnosis

Based on the usage senario and database structure, We aimed to develop an AI agent system that could: 
- Understand vague human queries
- Auto-generate SQL to explore relevant supply data
- Identify bottlenecks & root causes
- Present results in structured, readable form

To fullfill this, we need an interactive generative agent to interact with human to get the more clear defintion of questions. Also, we need multi agents for different sub tasks such as **question clarifications, SQL query generation and executions, Root cause analysis, and agent managements**. This part could best be implemented through **LangGraph**.



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

#### Agent-02: SQL Generation Agent
This agent translates validated user queries into executable, schema-aware SQL code. It uses dynamic prompting and metadata to understand the query structure. It returns a well-formed SQL query to fetch the requested insights.

This agent acts as the bridge between natural language and database execution.

#### Agent-03: SQL Execution Agent
This agent executes the SQL queries, retrieves results from the Digital Twin database and formats the output for easy interpretation. 


#### Agent-04: Root Cause Analyzer
This analytical component transforms raw simulation data into actionable insights by diagnosing the underlying causes of supply chain disruptions. Using a structured Chain-of-Thought framework, it systematically: (1) isolates critical patterns from the results, (2) generates prioritized corrective recommendations, and (3) proposes strategic follow-up questions to deepen the investigation when warranted. By revealing the fundamental "why" behind operational breakdowns, the agent empowers decision-makers with evidence-based understanding - converting retrospective analysis into proactive resolution strategies while maintaining full interpretive transparency throughout the diagnostic process.

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


---
