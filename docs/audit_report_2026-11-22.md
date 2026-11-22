# Technical audit: byd-france-bi

**Date:** 22 November 2026 · **Scope:** every file in the repository at commit `5ce19cc` (66 files, verified byte for byte against a ZIP download), plus the commit history.
**Method:** independent re-implementation, not a review of the existing code's own checks. The PDFs were re-extracted with a different tool, the KPIs recomputed with new code, the notebooks re-executed, the DAX semantics emulated, and the published numbers compared with CCFA's own printed figures.

---

## Summary

| Area | Verdict |
|---|---|
| Data integrity and extraction | ✅ **Reliable.** Two independent extraction methods agree on every number |
| KPIs (Python, SQL) | ✅ **Correct.** Every published figure reproduced; growth matches CCFA's printed growth |
| Reproducibility | ✅ PDF → raw → processed reproduced byte for byte. Power BI package now reproducible (fixed) |
| DAX measures | ⚠️ **1 confirmed bug** (wrong growth with several years in context) and 1 inconsistency, both fixed; still untested in Power BI itself |
| Documentation | ⚠️ README was wrong about status and scope. Fixed |
| Public release | ⛔ **Blocked by licensing**: the CCFA PDFs can't be redistributed electronically without authorisation |

---

## 1 · What was verified and found correct

| Check | Evidence | Result |
|---|---|---|
| Months | 34 consecutive months, Jan 2024 – Oct 2026; none missing or duplicated | ✅ |
| PDF extraction | Independent re-extraction of all 34 PDFs with `pdftotext -layout` (notebook 01 uses `pdfplumber`): 1 474 brand rows | ✅ 0 differences in monthly counts, YTD counts and printed % changes |
| Extraction ↔ printed shares | CCFA prints each brand's share; registrations ÷ market reproduce it for all 1 474 rows | ✅ max diff 0.0005 pp (rounding) |
| Totals | Sum of brands = CCFA printed Total, every month, month and YTD | ✅ |
| Periods and source IDs | Period in each PDF header = period in the data; every file has a register row with a URL | ✅ 34/34 |
| Manual or invented figures | All 1 440 `actual` values trace to their raw PDF row. 33 `aggregated` = Fiat + Abarth exactly. 1 `imputed` (Jaguar YTD Jan 2025, blank in the PDF), documented | ✅ none found |
| Brand names | 52 printed labels → 51 brands, one brand per label; typo "ALPHA ROMEO" and the merged "ABARTH/FIAT" handled | ✅ |
| Duplicates | No duplicate brand-month keys; no double counting (Fiat/Abarth components excluded from the universe) | ✅ |
| YTD revisions | 37 brand + 15 market gaps, all logged. The 4 large AUTRES restatements are **fully explained** by brands entering or leaving CCFA's list (e.g. Oct 2026: Chery, Omoda and Zeekr's earlier months, 1 365, minus Skyworth, 74, = −1 291) | ✅ |
| Growth vs CCFA | Our YTD growth = CCFA's printed growth for all **852** comparable brand rows | ✅ ≤ 0.1 pp |
| KPIs | BYD 5 415 / 13 533 / 28 086; +185.3%; 2.03%; rank 32/40 → 16/44; Tesla +117.1%, MG +23.7%, Leapmotor +172.6%, Xpeng +154.1%; Chinese-owned 7.07–7.20%; BYD part 28.7%; market −5.02% / +4.37% | ✅ all reproduced from scratch |
| Reproducibility | Re-ran notebooks 01 and 02 on the raw PDFs: all 6 output CSVs **byte-identical** to the committed files | ✅ |
| SQL joins and NULLs | Views join on primary keys (1:1 brand-month ↔ date ↔ market ↔ brand), so no fan-out. Growth uses a year − 1 join and returns NULL with no prior figure. No COALESCE-to-zero | ✅ |
| SQL checks | 22 load + 28 view checks pass (MySQL 8.0.46 in notebooks 04–05; re-run here on a SQLite replica) | ✅ |
| Credentials and personal data | Scanned all files: no keys, tokens, passwords, emails or private Drive links; notebook metadata clean | ✅ |
| Business conclusions | Notebook 03 and SQL findings use "registrations", not sales; no causes claimed; partial 2026 never compared with full years | ✅ |

---

## 2 · Issues found

### Critical bugs (results incorrect)

**C1 · DAX year-on-year measures compare the wrong periods when several years are in context**
1. **Where:** `docs/kpi_definitions.md` K04 `Registrations (same period LY)`, K05 `Market share % (same period LY)`, K12 `Market growth % vs LY`; the same three measures in `data/powerbi/measures.dax`.
2. **Problem:** the prior-year period was built from `MAX(dim_date[year])` and `MAX(dim_date[month])` separately. With no year slicer, max year = 2026 and max month = 12, so the current period (Jan–Oct 2026, via `LASTDATE`) was compared with **full-year 2025**.
3. **Why it matters:** a card without a year slicer would show BYD growth **+107.5% instead of +185.3%**. That's a wrong headline number.
4. **Evidence:** emulation of the DAX filter logic on the Power BI package. Year = 2026 gives 185.3% ✅. No year selected gives 107.5% ❌. Years 2025 + 2026 give 107.5% ❌.
5. **Fix:** read year and month from the last date in context (`VAR last_date = MAX(dim_date[date_key])`, then `YEAR`/`MONTH`), the same date `LASTDATE` uses.
6. **Status:** ✅ **fixed and tested** by emulation (185.3% in every slicer state; Jan–Jun 2026 = 140.2%, matching CCFA's printed figure). Automated as check B04. ⚠️ Not yet executed inside Power BI. A new row in the `data/powerbi/README.md` validation table tests it.

### Major issues (reliability)

**M1 · Public release blocked: CCFA documents can't be redistributed electronically** ⛔
1. **Where:** `data/raw/*.pdf` (34 CCFA reports + 1 PFA flash report).
2. **Problem:** CCFA's legal notice (ccfa.fr/mentions-legales) reserves all reproduction rights, *including downloadable documents*, and forbids electronic reproduction without CCFA's express authorisation. Paper copies are allowed only if free, unmodified and cited.
3. **Why it matters:** a public repository containing the PDFs is electronic reproduction. The repo **was public for a short time on 22 Nov 2026** and was switched back to private the same afternoon.
4. **Evidence:** CCFA legal notice. GitHub shows 0 stars and 0 watchers, but access logs aren't available.
5. **Recommended fix (needs your decision):**
   - (a) Ask CCFA for written permission, or
   - (b) publish a **new** public repository without the PDFs. Deleting them in a new commit is not enough: they stay in the git history.
   - Also decide whether the extracted full tables (`brand_month_raw.csv`) may be published. That is a database-right question, not answered here (I'm not a lawyer).
6. **Status:** ❌ **not fixed** (requires your decision; raw data preserved as instructed). README files now carry a rights notice.

**M2 · README misrepresented the project's status and scope**
1. **Where:** `README.md`.
2. **Problem:**
   - It said Phase 1 was in progress and Phases 2–9 not started; five phases were complete.
   - It claimed "a focus on BEV and PHEV", but the data has no powertrain split by brand (the README's own limitations said so).
   - It listed SDES and a PFA press kit as sources, but neither is in the repo or used.
   - It listed Q2 (model mix) without saying it can't be answered.
3. **Why it matters:** a recruiter reading it would get a wrong picture of both progress and scope.
4. **Evidence:** file inventory and commit history (71 commits, 7 Nov – 22 Nov 2026).
5. **Fix:** README rewritten from verified facts. It has a phase table with evidence, a question-coverage table, headline findings, a validation summary, reproduction steps, limitations and licensing.
6. **Status:** ✅ fixed and committed.

**M3 · Power BI data package was not reproducible**
1. **Where:** `data/powerbi/*.csv`.
2. **Problem:** the four CSVs were produced by queries that weren't committed.
3. **Why it matters:** nobody could regenerate or audit the dashboard's inputs.
4. **Fix:** added `sql/07_powerbi_extract.sql` (the exact queries) and `scripts/validate_project.py --powerbi`.
5. **Status:** ✅ fixed and tested. The extract SQL reproduces all four committed CSVs exactly (SQLite replica of the MySQL model; not yet re-run on MySQL itself).

**M4 · DAX rank changed under unrelated brand slicers**
1. **Where:** K06 `Rank (period)` and `Brands ranked (period)`.
2. **Problem:** `ALL(dim_brand[brand])` keeps other `dim_brand` filters. With a "Chinese-owned" slicer, BYD's rank would be among Chinese brands only, unlike the SQL definition (market-wide).
3. **Fix:** `ALL(dim_brand)`.
4. **Status:** ✅ fixed (logic); test row added to the Power BI validation table; not yet run in Power BI.

### Minor issues (documentation and presentation)

| # | Where | Problem | Status |
|---|---|---|---|
| m1 | `notebooks/README.md`, `sql/README.md` | Listed files that don't exist (`02_data_validation`, `04_kpis`, `02_load_data.sql`, `03_analysis_queries.sql`) | ✅ fixed |
| m2 | `data/processed/README.md`, `data/raw/README.md` | Described planned content (SDES xlsx, press kits; "planned main table") | ✅ fixed |
| m3 | `docs/source_register.csv` | `retrieved_on` is empty for all 37 rows | ❓ unresolved: exact download dates can't be verified, so they weren't filled in |
| m4 | `docs/source_register.csv` | SRC002 (SDES) and SRC003 (PFA press kit) are "to_check" with no file | Documented as unused in the README; register left unchanged |
| m5 | `notebooks/01`, `02` | Committed without outputs | Not changed; reproducibility verified byte for byte instead |
| m6 | `sql/04_views.sql` (`v_brand_period`) | Computes `growth_pct` for AUTRES, which isn't comparable year on year (CCFA restates its prior-year base). Not used by any KPI, query or rank | Documented; optionally set to NULL for AUTRES |
| m7 | GitHub profile README | Link to the BYD repo broke when it went private again | ✅ replaced with "private while licensing is checked" |

### Improvements

| # | Suggestion | Status |
|---|---|---|
| i1 | Automated validation: `scripts/validate_project.py`, 29 checks (data, KPIs, PDF re-extraction, Power BI package, DAX logic). A planted error was caught by 4 checks | ✅ added |
| i2 | Parameterise the "latest month". `dim_date` ends at `2026-10-01` and the like-for-like period is month 10, both hard-coded. Correct today, but needs editing when November data arrives | Recommended |
| i3 | Build the `.pbix`, complete the validation page (now 19 expected values), then publish screenshots | Next step (Phase 7) |
| i4 | Show the Chinese-owned share with and without Leapmotor (ownership judgement call) | Recommended |
| i5 | Run `validate_project.py` automatically on every commit (GitHub Actions) | Recommended |
| i6 | Use `Market share % (period)` on year-level cards; `(month)` re-sums months, which differs slightly from CCFA's published YTD | Documented in KPI definitions |

---

## 3 · Unresolved questions (not confirmed bugs)
1. Can the extracted full tables be published, or only aggregated results? This needs CCFA/AAA-DATA or legal advice.
2. Did anyone view or clone the repo while it was public? GitHub doesn't show this for personal repos.
3. Does the DAX behave in Power BI exactly as emulated? It will be confirmed on the validation page.
4. Retrieval dates for the source register.

## 4 · Final assessment

**Technically reliable? Yes, for the data, Python and SQL layers.** I tried to break the numbers with two independent extraction methods, a fresh KPI implementation, byte-for-byte notebook reproduction and CCFA's own printed growth rates. Everything held. The one real calculation bug (C1) was in DAX that no dashboard uses yet, and it's fixed.

**Ready to present to recruiters?** Partly.
* ✅ **Now:** in interviews, as a data engineering and SQL case study (pipeline, validation, star schema, KPI spec). This is unusually rigorous for an internship portfolio.
* ⛔ **Not yet as a public repo:** licensing (M1) has to be resolved first.
* ⏳ **Not yet as a "BI project":** there's no dashboard (Power BI in progress, Tableau not started) and no recommendations (Phase 8). Recruiters for BI Analyst roles will expect to see the dashboard.

**To reach "ready":** (1) licence-safe public version, (2) Power BI report built, with every value on its validation page matching, (3) a short insights and recommendations section.
