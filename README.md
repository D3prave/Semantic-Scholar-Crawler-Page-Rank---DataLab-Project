<h1 align="center">IN PROGRESS</h1>

---

# Academic Citation Crawler & Dashboard

This project implements a robust academic web crawler using the Semantic Scholar API. Citation and metadata are fetched, filtered, and stored in PostgreSQL. Deduplication is handled with RedisBloom. A FastAPI dashboard provides real-time monitoring of crawling progress and system metrics.

---

## 📚 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [How to Run](#-how-to-run)
- [Crawler Script (`crawler.py`)](#-crawlerpy)
- [Dashboard Script (`dashboard.py`)](#-dashboardpy)
- [API & Interfaces](#-api--interfaces)
- [Technologies Used](#-technologies-used)
- [Architecture](#-architecture)
- [Project Structure](#-project-structure)

---

## 📌 Overview

This project implements a robust academic crawler to collect citation data via the [Semantic Scholar API](https://api.semanticscholar.org/). The data is processed, stored in PostgreSQL, and de-duplicated using Redis with a Bloom filter. A live dashboard (FastAPI) monitors crawler performance and remote server resources.

---

## ✨ Features

- ✅ Distributed crawling with queue-based task management
- 🧠 Deduplication with Redis Bloom filters + SQL fallback
- 🔁 Retry logic with exponential backoff
- 🕸️ Citation graph generation (directed edges)
- 📊 Real-time dashboard with memory, rate, and paper stats
- 📦 Background memory checks on remote servers via SSH

---

## ▶ How to Run

### 🔧 Requirements

- PostgreSQL and Redis running locally
- Semantic Scholar API Key (`API_KEY`) (API key can be left blank for public usage)
- Python dependencies (pip install -r requirements.txt)
- Python packages: see `requirements.txt`

### 🚀 Commands

Start a **fresh** crawl with seed IDs:
```bash
python crawler.py --fresh <seed_id1> <seed_id2>
```
Resume a previously interrupted crawl:
```bash
python crawler.py --resume
```
Run the **Dashboard** server:
```bash
uvicorn dashboard:app --host xxxx --port xxxx --reload
```

---

## 🧪 crawler.py

The main script that handles fetching, deduplication, citation parsing, and task queue management.

### 🧩 Key Components

- send_request: Rate-limited API requests with retry logic
- fetch_references_paginated: Handles large reference sets (>1000)
- filter_new_ids: Bloom filter + SQL fallback deduplication
- safe_insert_citations: Robust insert with deadlock handling
- mark_processed: Marks paper as crawled in both Redis and SQL
- main(): Main crawl loop with seed support, resume, and batching

---

## 📊 dashboard.py

A FastAPI app providing real-time monitoring for crawler performance and system resource usage.

### 📡 Endpoints

- GET - HTML Dashboard UI
- GET /status - Returns crawler metrics as JSON
  
### 📈 Metrics Shown

- ✅ Number of processed papers
- 🔗 Citation edges discovered
- 🧠 Memory pressure (macOS local)
- 🧠 RAM usage on remote servers via SSH
- ⚡ Papers/second & 📈 papers/hour
- 🕒 Estimated time per 1000 papers

### 🧵 Background Tasks

- remote_ram_background_updater() – polls RAM usage every 60s
- speed_background_updater() – updates crawl rate every 15s

---

## 🔌 API & Interfaces

### 📡 Semantic Scholar API
Base URL: https://api.semanticscholar.org/graph/v1
- GET /paper/{id}/references: For paginated citation lists
- POST /paper/batch: For batched paper metadata

### 🔁 Redis
- paper_queue	- FIFO queue of papers to crawl
- processed_bloom	- RedisBloom filter to avoid duplicate paper IDs

### 🗃️ PostgreSQL
- processed_papers - Stores paper_id and its fields_of_study
- citations - Stores directed edges (citing_id → cited_id)

### 🌐 FastAPI Dashboard
- GET - HTML dashboard UI
- GET /status -	Returns system stats in JSON (see below)

---

## 🤖 Technologies Used

- **Python 3**
- **PostgreSQL** (`psycopg2`, `asyncpg`)
- **Redis** + Bloom Filter
- **FastAPI** (Dashboard backend)
- **asyncssh** (RAM monitoring via SSH)
- **Semantic Scholar API**
- **Tenacity** (for robust retry logic)
- **Uvicorn** (for serving dashboard)

---

## 📐 Architecture

                   +-----------------------+
                   |  Semantic Scholar API |
                   +----------+------------+
                              |
                              v
                   +-----------------------+
                   |      Crawler.py       |
                   |-----------------------|
                   |  - Batched Fetching   |
                   |  - Reference Parsing  |
                   |  - Deduplication      |
                   |  - Queue Handling     |
                   +----------+------------+
                              |
               +--------------+--------------+
               |                             |
               v                             v
      +-------------------+       +------------------------+
      |     PostgreSQL    |       |         Redis          |
      |-------------------|       |------------------------|
      |  - Processed IDs  |       |  - Task Queue (FIFO)   |
      |  - Citations      |       |  - Bloom Filter        |
      +-------------------+       +------------------------+
               |
               v
     +--------------------------+
     |      FastAPI Dashboard   |
     |--------------------------|
     | - Crawl Stats            |
     | - Crawl Speed            |
     | - Remote RAM Monitoring  |
     +--------------------------+
               |
               v
     +--------------------------+
     |     Web Browser UI       |
     +--------------------------+

---

## 📁 Project Structure
```
.
├── crawler.py
├── dashboard.py
└── requirements.txt
```

---
