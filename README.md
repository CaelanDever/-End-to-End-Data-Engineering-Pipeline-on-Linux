# End-to-End Data Engineering Pipeline on Linux

🧠 Overview
-

This project simulates a complete data engineering production workflow using only open-source tools on a Linux server. I ingest raw data from a public REST API, transform it with Python & Pandas, load it into a PostgreSQL database, and orchestrate everything with Apache Airflow.

This hands-on experience mirrors real enterprise-grade pipelines and showcases how to automate ETL/ELT operations end-to-end.

Technologies Used:
-

Python

Pandas

PostgreSQL

SQLAlchemy

Apache Airflow

Linux Shell (RHEL/CentOS)

REST API (public dataset: https://jsonplaceholder.typicode.com/posts)

Deployment Platform: Linux server (bare metal or VM)

📁 Project Structure
-
.
├── dags/                      # Airflow DAGs
│   └── data_pipeline.py
├── scripts/                   # Python scripts for ETL
│   ├── fetch_api_data.py
│   ├── transform_data.py
│   └── load_to_postgres.py
├── logs/                      # Pipeline logs
├── requirements.txt           # Python dependencies
├── docker-compose.yml         # (Optional) for Airflow
├── README.md
└── screenshots/               # 📌 Screenshot placeholders

# 🚀 Detailed Project Sections

# 1. ✅ Fetching Raw Data from API

What: Use Python's requests module to pull JSON data from a public API (jsonplaceholder.typicode.com/posts)

Why: APIs are common raw data sources for organizations

Real-world connection: Often used in dashboards, monitoring tools, reporting jobs

Learning moment: Practice HTTP GET, response codes, JSON parsing

📌 Screenshot Placeholder: fetch_api_data.py in terminal fetching API data

<img width="450" alt="pyapp" src="https://github.com/user-attachments/assets/5e6be800-7e1f-498f-9135-2093f1fa2155" />
<img width="478" alt="rawd" src="https://github.com/user-attachments/assets/f0c79c6a-531b-46b6-b6ab-e2a6057537c5" />


Troubleshooting Tip: Use response.status_code and .json() validation

# fetch_api_data.py
import requests
import json

url = "https://jsonplaceholder.typicode.com/posts"
response = requests.get(url)

if response.status_code == 200:
    with open("raw_data.json", "w") as f:
        json.dump(response.json(), f)
else:
    print(f"Failed to fetch: {response.status_code}")

2. 🧹 Data Cleaning and Transformation

What: Load raw JSON into Pandas, clean and normalize

Why: Raw data often has missing values, inconsistent types

Real-world connection: Data engineering teams spend 50–70% of time cleaning

Learning moment: DataFrames, type handling, renaming columns

📌 Screenshot Placeholder: Cleaned DataFrame in Pandas console

<img width="478" alt="transform" src="https://github.com/user-attachments/assets/65a9660c-b2c4-4da0-a269-606d54e2d52f" />

# transform_data.py
import pandas as pd
import json

with open("raw_data.json") as f:
    raw = json.load(f)

# raw is a list of dictionaries
if isinstance(raw, list):
    df = pd.DataFrame(raw)
else:
    raise TypeError("Expected list of records from API")

# Basic cleaning
df = df.dropna(subset=['title'])
df.rename(columns={'userId': 'user_id', 'id': 'post_id'}, inplace=True)

df.to_csv("cleaned_data.csv", index=False)

# 3. 🏗️ Load to PostgreSQL Database

What: Use SQLAlchemy to connect and load data into a relational table

Why: PostgreSQL is widely used for analytics and warehousing

Real-world connection: Data lakes → structured DBs

Learning moment: Schema creation, upserts, data typing

📌 Screenshot Placeholder: psql CLI showing new table and rows

<img width="470" alt="sqlsc" src="https://github.com/user-attachments/assets/a319676b-aadc-4818-acee-b14daf3b5bbd" />

📅 Pre-Configuration Steps

Before running this script, execute the following SQL commands inside the PostgreSQL shell:

CREATE USER pipeline_user WITH PASSWORD 'strongpass';
CREATE DATABASE pipeline_db;
GRANT ALL PRIVILEGES ON DATABASE pipeline_db TO pipeline_user;

📄 Updated Python Script

# load_to_postgres.py
import pandas as pd
from sqlalchemy import create_engine

# Use created credentials
engine = create_engine('postgresql://pipeline_user:strongpass@localhost:5432/pipeline_db')

df = pd.read_csv("cleaned_data.csv")
df.to_sql('posts_data', engine, if_exists='replace', index=False)

# 4. ⚙️ Automate with Apache Airflow

What: Define a DAG with tasks for fetch → transform → load

Why: Airflow is the standard orchestration tool for pipelines

Real-world connection: Data teams use DAGs for daily/hourly ingestion

Learning moment: Dependencies, operators, scheduling, retry logic

📌 Screenshot Placeholder: Airflow UI showing DAG runs

<img width="397" alt="dag" src="https://github.com/user-attachments/assets/945a4e9a-504a-4d4b-a418-ecb6b2b34153" />
<img width="461" alt="dagui" src="https://github.com/user-attachments/assets/ce06dbf1-323a-4ccc-8ec0-6ae1d1e80741" />
<img width="653" alt="airf" src="https://github.com/user-attachments/assets/87210483-feb8-456c-a88b-80487dea52b7" />

# dags/data_pipeline.py
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime

default_args = {
    'owner': 'airflow',
    'start_date': datetime(2024, 1, 1),
    'retries': 1
}

dag = DAG('data_pipeline', default_args=default_args, schedule_interval='@daily')

fetch = BashOperator(
    task_id='fetch_data',
    bash_command='python /usr/local/airflow/scripts/fetch_api_data.py',
    dag=dag
)

transform = BashOperator(
    task_id='transform_data',
    bash_command='python /usr/local/airflow/scripts/transform_data.py',
    dag=dag
)

load = BashOperator(
    task_id='load_to_postgres',
    bash_command='python /usr/local/airflow/scripts/load_to_postgres.py',
    dag=dag
)

fetch >> transform >> load

# 5. 🔍 Query the Database

What: Write SQL queries to extract insights

Why: Business teams rely on reports from data teams

Real-world connection: Ad hoc queries, dashboards, KPI tracking

📌 Screenshot Placeholder: Output of SQL query from psql

<img width="531" alt="quer" src="https://github.com/user-attachments/assets/84cef57f-cfc7-48f4-9209-c41e968f3784" />
<img width="325" alt="querr" src="https://github.com/user-attachments/assets/a3db3de3-4e6d-460b-bf42-e2e72a9858c8" />

-- Example Query
SELECT user_id, COUNT(*) AS post_count FROM posts_data GROUP BY user_id;

🧪 Challenges & Solutions
-
Issue: DNS resolution failed when reaching public API

Fix: Reconfigured /etc/resolv.conf and verified external connectivity

Issue: JSON structure mismatched expectations

Fix: Adjusted script to handle list-of-dictionaries instead of nested dict

Issue: Pandas not installed on system

Fix: Installed via pip3 install pandas

🌟 Summary: What I Learned & Built

✅ In this project, I:
-
Pulled real-world API data on a Linux server using Python

Cleaned and structured JSON into a flat table using Pandas

Built and tested a full ETL pipeline

Stored results in a PostgreSQL table

Scheduled and automated the flow with Airflow DAGs

Performed SQL analysis on loaded data

🧬 Cleanup Steps
-
Stop Airflow: docker-compose down or systemctl stop airflow

Drop PostgreSQL table (if needed):

DROP TABLE posts_data;

Delete temporary CSVs & JSON:

rm cleaned_data.csv raw_data.json

Clear Airflow logs:

rm -rf ~/airflow/logs/*

📙 References
-
Apache Airflow Docs

Pandas Documentation

PostgreSQL Official Docs

JSONPlaceholder API

✅ Ready for GitHub!
-
This file is fully GitHub-ready. Create a repo, add this as your README.md, and include your scripts/, dags/, and sample data files for others to replicate.

Congratulations — you've completed the project! 🎉
