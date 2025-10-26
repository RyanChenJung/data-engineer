# data-engineer
Final Project
# Global Craft Beer Market Intelligence and Competitive Analysis Platform: Project Proposal

This document outlines the final project proposal for building a modern Data Engineering pipeline and analysis platform focusing on global craft beer product metadata, utilizing Google Cloud Platform (GCP) and dbt for robust ELT implementation.

## I. Executive Summary

| Attribute | Description | Citation |
|---|---|---|
| **Project Core** | Build a data-driven platform for analyzing global craft beer product metadata.[1] | [1] |
| **Data Source** | Static metadata (breweries, beers, styles, location) from `beer.db`.[1] | [1] |
| **Business Value** | Deliver competitive intelligence, market positioning, and product trend insights to brewery owners and marketing managers.[2] | [2] |
| **Core Architecture** | Modern ELT paradigm: Python for E/L, dbt for T executed within GCP MySQL (Cloud SQL).[3, 4] | [3, 4] |
| **Data Model** | Query-optimized **Product Metadata Star Schema**.[5, 6] | [5, 6] |
| **Deliverable** | Tableau dashboards visualizing geographical saturation, style evolution trends, and ABV/IBU competitive benchmarking.[7, 8] | [7, 8] |

## II. Project Context and Business Value Proposition

### A. Project Background and Description

1.  **Dataset Characteristics and Challenges**
    *   **Source:** `beer.db` provides static, open product metadata.[1]
    *   **Format:** Data is often in unstructured pure text, CSV, or loose JSON format.[1]
    *   **Challenge:** Data quality is inconsistent, requiring deep cleansing and standardization, especially for geographical information and style tags.[9, 10]
    *   **Goal:** Establish a robust Staging Layer, execute complex data cleansing, and implement a dimensional model (Star Schema) optimized for BI analysis.[5, 6]

### B. Business Use Case: Market Intelligence and Competitive Benchmarking

1.  **Value Proposition**
    *   **Product Portfolio Strategy:** Identify highly saturated areas and "white spaces" within the ABV (Alcohol by Volume) and IBU (International Bitterness Units) dimensions for specific beer styles.[2, 11]
    *   **Market Entry & Distribution:** Assess competitive intensity and style concentration in specific geographical markets to guide expansion or distribution decisions.[12]
    *   **Competitive Benchmarking:** Quantitatively compare product attributes (e.g., average ABV, IBU range) against competitors to inform R&D and pricing.[13]

### C. Key Stakeholder Analysis and Information Needs

| Stakeholder | Role | Key Concern | Analytical Need |
|---|---|---|---|
| **Brewery Owners** | Executive Level | Expansion, investment, and operational efficiency . | Geographical Saturation Maps, overall market style evolution rate.[14] |
| **Marketing Managers** | Decision Support | Product differentiation, target market setting.[15, 16] | Competitive scatter plots of product attributes (ABV/IBU), style concentration metrics.[11] |
| **Industry Analysts** | External/Investors | Macro market structure and quantitative KPIs. | Product creation timeline analysis (`created_at` field), long-term style trends.[17, 18] |

## III. Data Engineering Architecture and Technology Stack

### A. Data Sources and Extraction Strategy

| Source | Description | Extraction Method |
|---|---|---|
| `beer.db` Core Metadata [1] | Static data (beers, breweries, styles, locations). | Python script (Requests/Pandas) to download CSV/JSON from GitHub repository.[1] |
| External Augmentation Data | Geocoding and Address Resolution. | Python (GeoPy/Usaddress) for address parsing and latitude/longitude conversion.[19, 12] |
| Future Expansion Data | Beer reviews, consumer preferences (e.g., Untappd).[20] | API or Web Scraping (for future expansion).[21] |

### B. Proposed Technology Stack (ELT Paradigm)

| Component | Tool | Key Use and Rationale | Citation |
|---|---|---|---|
| **Database/Warehouse** | **GCP MySQL (Cloud SQL)** | Used as the unified data warehouse (Staging and Production). MySQL is stable and integrates seamlessly into the Google Cloud Platform environment.[17, 4] | [17, 4] |
| **Extract/Load (E/L)** | Python (Pandas) | Handles initial extraction, complex pre-processing (address parsing, Regex) [22], and loading raw data into the MySQL Staging schema. | [19, 22] |
| **Transform (T)** | dbt (Data Build Tool) | Implements all SQL transformation logic (data testing, normalization, dimensional modeling). dbt executes T *inside* GCP MySQL, ensuring version control and governance.[3, 23] | [3] |
| **Visualization (BI)** | Tableau | Used to build interactive, multi-layered dashboards, connecting directly to the dbt-generated Production Schema.[24, 8] | [24, 8] |

### C. Data Flow and Pipeline Design

1.  **Extract (E):** Python scripts pull raw data from GitHub.[1]
2.  **Load (L):** Python loads raw CSV/JSON data into the `stage_beermeta` schema in **GCP MySQL**.[17]
3.  **Transform (T - via dbt):**
    *   dbt connects to GCP MySQL.[3]
    *   Executes sequential SQL models: Normalization (3NF) → Dimensional Modeling (Star Schema).[23]
    *   Production data is stored in the `prod_beermeta` schema in **GCP MySQL**.[17]
4.  **Analysis:** Tableau connects to `prod_beermeta` for BI reporting.[7, 24]

## IV. Data Cleansing and Pre-processing Strategy

### A. Data Quality Challenges and Cleansing

| Challenge Area | Expected Issue | Cleansing Strategy | Tool |
|---|---|---|---|
| **Geographical Data** | Inconsistent addresses, free-text location entries.[1] | Apply Python address parsing (Regex) and GeoPy to convert structured addresses to standard `latitude` and `longitude`.[19] | Python (Pandas/GeoPy) |
| **Product Attributes** | Non-numeric values (e.g., `N/A`) in ABV/IBU fields.[11] | Use dbt transformations to ensure ABV/IBU are standardized numerical types (DECIMAL); explicitly mark missing values as NULL. | dbt (SQL) |
| **Style Classification** | Synonyms, spelling errors, or multiple mixed tags (e.g., `ipa\|india_pale_ale`).[1] | Implement data mapping and lookup tables to unify style tags into a hierarchical `Category` structure for the `DIM_STYLE`.[9] | dbt (SQL) |

### B. Data Normalization Design (3NF in Staging)

Normalization is implemented using dbt SQL models to transform raw data into 3NF, eliminating redundancy prior to dimensional modeling.[17]

| 3NF Intermediate Tables | Core Purpose |
|---|---|
| `N_BREWERY_MASTER` | Unique brewery entities with standardized names and assigned surrogate keys. |
| `N_STYLE_MASTER` | Standardized style dictionary with consistent naming and category hierarchy. |
| `N_BEER_ATTRIBUTES` | Cleaned product attributes ensuring ABV, IBU, etc., are numeric and correctly formatted. |

## V. Core Data Model: Dimensional Star Schema

### A. Dimensional Modeling Rationale

The Star Schema is designed to optimize query performance in Tableau and simplify end-user interaction by separating descriptive context (Dimensions) from quantitative metrics (Facts) .

### B. Dimension Table Specifications (GCP MySQL)

| Dimension Table | Core Attributes | Data Type (MySQL) | Key |
|---|---|---|---|
| `DIM_BREWERY` | `Brewery_SK`, `Name`, `Established_Year`, `Status` | INT, VARCHAR, YEAR, VARCHAR | Primary Key: `Brewery_SK` |
| `DIM_STYLE` | `Style_SK`, `Style_Name`, `Category`, `Subcategory` | INT, VARCHAR, VARCHAR, VARCHAR | Primary Key: `Style_SK` |
| `DIM_LOCATION` | `Location_SK`, `Country`, `Region`, `City`, `Latitude`, `Longitude` | INT, VARCHAR, VARCHAR, VARCHAR, DECIMAL(9,6), DECIMAL(9,6) | Primary Key: `Location_SK` |
| `DIM_DATE` | `Date_SK`, `Date_Value`, `Year`, `Month`, `Quarter` | INT, DATETIME, INT, INT, INT | Primary Key: `Date_SK` |

### C. Fact Table Specifications

#### FACT\_BEER\_METADATA (Product Metadata Fact Table)

This fact table sits at the center of the Star Schema, where each row represents a beer product entity.[25, 26]

| Category | Column Name | Data Type (MySQL) | Description |
|---|---|---|---|
| **Foreign Keys (FK)** | `Brewery_SK` | INT | Links to `DIM_BREWERY` |
| | `Style_SK` | INT | Links to `DIM_STYLE` |
| | `Location_SK` | INT | Links to `DIM_LOCATION` (Brewery location) |
| | `Creation_Date_SK` | INT | Links to `DIM_DATE` (Product release date/year) |
| **Measures** | `ABV` | DECIMAL(4,2) | Alcohol by Volume [11] |
| | `IBU` | INT | International Bitterness Units [11] |
| | `SRM` | INT | Standard Reference Method (Color, if available) |
| | `Is_Organic` | BOOLEAN | Flag for organic status [11] |
| **Detail** | `Beer_Name` | VARCHAR(255) | Beer product name |

## VI. Analytical Roadmap and Dashboard Reporting

### A. Derived Metrics and KPI Proposals

| KPI Type | Proposed KPI | Calculation Logic/Use | Citation |
|---|---|---|---|
| **Market Saturation** | **Geographical Style Saturation Index** | Quantity of specific style beers in a region / Total beers in that region. **Use:** Measures competitor concentration in specific styles.[14, 15] | [14, 15] |
| **Product Efficiency** | **ABV/IBU Density Ratio** | Average ABV of a style / Average IBU of that style. **Use:** Provides R&D teams with a formulation benchmark.[13] | [13] |
| **Innovation & Trend** | **Style Evolution Rate** | Ratio of new styles appearing annually / Styles discontinued. **Use:** Monitors the pace of innovation and style lifecycle.[2] | [2] |

### B. Tableau Dashboard Design Proposals

1.  **Dashboard: Global Brewery Geographical Distribution & Style Saturation**
    *   **Components:** Global Map View (heatmap/bubble chart) showing brewery density. KPI cards highlight top style saturation indices.[12]
    *   **Application:** Identifies high-competition areas and style-underserved markets.[14, 27]

2.  **Dashboard: Product Trends and Attribute Analysis**
    *   **Components:** ABV vs. IBU Scatter Plot (color-coded by style/year) to reveal market positioning. Trend lines tracking average ABV/IBU change over time (using `DIM_DATE`).[17, 11]
    *   **Application:** Guides R&D by identifying "White Spaces" (under-explored ABV/IBU combinations).

3.  **Dashboard: Competitive and Recipe Benchmarking**
    *   **Components:** Box Plots comparing ABV and IBU distributions across different groups of breweries (e.g., Regional vs. Micro).[28] Style category distribution charts for selected competitors.
    *   **Application:** Enables quantifiable comparison of product attributes against market standards for internal KPI setting.[13]

### C. Extension: Future Predictive Modeling

*   **Application:** The structured product attributes (ABV, IBU, style category) can serve as features (X variables) for machine learning models (e.g., using Scikit-learn or Statsmodels [29]).
*   **Goal:** Predict potential consumer rating or popularity of a new beer based on its key attributes (upon integration of external ratings data).[20]
*   **Preparation:** Numerical attributes in the fact table will require statistical normalization (e.g., `StandardScaler`) before model training to optimize algorithm performance.[29, 30]

## VII. Conclusion and Future Outlook

This proposal details a comprehensive data engineering project utilizing Python, dbt, and GCP MySQL to transform static beer metadata into actionable business intelligence. The resulting Star Schema provides a high-performance foundation for BI tools. This project not only fulfills the requirements of a Data Engineer Final Project but also delivers a valuable data product capable of providing data-driven competitive advantages in the highly saturated craft beer market.[2, 15]
