---
layout: post
title: "Supply Chain Acharya - Part 1"
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

{{note}} [Kaggle Notebook 1: Supply Chain Data Simulation](https://www.kaggle.com/code/sonneygeorge/scacharyav1) {{end}}



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

- 1. Initialize Static Entities

```python
# Generate core tables (once)
vendors = pd.DataFrame({
    'Vendor_name': [f'Vendor_{i}' for i in range(1,11)],
    'VendorCity': np.random.choice(['NYC', 'Chicago', 'LA'], 10)
})

dcs = pd.DataFrame({
    'DC_name': ['East_DC', 'West_DC'],
    'DCCity': ['NYC', 'LA']
})

stores = pd.DataFrame({
    'Store_name': [f'Store_{i}' for i in range(1,31)],
    'StoreCity': np.random.choice(['Boston', 'Seattle', 'Miami'], 30)
})

items = pd.DataFrame({
    'Item_name': ['Eggs', 'Milk', ...],  # 20 items
    'price': np.round(np.random.uniform(1.0, 10.0, 20), 2)
})
```
- 2. Define Network Relationships

```python
# Assign vendors to DCs (with lead times)
vendor_dc = pd.DataFrame({
    'Vendor_id': np.repeat(range(1,11), 2),  # 10 vendors x 2 DCs
    'DC_id': [1,2]*10,
    'Vendor_DC_LT': np.random.randint(3,7, 20)  # System Lead Time (SLT)
})

# Assign stores to DCs
store_dc = pd.DataFrame({
    'Store_id': range(1,31),
    'DC_id': [1]*15 + [2]*15,  # 15 stores per DC
    'DC_StoreLT': np.random.randint(1,3, 30)
})
```

- 3. Simulate Demand & Orders

```python
def generate_daily_demand(store_id, item_id, date):
    base_demand = 10  # μ
    noise = np.random.normal(0, 1)  # σ=1
    return max(1, int(base_demand + noise))  # Ensure ≥1

# Generate 30 days of forecasts/sales
dates = pd.date_range('2024-01-01', periods=30)
for date in dates:
    for store_id in range(1,31):
        for item_id in range(1,21):
            forecast = 10  # Constant forecast
            actual = generate_daily_demand(store_id, item_id, date)
            # Write to StoreFC and StoreSales tables
```

- 4. Order Fulfillment with Variability

```python
def process_orders():
    for _, order in orders.iterrows():
        # Simulate vendor delay (ALT = SLT + randomness)
        alt = vendor_dc.loc[(order.Vendor_id, order.DC_id), 'Vendor_DC_LT'] 
        alt += np.random.randint(-1, 2)  # -1/0/+1 day variability
        
        ship_date = order.OrderDt + pd.Timedelta(days=alt)
        # Write to VendorShips, DCReceipts, etc.
```

# Evaluation samples

## Inject Failure Scenarios

```python
# Force a vendor delay for eggs (Item_id=1)
vendor_ships.loc[vendor_ships.Item_id == 1, 'ShipDt'] += pd.Timedelta(days=3)

# Simulate demand spike for Store 5
store_sales.loc[(store_sales.Store_id == 5) & (store_sales.Sales_dt == '2024-01-15'), 'Sales'] *= 2
```

## Inventory Reconciliation

```python
def update_inventory():
    for store_id in range(1,31):
        for date in dates:
            eod_qty = bod_qty + receipts - sales  # Simplified logic
            # Write to InventoryOH
```

---
