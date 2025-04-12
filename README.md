# DC Crime Analysis with Power BI

## Overview

This project focuses on analyzing crime incident data for Washington D.C., covering the period **from 2019 onwards**. The goal is to explore long-term trends, identify recurring patterns (temporal, geographic, type-based), and ultimately present key insights through an interactive Power BI dashboard designed for public awareness.

## Data Source

The primary data source is the **Crime Incidents** datasets provided by the **DC Open Data Portal**. Data is typically available on a yearly basis.

*   **Example Dataset Homepage (2024):** [https://opendata.dc.gov/datasets/DCGIS::crime-incidents-in-2024/explore](https://opendata.dc.gov/datasets/DCGIS::crime-incidents-in-2024/explore) *(Similar datasets exist for prior years, e.g., 2019, 2020, 2021, 2022, 2023)*
*   **API Endpoints Used:** GeoJSON feeds from the corresponding ArcGIS MapServer layers (e.g., `/6` for 2024, other layer IDs for previous years) or potentially a consolidated endpoint if available.
    *   Example (2024): `https://maps2.dcgis.dc.gov/dcgis/rest/services/FEEDS/MPD/MapServer/6/query`

## Data Acquisition & Processing (Power Query)

A significant challenge encountered was retrieving the complete dataset via the GeoJSON API endpoints. Direct API calls are limited by the server's `MaxRecordCount` (e.g., 1000 records per request).

**Solution:** Power Query was used to implement **pagination** directly within Power BI for each relevant API endpoint:
1.  The `List.Generate` function iteratively calls the specific API endpoint (for a given year or data segment).
2.  Each call requests a batch of records (e.g., 1000) using the `resultOffset` and `resultRecordCount` parameters.
3.  The loop continues until all records for that segment are retrieved.
4.  The features from all pages for that segment are combined.
5.  *(Repeat for other years/endpoints if necessary)* Data from different years/sources is appended together.

**Key Transformation Steps (applied after fetching and combining all data):**
*   Extraction of `features` list from the GeoJSON structure.
*   Expansion of `properties` records into tabular columns.
*   Conversion of Epoch timestamp columns (`REPORT_DAT`, `START_DATE`, `END_DATE`) to Power BI DateTime types.
*   Setting appropriate data types for various columns (Text, Whole Number, Decimal Number).
*   Extraction of separate Date and Time components.
*   Rounding `START_TIME` to the nearest 15-minute interval.
*   Removal of intermediate and original Epoch timestamp columns.

This approach allows the Power BI report to fetch the complete, multi-year dataset directly from the source APIs upon refresh.

## Data Model & DAX Measures

The Power BI report utilizes a data model designed to handle the multi-year data effectively:
*   A central Fact table: `CrimeIncidents` (containing the combined and processed data from 2019 onwards)
*   Dimension tables: `dimDate` (covering the full date range 2019-present), `dimTime` (for time-of-day analysis).

A comprehensive set of DAX measures were developed to facilitate analysis across the entire timeframe, grouped into categories:

1.  **Core Metrics:** Total Incidents, MTD/QTD/YTD calculations, Rolling Period counts.
2.  **Temporal Intelligence:** YoY calculations, Day vs. Night ratios, Peak Hour identification, Weekend vs. Weekday analysis, Seasonality Index.
3.  **Geographic Analysis:** Incidents by Ward/District/PSA/Neighborhood, Geographic Concentration, Location Density.
4.  **Crime Type Analysis:** Incidents by Offense/Method, Most Common Offense identification, Method-Offense Correlation.
5.  **Advanced Statistical Measures:** Moving Averages, Standard Deviation, Z-Score (for anomaly detection), Anomaly Flag.
6.  **Time Pattern Recognition:** Hour Pattern Similarity (comparing daily patterns), Day of Week Index (cyclical analysis).
7.  **Predictive Measures:** Simple Linear Trend forecast.
8.  **Crime Severity and Impact Measures:** Example Weighted Crime Index, Average Response Time (requires additional data), Clearance Rate placeholder.
9.  **Comparative Analysis:** Same Month Last Year comparison, Month-over-Month Change, Ward Benchmarking.
10. **AI-Enhanced Narrative Measures:** Examples generating dynamic text insights.

*Note: During development, measures were reviewed for efficiency, redundancy, and correct handling of the multi-year context.*

## Power BI Report Concept

The report concept, potentially titled **"D.C. Crime Trends & Insights (2019-Present)"**, aims to:

*   **Target Audience:** Washington D.C. Public, potentially policymakers or analysts.
*   **Goal:** Provide an interactive platform to explore crime trends and patterns over multiple years.
*   **Focus:** Long-term trends, comparisons across years, geographic shifts, and current status within historical context.
*   **Key Visuals (Potential):**
    *   Headline KPI Cards (e.g., Total Incidents YTD, YoY Change %, Most Common Offense Overall).
    *   Line Charts showing `Total Incidents` over time (by Year, Quarter, Month).
    *   Interactive Map showing Incident density (filterable by Year/Date Range).
    *   Bar/Column Charts comparing metrics (Incidents, YoY Change) across Wards, Offenses, or Years.
    *   Time-of-day and Day-of-week pattern charts.
    *   Slicers for `Year`, `Date Range`, `Ward`, `Offense Type`.

*(Placeholder for a screenshot of the report if available)*
<!-- ![Report Screenshot](link_to_screenshot.png) -->

## Setup/Usage

1.  Ensure you have Power BI Desktop installed.
2.  Open the `.pbix` file (if included in the repository).
3.  The report should refresh directly from the DC Open Data API endpoints, handling pagination and combining data from 2019 onwards automatically. Refresh may take longer due to the larger volume of data.