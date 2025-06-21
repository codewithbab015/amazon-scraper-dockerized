# 🕷️ Amazon Web Scraping Pipeline with Docker & CI/CD

## 🚀 Overview
The project extends the Amazon Web Scraper CLI by automating the ETL process using Docker.
This project scrapes Amazon product data using Playwright and loads it into a PostgreSQL database.  
The ETL pipeline is containerized with Docker and can be run either standalone or with Docker Compose for multi-container orchestration.  
Using Docker volumes, scraped data stored in PostgreSQL will persist even after container shutdown.

---

## ⚙️ Prerequisites

- [Docker](https://www.docker.com/) installed
- [Docker Compose](https://docs.docker.com/compose/install/) installed
- (Optional for dev) Python 3.8+ and virtual environment for manual debugging

---

## ▶️ Option 1: Run Docker Standalone

> Use this for single-stage execution (e.g., just extract or transform).

### 1. Login to Docker Hub

```bash
# Log in to Docker Hub
task docker:docker-login
````

### 2. Build Docker Image Locally

```bash
task docker:local-build \
  DOCKER_USER=<enter username> \
  IMAGE_NAME=amazon-scraper-cli \
  VERSION=latest \
  RUN_GROUP=electronics \
  RUN_TYPE=camera-photo \
  RUN_MODE=extract \
  MAX=1
```

### 3. Push Image to Docker Hub (Optional)

```bash
task docker:local-remote \
  DOCKER_USER=<enter username> \
  IMAGE_NAME=amazon-scraper-cli \
  VERSION=latest
```

### 4. Build and Push Image to Docker Hub (Optional)
```bash
task docker:remote-build \
  DOCKER_USER=<enter username> \
  IMAGE_NAME=amazon-scraper-cli \
  VERSION=latest
```

---

## ▶️ Option 2: Run with Docker Compose (with Data Persistence)

> Recommended for running the full pipeline and keeping data between runs.

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/amazon-scraper-dockerized.git
cd amazon-scraper-dockerized
```

### 2. Run with Docker Compose

```bash
docker-compose up --build
```

This will:

* Spin up the scraper container and PostgreSQL database
* Create a named volume (e.g., `pgdata`) to persist PostgreSQL data locally

### 3. Stop and Keep Data

```bash
docker-compose down
```

> ⚠️ This will **stop containers but keep the database volume**.

### 4. Stop and Remove Everything (Including Data)

```bash
docker-compose down -v
```

> ⚠️ Use `-v` only if you want to delete all volumes and start fresh.

---

## 🧭 ETL Pipeline Overview

| Stage     | Script                           | Description                                          |
| --------- | -------------------------------- | ---------------------------------------------------- |
| Extract   | `extract/scraper_extractor.py`   | Scrapes summary-level product data and saves as JSON |
| Transform | `transform/scraper_transform.py` | Extracts detailed data based on Step 1 output        |
| Load      | `load/scraper_load.py`           | Cleans and loads the data into PostgreSQL            |

---

## 🛠️ (Optional) Local Python Development

> For debugging or extending functionality outside of Docker.

### Create & Activate Virtual Environment

```bash
python -m venv .venv
source .venv/bin/activate
```

### Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### Install Playwright Browser

```bash
playwright install firefox
```