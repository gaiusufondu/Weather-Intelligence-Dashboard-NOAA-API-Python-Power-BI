# Weather-Intelligence-Dashboard-NOAA-API-Python-Power-BI

# Project Overview
A full analytics pipeline that extracts five years of daily weather data for Chicago, Los Angeles, and New York from the NOAA Climate Data Online API, cleans and engineers business-ready features in Python, and surfaces insights through a three-page interactive Power BI dashboard.
Tech Stack
•	Python 3.11 — data extraction, cleaning, feature engineering
•	Pandas — data manipulation and transformation
•	Requests — NOAA API calls with rate-limit handling
•	Jupyter Notebook — development environment
•	Power BI Desktop (December 2024) — dashboard and DAX measures
•	Power Query — data model transformations

## Data Source
NOAA Climate Data Online (CDO) — Daily Summaries
•	Endpoint: https://www.ncei.noaa.gov/access/services/data/v1
•	Station IDs: USW00094728 (NYC), USW00023174 (LA), USW00094846 (Chicago)
•	Date range: 2020-01-01 to 2024-12-31
•	Fields: TMAX, TMIN, PRCP, SNOW, AWND, WSF2
Requires a free NOAA API token — register at https://www.ncdc.noaa.gov/cdo-web/token

Repository Structure
weather-intelligence/
├── notebooks/
│   └── weather_data_extraction_and_cleaning.ipynb
├── data/
│   ├── weather_data_raw.csv
│   └── weather_data_complete_2020-01-01_2024-12-31.csv
├── dashboard/
│   └── Weather-Intelligence-Dashboard-preview.jpg
├── report/
│   └── Capstone_Report_Gaius_Ufondu.pdf
└── README.md

## Data Pipeline
1. Extraction — API calls with 2-second rate-limit delays, incremental saving, token validation
2. Cleaning — city-aware missing value imputation, temperature consistency validation, date continuity check
3. Feature Engineering — 5 derived columns: Tourism Comfort Index, Heating/Cooling Degree Days, Agriculture Score, Weather Event Classification, Temporal flags
4. Modeling — star schema in Power BI (fact table + date dimension), 12 DAX measures
5. Visualization — 21 visuals across 3 dashboard pages

## The Data Problem I Set for Myself
Weather data is everywhere, but it is usually either too granular to be useful or too aggregated to be interesting. I wanted something in between — daily records across multiple cities, over enough years to reveal real patterns.
I settled on three cities that tell genuinely different climate stories: Chicago (extreme and volatile), Los Angeles (stable and dry), and New York (somewhere in the middle, with worse storms). The time window was January 2020 through December 2024. Five years, three cities, daily readings. That is 5,481 records before I even open the dataset.
Building the Pipeline: NOAA API + Python
I registered for a free API token with NOAA's Climate Data Online service and hit their daily-summaries endpoint. Each city has a permanent station ID — USW00094728 for Central Park in New York, USW00023174 for Downtown Los Angeles, USW00094846 for O'Hare in Chicago.
The Python logic was not complicated, but getting it right required care. NOAA rate-limits aggressive scrapers, so I built in 2-second delays between calls. I also implemented token validation with a re-authentication prompt, because tokens expire without warning and you do not want to discover that mid-run on a 5-year pull.
def fetch_noaa_data(station_id, start_date, end_date):
    params = {"dataset": "daily-summaries", "stations": station_id,
              "startDate": start_date, "endDate": end_date, "format": "json"}
    response = requests.get(API_URL, params=params)
    return pd.DataFrame(response.json())
The raw output had six fields per row: max temp, min temp, precipitation, snowfall, average wind speed, and peak wind speed. That is the skeleton. The rest I built.
Cleaning: The Part Nobody Talks About
27.1% of the snowfall column was missing. At first that looks like a data quality problem. It is not — Los Angeles simply does not snow, so stations there do not record it. The fix was city-aware: fill LA's snowfall with zero, fill Chicago and New York with the monthly median.
62.7% of the fog indicator was missing. Fog is not consistently recorded across stations, and I documented that limitation rather than pretending the data was clean. What I did not tolerate was any missingness in the core temperature and precipitation fields. Those came out at zero missing values.

The more interesting work was feature engineering. I created five new columns that did not exist in the source data:
•	Tourism Comfort Index (0–100 scale): a composite score weighing temperature, humidity, and precipitation into a single "how pleasant is it to be outside" metric
•	Heating and Cooling Degree Days: standard energy industry measures for estimating seasonal HVAC demand
•	Agriculture Suitability Score (0–10): built from frost-day counts, growing-season temperature ranges, and precipitation regularity
•	Weather Event Classification: mapping raw conditions into Clear, Rainy, Freezing, Snowy, and Hot categories
•	Seasonal and temporal flags: season names, fiscal periods, month names — the kind of labels that make time intelligence in Power BI actually work

The dataset went from 23 columns to 35. Every new column exists for a reason.

## EDA: What the Data Said Before I Touched Power BI
Los Angeles averages 67.3°F across five years, with a standard deviation of 8.2°F. It is boring to live there, weatherwise, and that boring consistency is exactly what the tourism and energy industries care about.
Chicago's range is -5.3°F to 95.1°F. That is a 100-degree swing in a single location across a year. The implications for heating costs, infrastructure planning, and human misery are significant.
All three cities show a warming trend of 0.24°F per year across the study period. Over a five-year window, that is suggestive rather than conclusive. Over 30 years, it becomes alarming.
The strongest correlation I found: maximum temperature vs cooling degree days, at r = 0.89. The weakest meaningful one: wind speed vs precipitation, at r = 0.18. The negative correlation between temperature and precipitation (r = -0.42) tracks with intuition — hot days tend to be dry days, especially in LA.

## The Dashboard: Three Pages, Three Audiences
I structured the Power BI dashboard around a question I kept coming back to: who is actually going to use this? Different stakeholders care about different things, so I built three pages for three contexts.
Page 1 is the executive overview. KPI cards at the top (average temp, total precipitation, comfortable days, extreme weather days), then a 5-year temperature trend, a weather condition distribution donut, and a city comparison column chart. Everything is filterable by city, year, and season. A non-technical decision-maker can land here and leave with five useful numbers.
Page 2 is for the analyst. I ran a correlation heatmap across all numeric variables, used small multiples to compare seasonal patterns across cities, and implemented a Key Influencers visual asking: what drives extreme hot weather events? The answer, predictably, is July in New York and summer in general — but seeing the model surface that through Power BI's AI visual is worth the setup.
Page 3 is applied: tourism, energy, and agriculture in one place. The Monthly Comfort Index table lets you compare every month across all three cities on a single score. The energy demand chart shows heating vs cooling degree days by season. The agriculture gauge gives a suitability score with regional crop recommendations attached.
12 DAX measures power the interactivity, including a Year-over-Year temperature change measure that uses SAMEPERIODLASTYEAR() and handles blank periods cleanly.

## What I Would Do Differently
Three cities is not enough for regional conclusions. I would expand to 10–15 for the next version, and ideally extend the timeframe to 10 years so the warming trend has more statistical weight.
I also want to build a live data feed. The current dashboard is a static snapshot of 2020–2024. Connecting to a streaming NOAA API would turn this into a monitoring tool rather than a historical report — a meaningfully different product.
The Tourism Comfort Index is the output I am most proud of, but it needs validation against actual tourism revenue data to confirm it is measuring what I think it is. The correlation with available data is r = 0.68, which is promising. A holdout test on 2025 data would tell me whether the model holds.

## Tools, Files, and Access
Everything is documented and accessible:
•	Python scraping notebook: the full extraction and cleaning script with comments
•	Cleaned dataset: weather_data_complete_2020-01-01_2024-12-31.csv (5,481 rows × 35 columns)
•	Dashboard preview: screenshots of all three pages
•	Full capstone report: methodology, findings, and recommendations in a 15+ page document
This is the most complete project I have built end-to-end. Not the most technically complex — but the one where I most clearly understood what I was building and why, at every step. That clarity is what I want to carry into the next one.


## Key DAX Measures
-- Year-over-Year Temperature Change
Temp YoY Change =
VAR CurrentAvg = [Avg Temperature]
VAR PreviousAvg = CALCULATE([Avg Temperature], SAMEPERIODLASTYEAR(DateTable[Date]))
RETURN IF(NOT ISBLANK(PreviousAvg), CurrentAvg - PreviousAvg, BLANK())

-- Percentage of Comfortable Days
Percentage Comfortable Days =
DIVIDE([Comfortable Days], COUNTROWS(weather_data), 0)

## Key Findings
•	LA maintains the most stable climate: avg 67.3°F, σ = 8.2°F over 5 years
•	Chicago temperature range: -5.3°F to 95.1°F (100.4°F spread)
•	All three cities show +0.24°F/year warming trend (2020–2024)
•	Tourism Comfort Index peaks in May (TCI=82) and June (TCI=79) across all cities
•	Strong correlation: Max Temp vs Cooling Degree Days, r = 0.89
•	Chicago leads heating demand at 4,820 HDD/year vs LA's 1,840 CDD/year

## Data Quality
•	Raw records: 5,481 rows (3 cities × ~1,827 days)
•	Missing snowfall (27.1%): expected — LA has no snow; filled with 0 for LA, monthly median for others
•	Missing fog indicator (62.7%): inconsistent station recording; documented, not imputed
•	Missing wind speed (1.1%): imputed with monthly city median
•	Post-cleaning: 0% missing in all critical fields

### License
Data: NOAA Climate Data Online — Public Domain
Code: MIT License
Report: All rights reserved — Gaius Onyedikachukwu Ufondu, 2026

