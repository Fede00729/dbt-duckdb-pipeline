# dbt + DuckDB Job Postings Pipeline

My learning project while following the course **dbt for Data Analytics & Engineering**
by Luke Barousse ([YouTube](https://youtu.be/1Bf049kXV2g)).

🚧 Work in progress, built lesson by lesson.

## What it does

Takes raw job postings (parquet files) and turns them into a small warehouse in a single
DuckDB file, using dbt.

```
raw parquet files ──▶ staging ──▶ marts
(data/raw/)           (view)      (tables)
```

| Model | Type | What it is |
|---|---|---|
| `job_postings` | view | Cleaned postings, scraper error rows removed |
| `jobs_per_month` | table | Postings per month and role |
| `top_locations` | table | Top 10 locations by postings |
| `monthly_summary` | table | Monthly totals and % of remote postings |

Models are documented and have basic data tests (`not_null`, `unique`).

## Tools

- [dbt](https://www.getdbt.com/) 2.x
- [DuckDB](https://duckdb.org/)
- [uv](https://docs.astral.sh/uv/) for the Python environment

## How to run it

```bash
# 1. Install dependencies
uv sync

# 2. Download the raw data (from the repo root)
uv run scripts/download_data.py

# 3. Build and test everything
cd analytics
uv run dbt build
```

dbt also needs a profile called `analytics` in `~/.dbt/profiles.yml`:

```yaml
analytics:
  target: dev
  outputs:
    dev:
      type: duckdb
      path: ../data/dev.duckdb
      threads: 1
```

Raw data and `.duckdb` files are not in the repo.
