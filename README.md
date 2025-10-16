# PL Standings 20-25

![Python](https://img.shields.io/badge/Python-3.13+-3776AB?logo=python&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0+-4479A1?logo=mysql&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)

## Overview

This project pulls the English Premier League standings from the API-FOOTBALL service (via RapidAPI), reshapes the payload with pandas, and upserts the records into a MySQL database. The workflow lives in `test.ipynb`, showing the full data journey from HTTP request to database persistence, while `main.py` serves as the package entry point.

## Key Features

- Fetches league standings for any Premier League season exposed by API-FOOTBALL.
- Normalizes API responses into a tidy pandas `DataFrame`.
- Converts the frame into tuples ready for bulk insertion.
- Executes an idempotent MySQL upsert so existing rows are updated rather than duplicated.
- Demonstrates parameterized SQL queries to guard against injection attacks.

## Project Layout

```text
.
├── assets/
├── main.py
├── test.ipynb
├── pyproject.toml
├── uv.lock
└── .env                # local secrets (never commit real keys)
```

## Prerequisites

- Python 3.13 or newer.
- Jupyter (Notebook or Lab) for running `test.ipynb`.
- A reachable MySQL 8.0+ instance with rights to create tables and insert data.
- An API-FOOTBALL key issued via [RapidAPI](https://rapidapi.com/api-sports/api/api-football/).

## Setup

1. Clone the repository and move into the project directory.
2. Create and activate a virtual environment.
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Windows: .venv\Scripts\activate
   ```
3. Install the runtime dependencies.
   ```bash
   pip install pandas requests python-dotenv mysql-connector-python ipykernel
   ```
   Using [uv](https://github.com/astral-sh/uv)? Run `uv sync` instead.

## Configuration

Populate a `.env` file with your credentials (keep real secrets out of version control):

```ini
# RapidAPI - Football
API_KEY=your_rapidapi_key
API_HOST=v3.football.api-sports.io

# MySQL Credentials
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_USER=pl_user
MYSQL_PASSWORD=super_secret
MYSQL_DATABASE=premier_league_db
```

You can change `SEASON` and `LEAGUE_ID` inside the notebook to target different competitions or years.

## Database Preparation

Create a `standings` table before running the notebook. A minimal schema that works with the current UPSERT statement looks like this:

```sql
CREATE TABLE standings (
    season SMALLINT NOT NULL,
    position TINYINT NOT NULL,
    team_id INT NOT NULL,
    team VARCHAR(100) NOT NULL,
    played TINYINT NOT NULL,
    won TINYINT NOT NULL,
    draw TINYINT NOT NULL,
    lost TINYINT NOT NULL,
    goals_for SMALLINT NOT NULL,
    goals_against SMALLINT NOT NULL,
    goal_diff SMALLINT NOT NULL,
    points SMALLINT NOT NULL,
    form VARCHAR(50),
    PRIMARY KEY (season, team_id)
);
```

The UPSERT defined in `test.ipynb` relies on this primary key so that repeated runs simply update existing rows.

## Running the Pipeline

1. Load the environment variables:
   ```bash
   export $(grep -v '^#' .env | xargs)  # Windows PowerShell: Get-Content .env | %{ if ($_ -and $_ -notmatch '^#') { $_ } }
   ```
2. Start Jupyter and open the notebook:
   ```bash
   jupyter notebook test.ipynb
   ```
3. Run the cells in order. The notebook will:
   - Request the standings JSON.
   - Project the relevant fields into a pandas `DataFrame`.
   - Generate tuples for batch insertion.
   - Execute `cursor.executemany` with an UPSERT statement and commit the transaction.
