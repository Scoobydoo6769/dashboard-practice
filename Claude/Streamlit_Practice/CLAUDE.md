# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A Streamlit practice folder. The real project is the sibling folder `../Student_Dashboard/` (one level up, directly under `Claude/`), a self-contained app with its own `requirements.txt`, `.venv`, `.streamlit/config.toml`, `data/`, `tests/` and Thai-language `README.md`. The top-level `.py` files are throwaway demos, and each is independent with no shared module:

- `python.py` is a 7-line `st.button` demo.
- `practice_regional.py` is a widget and chart demo. It reads the four CSVs in this folder (`Aggregated_Metrics_By_Country_And_Subscriber_Status.csv`, `Aggregated_Metrics_By_Video.csv`, `All_Comments_Final.csv`, `Video_Performance_Over_Time.csv`), so keep them.
- `DuckDB.sql` is a scratch query used by nothing.
- `Competition_Dataset_2569.xlsx` is a loose copy of one year's data. The dashboard reads only `../Student_Dashboard/data/`.

The git repo root is the home directory (`/Users/guitar`), not this folder, so scope `git add` and `git status` to this directory.

## Commands

Run everything from `Claude/Student_Dashboard/`. It has its own `.venv`, separate from the `.venv` in this practice folder.

```bash
cd /Users/guitar/Claude/Student_Dashboard
source .venv/bin/activate            # first time: python3 -m venv .venv && pip install -r requirements.txt
streamlit run app.py                 # http://localhost:8501
python tests/smoke_test.py           # headless AppTest check; or: pytest tests/smoke_test.py
pytest tests/smoke_test.py::test_first_run   # single test
```

There is no linter or build step. Run the smoke test after every dashboard change. It drives the sidebar filters through `streamlit.testing.v1.AppTest` and asserts nothing raises.

## Student_Dashboard architecture

The code is split into four layers. `app.py` only wires them together, and each of the other three owns one concern:

- `data.py` loads every `data/*.xlsx` (one file per academic year) and stacks them into one tidy DataFrame with internal columns `year, level, dept, male, female, total`. The `COLUMNS` dict maps the Thai source headers to those names. A new file with the right headers is picked up with no code change. `load()` builds a cache signature from file paths and mtimes, so adding or replacing a file invalidates `@st.cache_data`. The same module holds the query helpers (`pick`, `yearly`, `by_year_level`, `snapshot`, `cagr`). Raise `DataError` for any message that is safe to show the user.
- `charts.py` holds all Plotly figure builders, plus `show` and `show_table` for rendering. Colors come from `theme`.
- `theme.py` holds the color tokens, CSS injection and KPI cards.
- `app.py` handles page layout, the sidebar filters, KPIs and tabs.

Conventions to keep:
- Every sidebar filter widget key starts with `f_`. The reset button clears all keys with that prefix, so a new filter without the prefix won't be reset.
- Colors are defined in two places and must stay in sync: `theme.TOKENS` and `.streamlit/config.toml`.
- The level categorical order is defined by `LEVEL_ORDER` in `data.py`, with the residual level `ตกค้าง` always last. Unknown levels are inserted before it.
- The residual level is included in yearly totals. The level filter does not affect the residual KPIs and charts, which are a dedicated comparison. Growth is year-over-year, and average growth is CAGR over the selected range. The README lists these as intentional.
- Level `ป.ตรี` only has data from 2567.
- All UI strings and data are Thai. Keep files UTF-8 and use `Path(__file__).parent`-relative paths so the Community Cloud checkout works. Thai fonts load from Google Fonts, with a system-font fallback offline.

## Gotchas

- `Streamlit_Practice/.streamlit/secrets.toml` holds credentials from an earlier login-gated version of `python.py`. `Streamlit_Practice/.gitignore` excludes it and nothing reads it now. Never copy its contents anywhere.
- `Student_Dashboard/.gitignore` and `Streamlit_Practice/.gitignore` both exclude `.venv/` and `__pycache__/`. There is no `.gitignore` in `Claude/` or the home directory, so anything new outside these two folders is not ignored.
- `tests/smoke_test.py` still has a template docstring that mentions `data/dataset.csv`. The app actually reads `data/*.xlsx`.
