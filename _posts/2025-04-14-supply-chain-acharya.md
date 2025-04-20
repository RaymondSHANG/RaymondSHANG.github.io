---
layout: post
title: "Supply Chain Acharya - Data simulation details"
subtitle: "Gen AI Capstone 2025Q1"
date: 2025-04-14 20:06:35
header-style: text
catalog: true
author: "Sonney George, Hari Priya Ramamoorthy, Yuan Shang, Toshal Warke (Alphabetical Order)"
tags: [Gen AI Intensive Course with Google, Kaggle, Generative AI, Few Shot Learning,  Agent, CoT, Function Calling, Langraph, Supply Chain,Simulation]
---
{% include linksref.html %}

>A supply chain is only as strong as its weakest data point.

# Kaggle Notebook
The Kaggle Notebook for this blogpost could be found at:

{{note}} [Kaggle Notebook 1: Supply Chain Data Simulation](<a href="https://www.kaggle.com/code/sonneygeorge/scacharyavapr16v3" target="_blank" rel="noopener noreferrer">
  https://www.kaggle.com/code/sonneygeorge/scacharyavapr16v3
</a>) {{end}}



# Overview
This project simulates a multi-tier supply chain network with 10 vendors (suppliers), 2 distribution centers (DCs), and 30 stores to analyze inventory disruptions. Each supplier provides 2 unique items, shipped to DCs with system-defined lead times (SLT), though actual deliveries (ALT) may vary due to delays. DCs then distribute items to stores, where daily demand follows a normal distribution (e.g., average 10 units ± random deviation). Calendars for shipping/receiving at each node introduce delays if orders fall on non-operational days. The simulation models common failure scenarios—like vendor delays or unexpected demand spikes—that lead to stockouts (e.g., "Why did Store X run out of eggs?"). By generating timestamped data on orders, shipments, and inventory levels, the system trains an AI agent to diagnose root causes, such as identifying whether a stockout resulted from late supplier deliveries, forecasting errors, or DC bottlenecks. The goal is to enable the agent to answer targeted supply chain queries using a combination of function calls (to fetch data) and chain-of-thought reasoning (to trace disruptions across the network).

# Database structure for a supply chain system
Database Structure for Supply Chain Simulation
This SQLite database models a multi-echelon supply chain with 14 relational tables tracking inventory flows from vendors to stores via distribution centers (DCs). Key components:

## 1. Core Entities

- *Vendor*, *DC*, and *Store* tables store location metadata (names, cities).

- *Item* tracks products with prices, linked to vendors via *VendorItem*.

## 2. Network Relationships

- *VendorDC* defines vendor-to-DC lead times (SLT), while *StoreDC* sets DC-to-store transit times.

## 3. Demand & Inventory

- *StoreFC* stores daily demand forecasts, validated against actual sales in StoreSales.

- *InventoryOH* captures daily opening/closing stock per store.

## 4. Order Fulfillment Pipeline

- *orders* initiates replenishment requests, triggering:

- Vendor shipments (*VendorShips*) → DC receipts (*DCReceipts*)

- DC shipments (*DCShips*) → store receipts (*StoreReceipts*)

- Timestamps enable lead time analysis (e.g., actual vs. planned).

## Design Notes

- Simulated variability: Actual lead times derive from ReceivedDt - OrderDt calculations.

- Stockout diagnosis uses joins across sales, inventory, and shipment tables.

- Foreign keys (not shown) enforce referential integrity for all relationships.

This schema supports AI agent training by providing traceable data linkages to answer questions like "Was the egg stockout caused by vendor delays or demand spikes?" through timestamp and quantity comparisons.

# Data simulation
Based on the database described above, we developped a data simulation pipeline for the supply chain system, designed to generate realistic inventory scenarios while maintaining referential integrity.

### 1. Simulate DC, Vendors, Stores, and Items
Randomly choose 2 DC city names, 10 Vendors, 30 Stores, and 20 daily used Items. An example code for generating DC table shown below
    ```python
        # 1. DC
        # Get 5 random cities from the top 50
        import random

        max_DC_Number = 2
        top_50_us_cities = [
        "New York, NY", "Los Angeles, CA", "Chicago, IL", 
        "Houston, TX", "Phoenix, AZ", "Philadelphia, PA",
        "San Antonio, TX", "San Diego, CA", "Dallas, TX",
        "San Jose, CA", "Austin, TX", "Jacksonville, FL",
        "Fort Worth, TX", "Columbus, OH", "Charlotte, NC",
        "San Francisco, CA", "Indianapolis, IN", "Seattle, WA",
        "Denver, CO", "Washington, DC", "Boston, MA",
        "El Paso, TX", "Nashville, TN", "Detroit, MI",
        "Oklahoma City, OK", "Portland, OR", "Las Vegas, NV",
        "Memphis, TN", "Louisville, KY", "Baltimore, MD",
        "Milwaukee, WI", "Albuquerque, NM", "Tucson, AZ",
        "Fresno, CA", "Sacramento, CA", "Kansas City, MO",
        "Mesa, AZ", "Atlanta, GA", "Omaha, NE",
        "Colorado Springs, CO", "Raleigh, NC", "Miami, FL",
        "Virginia Beach, VA", "Oakland, CA", "Minneapolis, MN",
        "Tulsa, OK", "Arlington, TX", "New Orleans, LA",
        "Wichita, KS", "Cleveland, OH"
        ]

        # random_cities = random.sample(top_50_us_cities, 2)

        # print(random_cities)
        # DCCity = [top_50_us_cities[n] for n in range(max_DC_Number)]

        DCCity = [top_50_us_cities[n] for n in range(max_DC_Number)]
        DC_name = ["DC_" + DCCity[n] for n in range(max_DC_Number)]
        #print(DCCity,DC_name)
        DC = pd.DataFrame({
            "DC_name": DC_name,
            "DCCity": DCCity})
        DC

        # insert 
        db_name = 'SupplyChainABC2.db'
        table_name = 'DC'
        insert_dataframe_to_sqlite(DC, db_name, table_name)
        execute_query("select * from DC")
    ```
### 2. Simulate VendorDC, VendorItem, StoreDC
Each vendors will ships to all DC(VendorDC);
Each Store only get items from One DC (StoreDC);
Each Item go to One Vendor (VendorItem);
Each Store sells all Items.

### 3. StoreFC
Forcast the sales of each item in each store using Normal Distributions with different mean and variance for each item and store for each date

### 4. StoreSales
Simulate the first day of sales, where we assume enough inventory. This first day sales could be a random factor multiply by StoreFC at Step3.

### 5. InventoryOH
Simulate the first day InventoryOH based on the first day StoreSales at Step4, so that inventory at the begin of day should be larger than sales. Generate end of day inventories based on sales, and the begin of day inventories.

### 6. orders
Simulate orders based on these steps:
-     Caculate OUTL = (LT + RT + SSdays) * FC
-     Calculate OP  = (LT + 0.5 * RT + SSdays) * FC
-     OH = EODQty
-     OO = Orderqty for all outstanding orders -  even if ordered today it is expected to be delivered within LT+RT window
-     Calculate AvailableInv = OH + OO
-     Rule: Create an Order = if AvailableInv < OP , (OUTL - AvailableInv) else 0

### 7. Update StoreReceipts, DCReceipts
Based on Orders, simulate StoreReceipts and DCReceipts.


# Saving data for AI agents
After generating db, we could upload it as dataset, make it public so others could use it.


Example code

```python
import sqlite3

db_file = "SupplyChainABC.db"
db_conn = sqlite3.connect(db_file)
# Insert DataFrame into SQL table, replace if table exists
df.to_sql('my_table', db_conn, if_exists='replace', index=False)

# Close connection
db_conn.close()

print("DataFrame inserted into SQLite successfully!")
```
---
