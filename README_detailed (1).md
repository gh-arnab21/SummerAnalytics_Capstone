# 🚗 Real-Time Dynamic Parking Pricing Dashboard

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1EJVB5KEEsG_vzsu49-3Oofi5fG1qThDV?usp=sharing)

> 📍 **Colab Notebook**: [Click here to open](https://colab.research.google.com/drive/1EJVB5KEEsG_vzsu49-3Oofi5fG1qThDV?usp=sharing)

---

## 📝 Overview

This project builds a real-time dynamic pricing system for 14 parking lots using live streaming data, processed via Pathway, and visualized using Bokeh + Panel. The aim is to dynamically update parking prices based on occupancy, demand, queue length, special days, and even competition from nearby parking lots.

---

## 🧰 Tech Stack

| Tool/Library | Purpose |
|--------------|---------|
| **Pathway**  | Real-time data streaming & temporal processing |
| **Panel**    | Building interactive visual dashboards |
| **Bokeh**    | Rendering time-series plots for pricing |
| **Pandas**   | Data manipulation in Colab |
| **Google Colab** | Development notebook |
| **Render / HuggingFace** | Optional deployment for the dashboard |

---

## 📊 Architecture Diagram

```mermaid
flowchart TD
    A[CSV Input (Per Parking Lot)] --> B[Pathway Streaming Engine]
    B --> C1[Model 1: Linear Baseline]
    B --> C2[Model 2: Demand-Based Pricing]
    B --> C3[Model 3: Competitive Geo Pricing]
    C1 --> D[Bokeh Line Plots]
    C2 --> D
    C3 --> D
    D --> E[Panel Dashboard (Grouped by Lot)]
    E --> F[Served via panel serve / deployed to Render]
```

---

## ⚙️ Models Explained

### ✅ Model 1: Baseline Linear Pricing
**Formula**: `price = 10 + (occ_max - occ_min) / cap`  
- Uses Pathway’s tumbling window (1 day) to group data
- Computes max/min occupancy and divides difference by capacity
- Each parking lot (SystemCodeNumber) is grouped separately
- Plots one curve per lot in a shared Bokeh figure

### ✅ Model 2: Demand-Based Pricing
**Formula**: 
```text
demand = α(occupancy/capacity) + β(queue) + γ(traffic) + δ(special_day) + ε(vehicle_weight)
price = base * (1 + λ * normalized_demand)
```
- All categorical features (traffic level, vehicle type) are encoded manually
- Weights (α, β, γ, δ, ε) are hand-tuned
- Normalization helps control demand scores
- Prices are clipped to a range [5, 20]

### ✅ Model 3: Competitive Pricing Model
- Extends Model 2 with location awareness and competition effects
- Calculates:
  - `competition_factor = 1 - (num lots nearby / 10)`
  - `location_premium = +10% if GPS is within a hot zone`
- Final price is calculated as:
```text
price = base * (1 + λ * demand + 0.2 * location - 0.2 * (1 - competition))
```
- Pricing range constrained to [5, 25]

---

## 📈 Plotting Structure

Each model generates:
- A `Bokeh` line chart with `SystemCodeNumber`-wise grouping
- A single plot showing **14 lines (one per parking lot)** using color mapping
- Red scatter points on top of line chart to show individual updates
- `Panel` is used to group these plots and serve them

```python
pn.Column(viz1, viz2, viz3).servable()
```

Each model defines a `plot_function(source)` which is passed into Pathway’s `plot()` method.

---

## 🗂 Folder Structure

```bash
.
├── data/                 # Generated per-lot CSVs
├── app.py                # Main Panel app
├── models/               # model1.py, model2.py, model3.py (optional split)
├── README.md             # This file
├── requirements.txt      # Dependencies
└── Parking_Pricing_Dashboard_Report.pdf
```

---

## ▶️ How to Run

### 🖥️ Local:

```bash
pip install -r requirements.txt
panel serve app.py --autoreload
```

### 🌐 Deploy (Render/HuggingFace):

```bash
panel serve app.py --address 0.0.0.0 --port 10000 --allow-websocket-origin=your-app-name.onrender.com
```

---

## 👤 Author

**Arnab Deka**  
IIT Guwahati · Real-time ML + Visualization Enthusiast  
GitHub: [yourusername](https://github.com/yourusername)

---

## 📘 License

Licensed under MIT License.