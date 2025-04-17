---
layout: post
title: "Forecasting 10.000 Emergencies a Day: Inside SEM’s Cloud‑Native AI Platform"
subtitle: "How the Catalan EMS moved incident‑density prediction to Huawei Cloud and Kibana"
tags: [cloud, healthcare‑AI, time‑series, case‑study]
author: David
comments: true
---

> **Key takeaway 📈**   
> By lifting data, compute and visualisation into Huawei Cloud, the *Servei d’Emergències Mèdiques* can now **anticipate call‑centre load hours or weeks ahead** and re‑allocate ambulances in near real time. The platform blends classic ARIMA with modern XGBoost/LSTM models and streams results straight into Kibana dashboards for dispatchers.

## Why predict emergencies?

Between 4 000 and 70 000 incidents can hit Catalonia’s EMS on any given day, depending on ordinary seasonality—or extraordinary events like Covid waves. Manual roster planning struggled to keep up. The SEM–CIDAI project therefore set out to:

* **Forecast incident volume** at 1 h / 4 h / 24 h resolution up to one week ahead.  
* **Surface anomalies** (spikes, dips) as fast as they appear.  
* Provide a **stand‑alone demonstrator** so analysts can iterate without touching live systems.&#8203;:contentReference[oaicite:0]{index=0}&#8203;:contentReference[oaicite:1]{index=1}

---

## Cloud architecture at a glance

| Layer | Huawei Cloud service | Specs / purpose |
|-------|----------------------|-----------------|
| **Compute** | Elastic Cloud Server (ECS) | 8 vCPU, 32 GB RAM, 200 GB SSD + *A30 GPU* for training/inference&#8203;:contentReference[oaicite:2]{index=2}&#8203;:contentReference[oaicite:3]{index=3} |
| **Public access** | Elastic IP (EIP) | Dynamic BGP, up to 300 Mb/s to expose APIs & dashboards&#8203;:contentReference[oaicite:4]{index=4}&#8203;:contentReference[oaicite:5]{index=5} |
| **Durability** | Cloud Backup & Recovery (CBR) | Point‑in‑time snapshots of data & notebooks&#8203;:contentReference[oaicite:6]{index=6}&#8203;:contentReference[oaicite:7]{index=7} |
| **Search & viz** | Cloud Search Service (CSS) | 3‑node Elasticsearch (4 vCPU, 8 GB RAM each) + Kibana 7.10.2 for interactive dashboards&#8203;:contentReference[oaicite:8]{index=8}&#8203;:contentReference[oaicite:9]{index=9} |

The physical layout keeps **data at rest on the ECS disk, GPU next to CPU, and a private VPC** linking everything; only Kibana’s public URL is exposed for read‑only viewing.&#8203;:contentReference[oaicite:10]{index=10}&#8203;:contentReference[oaicite:11]{index=11}

---

## Data pipeline

1. **Ingest & unify**  
   *10‑year call history*—time‑stamp, location, incident code—plus **external open data** (weather, traffic, hotel occupancy, public events).&#8203;:contentReference[oaicite:12]{index=12}&#8203;:contentReference[oaicite:13]{index=13}  
2. **Daily ETL** into CSV ➜ Elasticsearch indices (historic + prediction).&#8203;:contentReference[oaicite:14]{index=14}&#8203;:contentReference[oaicite:15]{index=15}  
3. **Model refresh** on week/bi‑week/month cut‑offs; results auto‑indexed so dashboards update with zero manual clicks.&#8203;:contentReference[oaicite:16]{index=16}&#8203;:contentReference[oaicite:17]{index=17}  

---

## Modelling strategy

> “Pick the **best model per horizon**, not one model to rule them all.” — PT3 notes

| Horizon | Leading model | Rationale |
|---------|---------------|-----------|
| **1 day** | **LSTM / GRU** | Captures fine‑grained temporal patterns, lowest RMSE at ≤ 24 h&#8203;:contentReference[oaicite:18]{index=18}&#8203;:contentReference[oaicite:19]{index=19} |
| **7 days** | **XGBoost** | More robust as error grows with horizon; handles exogenous vars well&#8203;:contentReference[oaicite:20]{index=20}&#8203;:contentReference[oaicite:21]{index=21} |
| **Seasonal trends** | **SARIMA** | Provides statistical baseline & interpretability for capacity planning&#8203;:contentReference[oaicite:22]{index=22}&#8203;:contentReference[oaicite:23]{index=23} |

A standard 70‑15‑15 split (train–val–test) keeps metrics honest before the chosen model is re‑trained on *all* available data and promoted to production only if it beats the incumbent.&#8203;:contentReference[oaicite:24]{index=24}&#8203;:contentReference[oaicite:25]{index=25}

---

## Dashboards that dispatchers actually use

* **Historic vs. forecast overlay**—quickly spot divergence.  
* **Error ribbon** auto‑updates once ground truth lands to flag model drift.&#8203;:contentReference[oaicite:26]{index=26}&#8203;:contentReference[oaicite:27]{index=27}  
* **Heat‑maps** by region & shift for ambulance allocation.  
* **Alert thresholds** in Kibana Lens trigger when volume exceeds predicted 95ᵗʰ percentile.&#8203;:contentReference[oaicite:28]{index=28}&#8203;:contentReference[oaicite:29]{index=29}  

Because Kibana rides on CSS, analysts get full‑text search, vector similarity and Lens drag‑and‑drop without running extra servers.  

---

## Impact & next steps

Early tests show **double‑digit reductions in over‑staffing** during calm periods and faster escalation during spikes. The team plans to:

* Feed **real‑time streaming data** (not just daily batches).  
* Experiment with **Temporal Fusion Transformers** for longer horizons.  
* Offer an **open‑API** so neighbouring emergency services can plug in their data.&#8203;:contentReference[oaicite:30]{index=30}&#8203;:contentReference[oaicite:31]{index=31}  

---

## Final thoughts

Cloud isn’t just about elastic GPUs; it’s about *turning predictions into action fast*. By combining Huawei Cloud’s managed stack with a thoughtful mix of statistical and deep‑learning models, SEM’s new platform shows how public health services can leap from reactive to proactive—without waiting for the next crisis.

*Citation (white‑paper):* CIDAI‑PAI 2024‑02, “Predicció de Densitat d’Incidents del Sistema d’Emergències Mèdiques (SEM)”.&#8203;:contentReference[oaicite:32]{index=32}&#8203;:contentReference[oaicite:33]{index=33}
