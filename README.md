# DC Crime Analysis with Power BI

## Overview

This project focuses on analyzing crime incident data for Washington D.C., covering the period **from 2008 onwards** (dynamically extending as new yearly data becomes available). The goal is to explore long-term trends, identify recurring patterns (temporal, geographic, type-based), and ultimately present key insights through an interactive Power BI dashboard designed for public awareness.

## Data Source

The primary data source is the **Crime Incidents** datasets provided by the **DC Open Data Portal**, accessed via the **MPD ArcGIS MapServer REST API**.

*   **MapServer Service Endpoint:** [https://maps2.dcgis.dc.gov/dcgis/rest/services/FEEDS/MPD/MapServer](https://maps2.dcgis.dc.gov/dcgis/rest/services/FEEDS/MPD/MapServer)
*   **Data Format Requested:** GeoJSON (`f=geojson`) from individual layers.
*   **Individual Layer Access:** Data is organized into layers, typically by year (e.g., Layer `6` for 2024, `5` for 2023, etc.).

## Dynamic Data Retrieval and Processing (Power Query)

A key feature of this project is the **automated discovery and retrieval** of relevant crime data layers directly within Power Query, eliminating the need for manual updates when new yearly datasets are added by DC GIS.

**Challenge:** Direct API calls to individual layers are limited by the server's `MaxRecordCount` (typically 1000 records per request), preventing retrieval of the full dataset for a given year in a single call.

**Automated Solution:**
1.  **Layer Discovery (via JSON):** The main Power Query script first connects to the MapServer's definition endpoint (`MapServerUrl?f=json`). It fetches and parses the JSON response which contains metadata for the entire service, including a list of all layers with their names and IDs.
2.  **Layer Filtering:** This list of layers is filtered to identify relevant ones, typically by checking if the layer `name` contains specific text (e.g., "Crime Incidents"). This dynamically creates the list of Layer IDs to process.
3.  **Iterative Fetching (Function):** A reusable Power Query function (`fnGetDataForLayer`) handles fetching data for a *single specified Layer ID*. This function implements pagination using `List.Generate`, iteratively calling the layer's `/query` endpoint with increasing `resultOffset` values until all records for that layer are retrieved. It includes error handling for individual page requests.
4.  **Invocation and Combination:** The main script iterates through the *dynamically discovered* Layer IDs. For each ID, it invokes the `fnGetDataForLayer` function. The resulting tables (one per layer/year) are then combined into a single master table (`AllCrimeData`).
5.  **Robustness:** This dynamic approach automatically includes new yearly "Crime Incidents" layers as soon as they are added to the MapServer service definition and match the filtering criteria.

**Key Transformation Steps (applied after fetching and combining all data):**
*   Extraction and expansion of GeoJSON `properties` into tabular columns.
*   Conversion of Epoch timestamps to Power BI DateTime types.
*   Setting appropriate column data types.
*   Extraction of Date and Time components.
*   Filtering data to desired date ranges (e.g., 2008 onwards).
*   Rounding `START_TIME` to the nearest 15-minute interval.
*   Removal of intermediate/unnecessary columns.

This ensures the Power BI report fetches the complete, relevant multi-year dataset directly from the source APIs upon refresh, adapting automatically to new data releases.

## Data Model & DAX Measures

The Power BI report utilizes a star schema data model designed to handle the multi-year data effectively.

### Data Model Overview

The core model consists of dimension tables linking to a central fact table:

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
