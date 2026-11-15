# dbt + DuckDB Job Postings Pipeline

![Pipeline](assets/pipeline.png)

My learning project while following the course **dbt for Data Analytics & Engineering**
by Luke Barousse ([YouTube](https://youtu.be/1Bf049kXV2g)).

🚧 Work in progress, built lesson by lesson (see [Progress](#progress)).

## What it does

Takes raw job postings (parquet files) and turns them into a small warehouse in a single
DuckDB file, using dbt. GitHub Actions rebuilds it on every push and publishes it, so anyone
can query it.

| Model | Layer | Type | What it is |
|---|---|---|---|
| `job_postings` | staging | view | Cleaned postings, scraper error rows removed |
| `jobs_per_month` | marts | table | Postings per month and role |
| `top_locations` | marts | table | Top 10 locations by postings |
| `monthly_summary` | marts | table | Monthly totals and % of remote postings |

Models are documented and have basic data tests (`not_null`, `unique`).

## Tools

- [dbt](https://www.getdbt.com/) 2.x
- [DuckDB](https://duckdb.org/)
- [uv](https://docs.astral.sh/uv/) for the Python environment
- GitHub Actions to build and test on every push (and weekly)
- GitHub Releases to publish the warehouse file

## Query the published warehouse

No setup needed, just DuckDB:

```sql
INSTALL httpfs; LOAD httpfs;
ATTACH 'https://github.com/Fede00729/dbt-duckdb-pipeline/releases/download/warehouse/prod.duckdb' AS jobs;
USE jobs;
SELECT * FROM monthly_summary;
```

It's read-only. The marts tables work; `job_postings` doesn't, because it's a view that reads
the raw parquet files from the machine that built it.

## How to run it locally

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

## How CI works

On every push to `main` (and every Monday), [`.github/workflows/dbt_build.yml`](.github/workflows/dbt_build.yml):

1. installs uv and the dependencies,
2. downloads the raw data,
3. runs `dbt build` with the CI profile in [`ci/profiles.yml`](ci/profiles.yml) → `data/prod.duckdb`,
4. uploads `prod.duckdb` to the `warehouse` release.

If a data test fails, the build stops and nothing gets published.

## Progress

- [x] Project setup with uv and dbt
- [x] Raw data download script
- [x] First models, linked with `ref()`
- [x] Raw data declared as a dbt `source()`
- [x] Docs and descriptions (`schema.yml`, `dbt docs`)
- [x] Materializations: staging as view, marts as tables
- [x] Generic data tests
- [x] CI with GitHub Actions + publish to a GitHub Release
- [ ] Next lessons…

The diagram source is [`assets/pipeline.excalidraw`](assets/pipeline.excalidraw) (open it in excalidraw.com).
