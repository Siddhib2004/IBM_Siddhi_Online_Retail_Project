# 📊 Online Retail Business Intelligence & Analytics

An end-to-end **Business Intelligence and Analytics project** built with Python and Streamlit using the **UCI Online Retail dataset**.

The application follows:

**Raw Data → Data Cleaning → KPI Calculation → Descriptive Analytics → Diagnostic Analytics → Predictive Analytics → Prescriptive Analytics → Business Insights**

The main application is implemented in [`project.py`](project.py).

## 📌 Project Overview

This project analyzes online retail transactions to understand:

- Revenue, orders, customers, products and units sold
- Average Order Value (AOV)
- Monthly revenue and order trends
- Product and country performance
- Cancellation rate
- Product and customer revenue concentration
- Relationships between quantity, unit price and revenue
- Three-month revenue forecasts using Random Forest
- Data-driven business recommendations

## ✨ Key Features

### 1. Executive Dashboard

The Streamlit dashboard provides:

- Total Revenue
- Total Orders
- Total Customers
- Total Products
- Average Order Value
- Monthly Revenue Trend
- Top 10 Products by Revenue
- Top 10 Countries by Revenue
- Monthly Orders
- Key Business Findings

### 2. Data Quality & Cleaning

The application:

- Removes empty rows and duplicates
- Converts `InvoiceDate` to datetime
- Converts `Quantity`, `UnitPrice` and `CustomerID` to numeric values
- Trims text fields
- Identifies cancellations when `InvoiceNo` starts with `C`
- Removes invalid records from sales analysis
- Keeps positive, non-cancelled transactions for revenue analysis
- Creates `Revenue = Quantity × UnitPrice`

Time features created:

`Year`, `Month`, `MonthName`, `YearMonth`, `Day`, `DayName`, `Hour`

The cleaned dataset can be downloaded from the dashboard.

## 📈 Descriptive Analytics

Answers **“What happened?”**

Includes:

- Numerical summaries
- Revenue and quantity distributions
- Monthly revenue
- Monthly orders
- Average Order Value
- Revenue growth
- Revenue by day of week
- Revenue by hour
- Revenue and AOV trends

## 🔎 Diagnostic Analytics

Answers **“Why did it happen?”**

### Cancellation Rate

```text
Cancelled Unique Invoices / Total Unique Invoices × 100
```

### Product Concentration

Calculates cumulative product revenue and the revenue contribution of the top 20% of products.

### Customer Concentration

Calculates the revenue contribution of the top 10 customers.

### Country Analysis

Ranks countries by revenue, orders and customers.

### Correlation Analysis

Examines relationships between:

- Quantity
- Unit Price
- Revenue

> Correlation indicates association and does not prove causation.

## 🤖 Predictive Analytics

The project forecasts the next **3 months of revenue** using `RandomForestRegressor`.

### Model Features

- Year
- Month
- 1-month revenue lag
- 2-month revenue lag
- 3-month revenue lag
- 3-month rolling mean

### Model Configuration

```text
n_estimators = 300
random_state = 42
max_depth = 8
min_samples_leaf = 2
```

A time-based split uses the earliest observations for training and the latest 20% for testing.

Evaluation metrics:

- MAE — Mean Absolute Error
- RMSE — Root Mean Squared Error
- R² — R-squared

Future predictions are generated recursively and constrained to non-negative revenue.

## 💡 Prescriptive Analytics

Recommendations are generated dynamically from the current dataset and may address:

- High-revenue product inventory and visibility
- Low-performing products
- Country-level retention/localization
- Cancellation risk
- Customer concentration
- Recent revenue growth or decline
- Forecast-based planning

## 🖥️ Streamlit Dashboard

Navigation pages:

1. **Executive Dashboard**
2. **Data Quality & Cleaning**
3. **Descriptive Analysis**
4. **Diagnostic Analysis**
5. **Predictive Analysis**
6. **Prescriptive Analysis**
7. **Detailed Tables**
8. **Project Methodology**

The sidebar also supports uploading an alternative Excel dataset.

## 📂 Project Structure

```text
IBM_Siddhi_Online_Retail_Project/
│
├── project.py
├── Online Retail.xlsx
├── requirements.txt
├── README.md
│
└── outputs/
    ├── cleaned_sales_data.csv
    ├── monthly_kpis.csv
    ├── product_analysis.csv
    ├── country_analysis.csv
    └── revenue_forecast.csv
```

| File | Description |
|---|---|
| `project.py` | Main Streamlit application and analytics pipeline |
| `Online Retail.xlsx` | Input retail transaction dataset |
| `requirements.txt` | Python dependencies |
| `README.md` | Project documentation |
| `outputs/cleaned_sales_data.csv` | Cleaned positive sales transactions |
| `outputs/monthly_kpis.csv` | Monthly KPI metrics |
| `outputs/product_analysis.csv` | Product-level analysis |
| `outputs/country_analysis.csv` | Country-level analysis |
| `outputs/revenue_forecast.csv` | Actual and forecast revenue |

## 🧹 Data Processing Flow

```text
Load Excel File
      ↓
Standardize Columns
      ↓
Remove Empty Rows
      ↓
Remove Duplicates
      ↓
Convert Data Types
      ↓
Clean Text Fields
      ↓
Identify Cancellations
      ↓
Validate Transactions
      ↓
Keep Positive Sales
      ↓
Calculate Revenue
      ↓
Create Time Features
      ↓
Calculate KPIs
      ↓
Generate Analytical Tables
      ↓
Run Diagnostic Analysis
      ↓
Train Random Forest Model
      ↓
Generate 3-Month Forecast
      ↓
Generate Recommendations
      ↓
Export CSV Files
      ↓
Display Streamlit Dashboard
```

## 📊 KPI Calculations

| KPI | Calculation |
|---|---|
| Revenue | Sum of transaction revenue |
| Orders | Number of unique invoice numbers |
| Customers | Number of unique customer IDs |
| Products | Number of unique stock codes |
| Units Sold | Sum of quantity |
| Average Order Value | Revenue / Orders |
| Units per Order | Units Sold / Orders |
| Revenue per Customer | Revenue / Customers |

## 📤 Generated Outputs

The application writes these files to `outputs/`:

### `cleaned_sales_data.csv`
Cleaned, positive, non-cancelled transactions with engineered fields.

### `monthly_kpis.csv`
Monthly revenue, orders, customers, units, AOV and revenue growth.

### `product_analysis.csv`
Product-level revenue, units and orders.

### `country_analysis.csv`
Country-level revenue, orders and customers.

### `revenue_forecast.csv`
Historical actual revenue plus three forecast periods.

## 🚀 Installation

### 1. Clone the repository

```bash
git clone https://github.com/Siddhib2004/IBM_Siddhi_Online_Retail_Project.git
cd IBM_Siddhi_Online_Retail_Project
```

### 2. Create a virtual environment

**Windows**

```bash
python -m venv venv
venv\Scripts\activate
```

**macOS / Linux**

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

## ▶️ Run the Application

```bash
streamlit run project.py
```

Then open:

```text
http://localhost:8501
```

## 📁 Dataset

The project uses the **UCI Online Retail dataset**.

The default input filename is:

```text
Online Retail.xlsx
```

The application expects these columns:

```text
InvoiceNo
StockCode
Description
Quantity
InvoiceDate
UnitPrice
CustomerID
Country
```

If the default Excel file is unavailable, an Excel file can be uploaded from the Streamlit sidebar.

## 🛠️ Technologies Used

- **Python**
- **Pandas**
- **NumPy**
- **Plotly**
- **Streamlit**
- **Scikit-learn**
- **OpenPyXL**

| Library | Purpose |
|---|---|
| Pandas | Data cleaning, transformation and aggregation |
| NumPy | Numerical calculations |
| Plotly | Interactive visualizations |
| Streamlit | Interactive dashboard |
| Scikit-learn | Random Forest forecasting and evaluation |
| OpenPyXL | Excel file reading |

## 🧠 Main Python Functions

### `load_raw_data(file_path)`
Loads the Excel dataset with Pandas.

### `clean_data(df)`
Cleans transactions, identifies cancellations, keeps valid positive sales and creates engineered features.

### `calculate_kpis(sales)`
Calculates the main business KPIs.

### `monthly_metrics(sales)`
Creates monthly revenue, orders, customers, units, AOV and growth metrics.

### `create_forecast(monthly, horizon=3)`
Builds and evaluates the Random Forest model and produces recursive future forecasts.

### `diagnostic_analysis(sales, raw_all)`
Calculates cancellation rate, product concentration, country concentration, customer concentration and correlations.

### `generate_recommendations(...)`
Creates business recommendations from the current analytical results.

### `save_outputs(...)`
Exports the analytical datasets to CSV.

## ⚠️ Important Considerations

- The forecast uses historical monthly revenue and lag-based features.
- Promotions, holidays, inventory constraints, external economic factors and future pricing are not explicitly modeled.
- Forecasts are planning estimates, not guarantees.
- Correlation does not establish causation.
- Results depend on the quality and coverage of the input dataset.

## 🔮 Future Enhancements

Potential extensions include:

- Customer RFM segmentation
- Customer cohort and retention analysis
- Product category analysis
- Market basket analysis
- Product recommendation system
- Seasonal forecasting
- Comparison of multiple forecasting models
- Forecast confidence intervals
- Interactive filters and drill-downs
- Automated model monitoring
- Cloud deployment
- Authentication and role-based access
- Automated reporting

## 🎯 Project Objective

The project demonstrates how raw retail transaction data can be transformed into actionable business intelligence using:

```text
Data Engineering
        +
Business Intelligence
        +
Data Visualization
        +
Diagnostic Analytics
        +
Machine Learning
        +
Prescriptive Analytics
```

It provides an end-to-end example of applying Python-based analytics to a real-world retail business problem.

## 👩‍💻 Author

**Siddhi Bhavsar**

GitHub: https://github.com/Siddhib2004

## 📜 License

This project is intended for educational, internship and portfolio purposes.

## 🙏 Acknowledgements

- UCI Machine Learning Repository for the Online Retail dataset
- Python open-source data science ecosystem
- Streamlit for the interactive dashboard framework
- Scikit-learn for machine learning functionality
