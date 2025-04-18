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
The Kaggle Notebook for this blogpost could be found at:

{{note}} [Kaggle Notebook 1: Supply Chain Data Simulation](<a href="https://www.kaggle.com/code/sonneygeorge/scacharyav1" target="_blank" rel="noopener noreferrer">
  https://www.kaggle.com/code/sonneygeorge/scacharyav1
</a>) {{end}}

# Problem overview: Why Are the Shelves Empty—or Overflowing?
Imagine walking into your local store looking for eggs, only to find the shelves bare. Or picture a store associate asking, "Why are we getting so many eggs we can’t sell?" These everyday questions reflect a deeper issue in the retail supply chain: inventory mismanagement. Whether it’s understocking that disappoints customers or overstocking that drives up waste and cost, the underlying problem is surprisingly complex.

## The Problem at a Glance
•	Customer dissatisfaction due to stockouts of essential items
•	Operational waste from overstocking, especially perishables
•	Disconnected data across suppliers, DCs, and stores
•	Hidden parameters and overlooked configurations causing inefficiencies

These problems are not just operational—they’re strategic and systemic. From decisions about the physical supply network to individual item parameters affecting availability, the supply chain is governed by a vast, interdependent set of rules.

## Enter: Supply Chain Acharya
To tackle this challenge, our team developed Supply Chain Acharya, a GenAI-powered assistant built on top of a Digital Twin of a simplified retail supply chain. The goal? To simulate, diagnose, and answer questions about inventory behavior across the network in a human-friendly way.

## Our Approach: Digital Twin Simulation
We created a 30-day simulation of demand and replenishment for a small but representative network:
•	2 Distribution Centers (DCs)
•	10 Suppliers
•	30 Stores
•	20 Items (2 per supplier in each store)
•	Simulated lead times between vendors, DCs, and stores
•	Calendar-based logic for shipping and receiving (assumed open daily for this phase)
This setup allows us to inject disturbances, model replenishment strategies, and understand the downstream effects of seemingly small parameter changes.

## Tech Stack
•	Python for simulation and logic
•	SQLite for structured data management
•	Pandas for data analysis
•	GenAI (ChatGPT) as the question-answering interface
•	Visualizations using Matplotlib/Plotly for insights

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
