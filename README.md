# BYD France: Business Intelligence & Market Analytics

End-to-end Business Intelligence project analysing BYD's performance in the French new passenger car market, with a focus on electric (BEV) and plug-in hybrid (PHEV) vehicles.

**Status:** work in progress (Phase 1: data sourcing and verification)

## Business questions

| # | Question |
|---|---|
| 1 | How is BYD performing in France? |
| 2 | Which BYD models contribute most to registrations? |
| 3 | How is BYD growing over time? |
| 4 | How does BYD compare with competitors? |
| 5 | What opportunities exist in the French EV market? |
| 6 | What commercial recommendations can we make? |

## Methodology

| Phase | Description | Status |
|---|---|---|
| 1 | Data sourcing and verification | In progress |
| 2 | Data cleaning with Python | Not started |
| 3 | Exploratory data analysis | Not started |
| 4 | SQL database design and analysis (MySQL) | Not started |
| 5 | KPI development | Not started |
| 6 | Tableau dashboard | Not started |
| 7 | Power BI dashboard | Not started |
| 8 | Business insights and recommendations | Not started |
| 9 | Documentation and portfolio publication | Not started |

## Data sources

| Source | Content | Granularity |
|---|---|---|
| CCFA / AAA-DATA monthly registrations | New passenger car registrations by brand (incl. BYD and competitors) | Brand x month |
| SDES (French Ministry of Ecological Transition) | Market share by powertrain (BEV, PHEV, petrol, diesel) | Market x month |
| PFA / AAA-DATA press kit | Top 100 models | Model x year-to-date |

Full details (URLs, reporting periods, publication dates) are in [docs/source_register.csv](docs/source_register.csv).

## Data integrity rules

Only real, traceable sources are used; no registration figures are invented. Every figure is labelled as actual, derived or estimated. Missing observations are stored as missing, never as zero. BYD-specific figures are kept separate from aggregate French market data, and figures from sources with different scopes (e.g. CCFA vs SDES) are not combined in one ratio.

## Known limitations

No official open dataset splits registrations by brand and powertrain, so BYD's share of the BEV market cannot be calculated from official data alone. Model-level data covers only the Top 100 models. Registrations include demonstration and short-term rental vehicles, so they are not equal to retail sales.

## Tools

Python (Pandas) in Google Colab, MySQL, Excel, Tableau, Power BI, GitHub.

## Repository structure

```
byd-france-bi/
  data/raw/          original source files, never edited
  data/processed/    clean CSVs produced by notebooks
  notebooks/         Colab notebooks in execution order
  sql/               MySQL schema and analysis queries
  docs/              source register and documentation
```

## Author

Muhammad Asad Zaheer, MSc Data & Business Analytics, Rennes School of Business
