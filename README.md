# Online Retail Business Intelligence & Analytics

End-to-end data analytics project covering:

- Data acquisition
- Data cleaning and quality checks
- Descriptive analytics
- KPI analysis
- Trend analysis
- Diagnostic analytics
- Driver analysis
- Risk and opportunity analysis
- Predictive revenue forecasting
- Prescriptive recommendations
- Interactive Streamlit dashboard

## Dataset

UCI Machine Learning Repository — Online Retail.

Official source:
https://archive.ics.uci.edu/dataset/352/online%2Bretail

Download `Online Retail.xlsx` from the UCI page and place it beside `project.py`.

## Project structure

```text
online-retail-data-analysis/
│
├── project.py
├── requirements.txt
├── README.md
├── Online Retail.xlsx
└── outputs/
    ├── cleaned_sales_data.csv
    ├── monthly_kpis.csv
    ├── product_analysis.csv
    ├── country_analysis.csv
    └── revenue_forecast.csv
```

## Analytics flow

Data → Cleaning → KPIs → Descriptive → Diagnostic → Predictive → Prescriptive → Action

## Predictive model

A Random Forest regression model forecasts the next three months of revenue using:

- year
- month
- previous month revenue
- previous two-month revenue
- previous three-month revenue
- three-month rolling average

The model is evaluated with MAE, RMSE and R².

## Important limitation

The forecast uses only historical transaction data. It does not account for promotions, holidays, inventory constraints, price changes or external economic factors. Treat forecasts as planning estimates.
