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

# Kaggle Notebook
The Kaggle Notebook for this blogpost (including code implementations and demos) could be found at:

{{note}} [Kaggle Notebook for Supply Chain Acharya: A GenAI-Powered Digital Twin](<a href="https://www.kaggle.com/code/toshall/agentic-supply-chain-acharya" target="_blank" rel="noopener noreferrer">
  https://www.kaggle.com/code/toshall/agentic-supply-chain-acharya
</a>) {{end}}

# Problem Overview: Why Are the Shelves Empty—or Overflowing?
Imagine walking into your local store looking for eggs, only to find the shelves bare. Or picture a store associate asking, "Why are we getting so many eggs we can’t sell?" These everyday questions reflect a deeper issue in the retail supply chain: inventory mismanagement. Whether it’s understocking that disappoints customers or overstocking that drives up waste and cost, the underlying problem is surprisingly complex.

These problems are not just operational—they’re strategic and systemic. From decisions about the physical supply network to individual item parameters affecting availability, the supply chain is governed by a vast, interdependent set of rules.

# Our Solution: Supply Chain Acharya
To tackle this challenge, our team developed Supply Chain Acharya, a GenAI-powered assistant built on top of a Digital Twin of a simplified retail supply chain. 

With this assistant, we could just tell it what we want, and it would:

- Understand our problem.
- Search across multiple databases store inventory,orders,forecast,sales, logistic lead times.
- Analyze and Root-cause and Recommend the best possible solution/next steps.
- Show results in a structured format like JSON or table

# Method
Supply Chain Acharya is a Gen AI-powered assistant designed to uncover these root causes dynamically helping the store managers, replenishment planners rootcause and recommend on next steps. The functionality has been tested with the information using a Digital Twin of a retail supply chain network.

## Digital Twin of a retail supply chain network
{{note}} [Kaggle Notebook for Digital Twin of a retail supply chain network](<a href="https://www.kaggle.com/code/sonneygeorge/scacharyavapr16v1" target="_blank" rel="noopener noreferrer">
  https://www.kaggle.com/code/sonneygeorge/scacharyavapr16v1
</a>) {{end}}

We created a 30-day simulation of demand and replenishment for a small but representative network:
•	2 Distribution Centers (DCs)
•	10 Suppliers
•	30 Stores
•	20 Items (2 per supplier in each store)
•	Simulated lead times between vendors, DCs, and stores
•	Calendar-based logic for shipping and receiving 

This setup allows us to inject disturbances, model replenishment strategies, and understand the downstream effects of seemingly small parameter changes.

Details of the supply chain network could be viewed below, and 
[Kaggle Notebook for Digital Twin of a retail supply chain network](https://www.kaggle.com/code/sonneygeorge/scacharyavapr16v1)

![supply network](/img/in-post/supply_network.png)

## Supply Chain Acharya – A Multi-Agent System for Intelligent Supply Chain Diagnosis

Based on the usage senario and database structure, We aimed to develop an AI agent system that could: 
- Understand vague human queries
- Auto-generate SQL to explore relevant supply data
- Identify bottlenecks & root causes
- Present results in structured, readable form

## What Can Supply Chain Acharya Do?
•	Answer questions like: "Why did this item stock out on Day 10?"
•	Trace inventory flows across the supply chain
•	Help diagnose root causes like incorrect lead times or missed order windows
•	Explain the logic behind replenishment quantities and forecast mismatches

## Why It Matters
This proof-of-concept lays the groundwork for:
•	Scalable decision support tools for large retailers
•	Explainable AI in operational environments
•	Improved training for replenishment analysts
•	Faster diagnosis of recurring inventory issues

---
