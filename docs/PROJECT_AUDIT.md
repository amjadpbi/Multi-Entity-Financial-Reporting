# Project Audit: Multi Location Financial Report

## Scope and caveat

This audit is limited to files present in the project folder and the Power BI project metadata itself. It does not rely on assumptions about a client relationship or production deployment. Where project history is not directly supported by evidence in the files, it is labeled as unverified or creator-provided context.

## 1) Business problem represented by the project

The project is not a funnel or customer-journey exercise. Based on the tables, mappings, and report pages, the business problem is a multi-entity financial reporting and consolidation exercise for a business operating across multiple entities and countries.

Observed evidence from the model and report:
- Source files are named for entities and time periods such as `UT P&L 2026.xls`, `UT Balance Sheet FY 2026.xls`, `US Profit+and+Loss - FY 2026.xlsx`, and `Budget FY 2026.xlsx`.
- The model includes tables such as `UT & US Data 2025-26`, `Budget FY26`, `Map`, `PnL BS & CFS Main`, `Consolidated TBalance`, and `BS HL`.
- The report pages are named for financial statements and consolidation views: `Actual vs Budget by Entity`, `P&L (EBITDA High Level)`, `Balance Sheet (High Level)`, `Cash Flow Statement`, `Consolidation (Trial Balance)`, `P&L (Detailed)`, `Balance Sheet (Detailed)`, `GP by Product`, `SMA`, and `Graphs`.
- Measures include `Actual`, `Budget`, `MTD Actual`, `MTD Variance`, `YTD Actual`, `YTD Budget`, and `YTD Variance`.

This indicates a business question centered on:
- comparing actual financial results against budget,
- analyzing profit and loss and balance sheet movements by month,
- consolidating data across multiple entities/countries,
- reconciling accounting categories to dashboard mappings,
- and reviewing high-level and detailed financial performance.

## 2) Synthetic data created for the project

The project contains a financial dataset that appears to be a structured learning/demo dataset, not an operational production source. Evidence for this is the file structure and the use of workbook-based mapping tables and CSV exports.

Observed source artifacts:
- `Budget FY 2026.xlsx`
- `US Profit+and+Loss - FY 2025.xlsx`
- `US Profit+and+Loss - FY 2026.xlsx`
- `US Balance+Sheet - FY 2025.xlsx`
- `US Balance+Sheet - FY 2026.xlsx`
- `UT P&L 2025.xls`
- `UT P&L 2026.xls`
- `UT Balance Sheet FY 2025.xls`
- `UT Balance Sheet FY 2026.xls`
- Folder `CSV/` containing export files such as `UT P&L 2026.csv`, `UT P&L 2025.csv`, `US Profit+and+Loss - FY 2026.csv`, `US Balance+Sheet - FY 2026.csv`, and mapping files named `USA Mapping.csv` and `UT Australia Mapping.csv`.

Observed labeling in the data:
- Entities/countries include `USA` and `AUS` / `UT` references.
- Accounts include `GL Account #`, `GL Account`, `Account Description`, `Type`, `Month`, `Year`, `Amount`, and `Country`.
- Mapping files use columns such as `GL Account #`, `Account Description`, `Type`, `OMNITRONICS DECK MAPPING for P&L`, and `Consolidation (Trial Balance)`.
- Mapping values include business-style labels like `Revenue USA`, `Hardware sales`, `Cash on hand`, `Trade receivables`, and other financial statement groupings.

This is consistent with a synthetic or anonymized practice dataset built to resemble a real multi-entity financial model.

## 3) Tables and data structures that exist

The semantic model contains both imported source tables and calculated helper tables.

Imported and structurally relevant tables:
- `UT P&L 2026`
- `UT P&L 2025`
- `UT Balance Sheet FY 2026`
- `UT Balance Sheet FY 2025`
- `US P&L FY 2026`
- `US P&L FY 2025`
- `US Balance Sheet FY 2026`
- `US Balance Sheet FY 2025`
- `UT & US Data 2025-26`
- `Budget FY26`
- `Map`
- `Balance Sheet`
- `P&L`
- `P&L (2)`
- `BS`
- `PnL BS & CFS Main`
- `PnL Detailed`
- `Consolidated TBalance`
- `BS HL`

Calculated helper tables:
- `MONTH` (manual date sequence with `INDEX` and `MONTH` values from July to June)
- `BS HL` = crossjoin of distinct `BS HL` values and distinct month values
- `Consolidated TBalance` = crossjoin of distinct mapping combinations and month values
- `DateTableTemplate_c0f96948-9800-490f-9a6a-c939565107f4` (Power BI auto date table)

Key schema patterns:
- `UT & US Data 2025-26` has columns like `GL Code`, `GL Account`, `Type`, `Month`, `Year`, `Amount`, `Country`, `Date`, `BS HL`, `Cash Flow`, `GP Product`, and `PnL`.
- `Budget FY26` is a row-based budget table with `Entity`, `CC`, `PROFIT & LOSS`, `Deck Mapping`, `FY 2026 BUDGET`, `Date`, `Amount`, and `Month`.
- `Map` is a category mapping table that translates GL accounts to higher-level P&L/BS/cash-flow categories by country and type.
- `PnL BS & CFS Main` is a central financial category table where `Actual` and `Budget` are joined by `Mapping` and `MONTH`.

## 4) How the data flows into Power BI

The flow is clear in the TMDL model and Power Query M definitions.

1. Source raw files are read directly from local folders, including Excel and CSV files.
   - Example evidence from `Budget FY26.tmdl`: `Excel.Workbook(File.Contents("D:\Power BI\Data 1 Financial\Budget FY 2026.xlsx"), null, true)`
   - Example evidence from `P&L.tmdl` and `BS.tmdl`: direct `Excel.Workbook(File.Contents(...))` reads from local machine paths.

2. Multiple source sheets are combined or normalized.
   - `UT & US Data 2025-26` source combines `UT P&L 2026 a`, `UT P&L 2025`, `UT Balance Sheet FY 2026`, `UT Balance Sheet FY 2025`, `US P&L FY 2026 a`, `US P&L FY 2025`, `US Balance Sheet FY 2026`, and `US Balance Sheet FY 2025`.
   - `Map` combines multiple mapping tables (`Map_`, `Balance Sheet`, `P&L`, `P&L (2)`, `BS`).

3. Transformation steps include:
   - reading Excel sheets,
   - promoting headers,
   - changing data types,
   - unpivoting month columns,
   - renaming columns,
   - replacing values,
   - adding date and month fields,
   - removing blank columns,
   - adding mapping columns and categories.

4. The model then creates analytical tables and relationships to support financial reporting.
   - `MONTH` is a manual calendar-like dimension to support month order.
   - `UT & US Data 2025-26` is related to `MONTH` via month name and to a date table by the `Date` column.
   - `PnL BS & CFS Main` is related to `MONTH` and to a date table using the generated `Date` column.
   - `Budget FY26` is also related to a date table via `Date`.
   - `Map` is used as a lookup reference table for GL account-to-category mapping.

## 5) Transformations performed

The model has substantial Power Query M logic. The clearly observed transformations are:

- Excel import and sheet extraction from `*.xlsx` and `*.xls` files.
- Header promotion and repeated header cleanup.
- Column type conversion with `Int64.Type`, `Currency.Type`, `type date`, and `type number`.
- Unpivoting monthly columns such as `01/07/2025`, `01/08/2025`, ..., `01/06/2026` into a normalized `Date`/`Value` structure.
- Normalizing month naming to display value like `January`, `February`, etc.
- Replacing text values such as `PROFIT & LOSS` -> `P&L`.
- Adding custom columns for `Date`, `Month`, `Country`, `Type.1`, `BS HL`, `Cash Flow`, `GP Product`, and `PnL`.
- Removing blank/error rows and extraneous columns.
- Crossjoining dimension values to create analytical tables for financial categories by month.
- Creating `Actual` and `Budget` logic using `SUM` with `FILTER`, `LOOKUPVALUE`, and `CALCULATE` in DAX.

Notable modeling behaviors:
- The `MONTH` table is not a standard calendar table; it is a custom fiscal-like month ordering table beginning with July and ending June.
- The table definitions show deliberate fiscal-year modeling and repeated account category mapping across `Map`, `PnL BS & CFS Main`, `BS HL`, and `Consolidated TBalance`.

## 6) Semantic model created

The semantic model is a multi-table financial reporting model designed around a fact-like raw GL data table and a set of mapping / category tables.

Observed model design:
- `UT & US Data 2025-26` is the main fact table for raw ledger activity.
- `Map` acts as a master mapping layer for account/category translation.
- `Budget FY26` is a separate budget fact table.
- `MONTH` is a month dimension for order control.
- `PnL BS & CFS Main` is the central analytical table used to summarize mapped financial data by month and category.
- `BS HL` and `Consolidated TBalance` are calculated category-by-month tables for financial statement rollups.

Model relationships:
- `UT & US Data 2025-26`.Month -> `MONTH`.MONTH
- `UT & US Data 2025-26`.Date -> local date table
- `Map`.`GL Account #` -> `BS`.`GL Account #`
- `PnL BS & CFS Main`.Date -> local date table
- `Budget FY26`.Date -> local date table
- `Consolidated TBalance`.MONTH -> `MONTH`.MONTH
- `PnL BS & CFS Main`.MONTH -> `MONTH`.MONTH
- `PnL Detailed`.MONTH -> `MONTH`.MONTH
- `BS HL`.MONTH -> `MONTH`.MONTH

The model is therefore a modern Power BI star/bridge pattern with financial mappings and period dimensions, but it is not a pure star schema because several helper tables and calculated crossjoins are used to support category rollups and financial statement structures.

## 7) DAX and measures created

The ` _Measuers ` table contains the main report metrics. Observed measures include:

- `Actual` = `SUM('PnL BS & CFS Main'[Actual Final])`
- `Budget` = `SUM('PnL BS & CFS Main'[Budget Actual])`
- `MTD Actual` = actual value at the maximum date present in `UT & US Data 2025-26`
- `Budget MTD` = budget value at the same maximum date
- `MTD Variance` = `MTD Actual - Budget MTD`
- `YTD Actual` = actuals from fiscal start to max date using `DATESBETWEEN`
- `YTD Budget` = budget from fiscal start to max date
- `YTD Variance` = `YTD Actual - YTD Budget`
- `Actual LY` = last-year actual based on `Actual Final LY`
- `YTD Actual LY` = last-year comparison
- `YTD Variance LY` = `YTD Actual - YTD Actual LY`
- `Budget&Actual_Variance` = `YTD Actual - Budget`

Also evident in table definitions:
- `PnL BS & CFS Main` has a calculated `Actual` column that looks up `UT & US Data 2025-26` by month, mapping, and year.
- `PnL BS & CFS Main` has a complex `Actual Final` column using `SWITCH` and `EARLIER` to create final financial statement rollups, including special handling for account indexes like 3, 4, 5, 10, 11, 18, 20.
- `UT & US Data 2025-26` has `BS HL`, `Cash Flow`, `GP Product`, and `PnL` columns using `LOOKUPVALUE` with GL accounts and country/type combinations.
- `MONTH` is a custom seven-to-June fiscal month ordering table used to preserve a non-standard accounting period order.

The DAX is strongly focused on business finance logic: month-to-date, year-to-date, budget versus actual, and year-over-year comparison.

## 8) Analytical questions the report can answer

The report is designed to answer financial-analysis questions such as:

- Which business entities or countries are over or under budget this year?
- How do actual results compare with budget on a month-to-date and year-to-date basis?
- Which P&L and balance-sheet categories are contributing most to variance?
- How do results differ between high-level financial categories and detailed account lines?
- What is the trend of cash flow, trial-balance consolidation, or GP by product?
- How do specific accounts map into consolidated financial categories across country/region data?
- Which months show the largest shifts from actual to budget or year-over-year actuals?
- How does detailed P&L information reconcile to a higher-level summary view?

These questions match the report pages and measure logic.

## 9) What the report actually contains

The report contains 10 pages in the page order list, with the following page names:

1. `Actual vs Budget by Entity`
2. `P&L (EBITDA High Level)`
3. `Balance Sheet (High Level)`
4. `Cash Flow Statement`
5. `Consolidation (Trial Balance)`
6. `P&L (Detailed)`
7. `Balance Sheet (Detailed)`
8. `GP by Product`
9. `SMA`
10. `Graphs`

The page definitions confirm these are standard Power BI report pages, not just placeholder pages. They are not the kind of interactive customer funnel pages seen in other project examples; they are finance-focused reporting pages with the expected financial-reporting layout logic.

The report theme is the standard Power BI base theme `CY25SU11`, and it uses report-level settings such as `useEnhancedTooltips`, `defaultDrillFilterOtherVisuals`, and `AllowSummarized` export.

## 10) Power BI skills this project demonstrates

This project demonstrates several relevant Power BI skill areas:

- PBIP / report + semantic model project structure
- Excel and CSV ingestion using Power Query
- Unpivoting and normalization of financial data
- Mapping tables and account category reconciliation
- Relationship design across fact tables and date/month dimensions
- Use of calculated tables and crossjoins for structured financial rollups
- DAX measures for actual-vs-budget and YTD/MTD variance logic
- Date manipulation and fiscal-period modeling
- Multi-entity, multi-country financial model design
- Drill-through and multi-page financial reporting
- Use of Power BI report pages for statement-style reporting and consolidation views

This is a strong example of a Power BI financial-modeling project, especially around consolidation and variance analysis.

## 11) What is directly verified versus creator-provided history

### Directly verified from project files
- The project contains local financial datasets, mapping tables, and Power BI model/report metadata.
- The report pages and semantic model tables are present and named as shown above.
- The model uses actual Excel/CSV inputs and TMDL definitions.
- The core business domain is multi-entity financial reporting with actual-vs-budget and statements.
- There are calculated tables, time logic, mapping logic, and variance measures in the model.

### Creator-provided / unverified history
- Any claim that this was a real client engagement, production deployment, or a formal corporate implementation is not directly supported by the files in the project folder.
- Statements such as “anonymized client data” or “course cohort project” may be plausible but are not directly evidenced by the project artifacts themselves.
- The file names and mapping table names suggest a learning/demo or synthetic-finance dataset designed to resemble a real reporting scenario, but not a verified live business case.

### Best conservative characterization
- This appears to be a practice or training project that models a multi-entity finance and consolidation scenario using structured synthetic/anonymized company data.
- The project is credible as a Power BI learning artifact, but the evidence does not justify presenting it as a live client deliverable.

## 12) Overall assessment

This project is a well-structured Power BI financial model covering multi-entity actuals, budgets, statement rollups, account mapping, and consolidation logic. The evidence in the project files is strong: the data sources, transformations, measures, and final report pages all align around a coherent finance narrative.

The most accurate summary is:
- It is a finance dashboard project built for learning/practice.
- It uses structured, realistic-looking financial data and mappings.
- It is focused on multi-entity financial consolidation and variance analysis.
- It demonstrates substantial Power BI modeling and DAX skill.
- It should be presented honestly as a portfolio/learning project, not as a verified client implementation.

## Bottom line

This project is best understood as a synthetic or anonymized multi-entity financial reporting model created for learning and demonstration. The evidence strongly supports a finance/GL reporting scenario about actuals, budget, P&L, balance sheet, cash flow, and consolidation across countries/entities.
