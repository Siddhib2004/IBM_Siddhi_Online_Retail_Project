
import os
import warnings
from pathlib import Path

import numpy as np
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
import streamlit as st

from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score

warnings.filterwarnings("ignore")

# ============================================================
# ONLINE RETAIL BUSINESS INTELLIGENCE & ANALYTICS PROJECT
# Descriptive + Diagnostic + Predictive + Prescriptive Analytics
# Dataset: UCI Online Retail
# ============================================================

st.set_page_config(
    page_title="Online Retail Analytics",
    layout="wide",
)

DATA_FILE = "Online Retail.xlsx"
OUTPUT_DIR = Path("outputs")
OUTPUT_DIR.mkdir(exist_ok=True)


# -----------------------------
# DATA LOADING
# -----------------------------
@st.cache_data(show_spinner=False)
def load_raw_data(file_path):
    df = pd.read_excel(file_path)
    df.columns = [str(c).strip() for c in df.columns]
    return df


# -----------------------------
# DATA CLEANING
# -----------------------------
@st.cache_data(show_spinner=False)
def clean_data(df):
    data = df.copy()

    # Standardize column names
    rename_map = {
        "InvoiceNo": "InvoiceNo",
        "StockCode": "StockCode",
        "Description": "Description",
        "Quantity": "Quantity",
        "InvoiceDate": "InvoiceDate",
        "UnitPrice": "UnitPrice",
        "CustomerID": "CustomerID",
        "Country": "Country",
    }
    data = data.rename(columns=rename_map)

    # Remove completely empty rows
    data = data.dropna(how="all")

    # Remove duplicate rows
    duplicates_before = data.duplicated().sum()
    data = data.drop_duplicates()

    # Convert data types
    data["InvoiceDate"] = pd.to_datetime(data["InvoiceDate"], errors="coerce")
    data["Quantity"] = pd.to_numeric(data["Quantity"], errors="coerce")
    data["UnitPrice"] = pd.to_numeric(data["UnitPrice"], errors="coerce")
    data["CustomerID"] = pd.to_numeric(data["CustomerID"], errors="coerce")

    # Clean text
    data["Description"] = data["Description"].astype("string").str.strip()
    data["Country"] = data["Country"].astype("string").str.strip()
    data["InvoiceNo"] = data["InvoiceNo"].astype("string").str.strip()
    data["StockCode"] = data["StockCode"].astype("string").str.strip()

    # Cancellation flag
    data["IsCancelled"] = data["InvoiceNo"].str.upper().str.startswith("C").fillna(False)

    # Missing descriptions are not useful for product analysis
    data = data[data["Description"].notna()]

    # Remove invalid transactions from sales analysis
    # Keep cancellations separately for diagnostic analysis.
    data = data[data["InvoiceDate"].notna()]
    data = data[data["Quantity"].notna()]
    data = data[data["UnitPrice"].notna()]

    # Positive sales transactions
    sales = data[
        (~data["IsCancelled"])
        & (data["Quantity"] > 0)
        & (data["UnitPrice"] > 0)
    ].copy()

    # Monetary value
    sales["Revenue"] = sales["Quantity"] * sales["UnitPrice"]

    # Time features
    sales["Year"] = sales["InvoiceDate"].dt.year
    sales["Month"] = sales["InvoiceDate"].dt.month
    sales["MonthName"] = sales["InvoiceDate"].dt.strftime("%b")
    sales["YearMonth"] = sales["InvoiceDate"].dt.to_period("M").astype(str)
    sales["Day"] = sales["InvoiceDate"].dt.day
    sales["DayName"] = sales["InvoiceDate"].dt.day_name()
    sales["Hour"] = sales["InvoiceDate"].dt.hour

    return sales, data, duplicates_before


# -----------------------------
# KPI CALCULATIONS
# -----------------------------
def calculate_kpis(sales):
    revenue = sales["Revenue"].sum()
    orders = sales["InvoiceNo"].nunique()
    customers = sales["CustomerID"].nunique()
    products = sales["StockCode"].nunique()
    units = sales["Quantity"].sum()

    aov = revenue / orders if orders else 0
    units_per_order = units / orders if orders else 0
    revenue_per_customer = revenue / customers if customers else 0

    return {
        "Revenue": revenue,
        "Orders": orders,
        "Customers": customers,
        "Products": products,
        "Units Sold": units,
        "Average Order Value": aov,
        "Units / Order": units_per_order,
        "Revenue / Customer": revenue_per_customer,
    }


# -----------------------------
# MONTHLY DATA
# -----------------------------
def monthly_metrics(sales):
    monthly = (
        sales.groupby("YearMonth")
        .agg(
            Revenue=("Revenue", "sum"),
            Orders=("InvoiceNo", "nunique"),
            Customers=("CustomerID", "nunique"),
            Units=("Quantity", "sum"),
        )
        .reset_index()
    )
    monthly["AOV"] = monthly["Revenue"] / monthly["Orders"].replace(0, np.nan)
    monthly["RevenueGrowthPct"] = monthly["Revenue"].pct_change() * 100
    return monthly


# -----------------------------
# PREDICTIVE MODEL
# Forecast monthly revenue
# -----------------------------
def create_forecast(monthly, horizon=3):
    df = monthly.copy()
    df["Date"] = pd.to_datetime(df["YearMonth"] + "-01")

    # Time features
    df["YearNum"] = df["Date"].dt.year
    df["MonthNum"] = df["Date"].dt.month

    # Lag/rolling features
    for lag in [1, 2, 3]:
        df[f"Lag_{lag}"] = df["Revenue"].shift(lag)

    df["RollingMean3"] = df["Revenue"].shift(1).rolling(3).mean()

    model_df = df.dropna().copy()

    feature_cols = [
        "YearNum",
        "MonthNum",
        "Lag_1",
        "Lag_2",
        "Lag_3",
        "RollingMean3",
    ]

    if len(model_df) < 8:
        return None, None, None, None

    # Time-based split: last 20% for testing
    split = max(1, int(len(model_df) * 0.8))
    train = model_df.iloc[:split]
    test = model_df.iloc[split:]

    model = RandomForestRegressor(
        n_estimators=300,
        random_state=42,
        max_depth=8,
        min_samples_leaf=2,
    )
    model.fit(train[feature_cols], train["Revenue"])

    pred_test = model.predict(test[feature_cols])

    mae = mean_absolute_error(test["Revenue"], pred_test)
    rmse = np.sqrt(mean_squared_error(test["Revenue"], pred_test))
    r2 = r2_score(test["Revenue"], pred_test) if len(test) > 1 else np.nan

    # Recursive future forecast
    history = df[["Date", "Revenue"]].copy()
    future_rows = []

    for _ in range(horizon):
        next_date = history["Date"].max() + pd.offsets.MonthBegin(1)

        lag1 = history["Revenue"].iloc[-1]
        lag2 = history["Revenue"].iloc[-2] if len(history) >= 2 else lag1
        lag3 = history["Revenue"].iloc[-3] if len(history) >= 3 else lag2
        rolling3 = np.mean([lag1, lag2, lag3])

        X_future = pd.DataFrame(
            [{
                "YearNum": next_date.year,
                "MonthNum": next_date.month,
                "Lag_1": lag1,
                "Lag_2": lag2,
                "Lag_3": lag3,
                "RollingMean3": rolling3,
            }]
        )

        forecast_value = max(0, float(model.predict(X_future[feature_cols])[0]))

        future_rows.append(
            {"Date": next_date, "Revenue": forecast_value, "Type": "Forecast"}
        )

        history = pd.concat(
            [history, pd.DataFrame([{"Date": next_date, "Revenue": forecast_value}])],
            ignore_index=True,
        )

    actual = df[["Date", "Revenue"]].copy()
    actual["Type"] = "Actual"

    forecast_df = pd.concat(
        [actual, pd.DataFrame(future_rows)],
        ignore_index=True,
    )

    return forecast_df, model, {"MAE": mae, "RMSE": rmse, "R2": r2}, feature_cols


# -----------------------------
# DIAGNOSTIC ANALYSIS
# -----------------------------
def diagnostic_analysis(sales, raw_all):
    # Cancellation statistics
    total_transactions = raw_all["InvoiceNo"].nunique()
    cancelled_transactions = raw_all.loc[
        raw_all["IsCancelled"], "InvoiceNo"
    ].nunique()

    cancellation_rate = (
        cancelled_transactions / total_transactions * 100
        if total_transactions
        else 0
    )

    # Product concentration
    product_rev = (
        sales.groupby(["StockCode", "Description"])["Revenue"]
        .sum()
        .sort_values(ascending=False)
        .reset_index()
    )
    total_rev = product_rev["Revenue"].sum()
    product_rev["CumPct"] = product_rev["Revenue"].cumsum() / total_rev * 100
    top_20_share = (
        product_rev.head(max(1, int(len(product_rev) * 0.20)))["Revenue"].sum()
        / total_rev
        * 100
        if len(product_rev)
        else 0
    )

    # Country concentration
    country_rev = (
        sales.groupby("Country")["Revenue"]
        .sum()
        .sort_values(ascending=False)
        .reset_index()
    )

    # Customer concentration
    customer_rev = (
        sales.groupby("CustomerID")["Revenue"]
        .sum()
        .sort_values(ascending=False)
        .reset_index()
    )

    top_10_customer_share = (
        customer_rev.head(10)["Revenue"].sum() / total_rev * 100
        if total_rev
        else 0
    )

    # Correlations among business metrics
    corr_df = sales[["Quantity", "UnitPrice", "Revenue"]].corr()

    return {
        "cancellation_rate": cancellation_rate,
        "product_rev": product_rev,
        "country_rev": country_rev,
        "customer_rev": customer_rev,
        "top_20_share": top_20_share,
        "top_10_customer_share": top_10_customer_share,
        "corr": corr_df,
    }


# -----------------------------
# PRESCRIPTIVE RECOMMENDATIONS
# -----------------------------
def generate_recommendations(sales, monthly, diagnostic, forecast_df):
    recommendations = []

    # 1. Product opportunity
    product = (
        sales.groupby(["StockCode", "Description"])
        .agg(Revenue=("Revenue", "sum"), Units=("Quantity", "sum"))
        .sort_values("Revenue", ascending=False)
        .reset_index()
    )

    if len(product):
        top_product = product.iloc[0]
        recommendations.append(
            f"Prioritize inventory and visibility for '{top_product['Description']}' "
            f"because it is the highest-revenue product in the cleaned sales data."
        )

    # 2. Low-performing products
    if len(product) >= 10:
        low_products = product.tail(max(5, int(len(product) * 0.10)))
        recommendations.append(
            f"Review the lowest-revenue product segment ({len(low_products)} products "
            f"shown in the bottom 10% by revenue) for bundling, pricing, or assortment decisions."
        )

    # 3. Country opportunity
    country = diagnostic["country_rev"]
    if len(country) > 1:
        top_country = country.iloc[0]
        second_country = country.iloc[1]
        recommendations.append(
            f"Focus customer-retention and localization efforts on {top_country['Country']}, "
            f"the largest revenue market, while investigating why {second_country['Country']} "
            f"ranks next."
        )

    # 4. Cancellation risk
    cancel_rate = diagnostic["cancellation_rate"]
    if cancel_rate >= 5:
        recommendations.append(
            f"Cancellation rate is {cancel_rate:.2f}%. Investigate cancellation reasons, "
            f"product availability, order-entry issues, and customer-service processes."
        )
    else:
        recommendations.append(
            f"Cancellation rate is {cancel_rate:.2f}%. Continue monitoring cancellations "
            f"by month and product to detect emerging operational issues."
        )

    # 5. Customer concentration
    concentration = diagnostic["top_10_customer_share"]
    if concentration >= 30:
        recommendations.append(
            f"The top 10 customers contribute about {concentration:.1f}% of revenue. "
            "Protect these accounts with retention plans while broadening the customer base."
        )
    else:
        recommendations.append(
            f"The top 10 customers contribute about {concentration:.1f}% of revenue. "
            "Use customer segmentation to identify high-value customers for retention and cross-selling."
        )

    # 6. Trend
    if len(monthly) >= 3:
        recent_growth = monthly["Revenue"].pct_change().iloc[-1] * 100
        if recent_growth < 0:
            recommendations.append(
                f"Latest-month revenue changed by {recent_growth:.1f}%. "
                "Investigate the products, countries, and customer groups responsible for the decline "
                "before planning promotions."
            )
        else:
            recommendations.append(
                f"Latest-month revenue changed by {recent_growth:.1f}%. "
                "Identify which products and markets generated the increase and allocate resources accordingly."
            )

    # 7. Forecast
    if forecast_df is not None:
        future = forecast_df[forecast_df["Type"] == "Forecast"]
        if len(future) >= 1:
            future_avg = future["Revenue"].mean()
            actual_avg = monthly["Revenue"].tail(3).mean()
            change = (future_avg - actual_avg) / actual_avg * 100 if actual_avg else 0
            recommendations.append(
                f"The model's average forecast for the next {len(future)} months is "
                f"{change:+.1f}% versus the latest 3-month average. "
                "Use this as a planning signal rather than a guarantee."
            )

    return recommendations


# -----------------------------
# EXPORTS
# -----------------------------
def save_outputs(sales, monthly, product, country, forecast):
    sales.to_csv(OUTPUT_DIR / "cleaned_sales_data.csv", index=False)
    monthly.to_csv(OUTPUT_DIR / "monthly_kpis.csv", index=False)
    product.to_csv(OUTPUT_DIR / "product_analysis.csv", index=False)
    country.to_csv(OUTPUT_DIR / "country_analysis.csv", index=False)
    if forecast is not None:
        forecast.to_csv(OUTPUT_DIR / "revenue_forecast.csv", index=False)


# ============================================================
# STREAMLIT APPLICATION
# ============================================================

st.title("📊 Online Retail Business Intelligence & Analytics")
st.caption(
    "End-to-end project: Raw Data → Cleaning → Descriptive → Diagnostic → "
    "Predictive → Prescriptive Analytics → Business Action"
)

st.sidebar.header("Project Controls")

uploaded_file = st.sidebar.file_uploader(
    "Upload Online Retail.xlsx",
    type=["xlsx", "xls"],
)

file_to_use = uploaded_file if uploaded_file is not None else DATA_FILE

if uploaded_file is None and not os.path.exists(DATA_FILE):
    st.error(
        f"'{DATA_FILE}' was not found. Download the official UCI Online Retail.xlsx "
        "file and place it in the same folder as project.py."
    )
    st.stop()

try:
    raw_df = load_raw_data(file_to_use)
except Exception as e:
    st.error(f"Could not read the Excel file: {e}")
    st.stop()

sales_df, cleaned_all, duplicate_count = clean_data(raw_df)
kpis = calculate_kpis(sales_df)
monthly = monthly_metrics(sales_df)
diagnostic = diagnostic_analysis(sales_df, cleaned_all)
forecast_df, model, model_metrics, feature_cols = create_forecast(monthly, horizon=3)
recommendations = generate_recommendations(
    sales_df, monthly, diagnostic, forecast_df
)

# Save output tables
product_analysis = (
    sales_df.groupby(["StockCode", "Description"])
    .agg(
        Revenue=("Revenue", "sum"),
        Units=("Quantity", "sum"),
        Orders=("InvoiceNo", "nunique"),
    )
    .sort_values("Revenue", ascending=False)
    .reset_index()
)

country_analysis = (
    sales_df.groupby("Country")
    .agg(
        Revenue=("Revenue", "sum"),
        Orders=("InvoiceNo", "nunique"),
        Customers=("CustomerID", "nunique"),
    )
    .sort_values("Revenue", ascending=False)
    .reset_index()
)

save_outputs(
    sales_df,
    monthly,
    product_analysis,
    country_analysis,
    forecast_df,
)

# Sidebar navigation
page = st.sidebar.radio(
    "Navigate",
    [
        "Executive Dashboard",
        "Data Quality & Cleaning",
        "Descriptive Analysis",
        "Diagnostic Analysis",
        "Predictive Analysis",
        "Prescriptive Analysis",
        "Detailed Tables",
        "Project Methodology",
    ],
)

# ============================================================
# EXECUTIVE DASHBOARD
# ============================================================
if page == "Executive Dashboard":
    st.header("1. Executive Dashboard")

    cols = st.columns(5)
    cols[0].metric("Revenue", f"£{kpis['Revenue']:,.0f}")
    cols[1].metric("Orders", f"{kpis['Orders']:,}")
    cols[2].metric("Customers", f"{kpis['Customers']:,}")
    cols[3].metric("Products", f"{kpis['Products']:,}")
    cols[4].metric("AOV", f"£{kpis['Average Order Value']:,.2f}")

    st.divider()

    c1, c2 = st.columns(2)

    with c1:
        fig = px.line(
            monthly,
            x="YearMonth",
            y="Revenue",
            markers=True,
            title="Monthly Revenue Trend",
        )
        fig.update_layout(xaxis_title="Month", yaxis_title="Revenue (£)")
        st.plotly_chart(fig, use_container_width=True)

    with c2:
        top10 = product_analysis.head(10).sort_values("Revenue")
        fig = px.bar(
            top10,
            x="Revenue",
            y="Description",
            orientation="h",
            title="Top 10 Products by Revenue",
        )
        st.plotly_chart(fig, use_container_width=True)

    c3, c4 = st.columns(2)

    with c3:
        top_country = country_analysis.head(10).sort_values("Revenue")
        fig = px.bar(
            top_country,
            x="Revenue",
            y="Country",
            orientation="h",
            title="Top 10 Countries by Revenue",
        )
        st.plotly_chart(fig, use_container_width=True)

    with c4:
        fig = px.bar(
            monthly,
            x="YearMonth",
            y="Orders",
            title="Monthly Orders",
        )
        st.plotly_chart(fig, use_container_width=True)

    st.subheader("Key Business Findings")
    for item in recommendations[:5]:
        st.write("•", item)


# ============================================================
# DATA QUALITY
# ============================================================
elif page == "Data Quality & Cleaning":
    st.header("2. Data Quality & Cleaning")

    st.write("### Raw Dataset")
    st.write(f"Rows: **{len(raw_df):,}** | Columns: **{raw_df.shape[1]}**")

    quality = pd.DataFrame({
        "Column": raw_df.columns,
        "Data Type": [str(raw_df[c].dtype) for c in raw_df.columns],
        "Missing Values": [int(raw_df[c].isna().sum()) for c in raw_df.columns],
        "Missing %": [
            round(raw_df[c].isna().mean() * 100, 2)
            for c in raw_df.columns
        ],
        "Unique Values": [raw_df[c].nunique(dropna=True) for c in raw_df.columns],
    })

    st.dataframe(quality, use_container_width=True)

    st.metric("Duplicate Rows Removed", f"{duplicate_count:,}")

    st.write("### Cleaning rules applied")
    rules = [
        "Removed completely empty rows.",
        "Removed duplicate records.",
        "Converted InvoiceDate to datetime.",
        "Converted Quantity and UnitPrice to numeric.",
        "Trimmed text fields.",
        "Identified cancellation invoices using InvoiceNo starting with C.",
        "Removed records without valid date, quantity, unit price, or description.",
        "Removed cancelled transactions and non-positive quantity/price from revenue analysis.",
        "Created Revenue = Quantity × UnitPrice.",
        "Created Year, Month, YearMonth, Day, DayName, and Hour features.",
    ]
    for rule in rules:
        st.write("✓", rule)

    st.write("### Cleaned Sales Data Sample")
    st.dataframe(sales_df.head(100), use_container_width=True)

    csv = sales_df.to_csv(index=False).encode("utf-8")
    st.download_button(
        "⬇ Download Cleaned Sales CSV",
        csv,
        "cleaned_sales_data.csv",
        "text/csv",
    )


# ============================================================
# DESCRIPTIVE ANALYSIS
# ============================================================
elif page == "Descriptive Analysis":
    st.header("3. Descriptive Analysis — What Happened?")

    st.subheader("Numerical Summary")
    st.dataframe(sales_df[["Quantity", "UnitPrice", "Revenue"]].describe().T)

    c1, c2 = st.columns(2)

    with c1:
        fig = px.histogram(
            sales_df,
            x="Revenue",
            nbins=50,
            title="Revenue Distribution per Line Item",
        )
        st.plotly_chart(fig, use_container_width=True)

    with c2:
        fig = px.histogram(
            sales_df,
            x="Quantity",
            nbins=50,
            title="Quantity Distribution",
        )
        st.plotly_chart(fig, use_container_width=True)

    st.subheader("Revenue by Month")
    st.dataframe(monthly, use_container_width=True)

    fig = px.line(
        monthly,
        x="YearMonth",
        y=["Revenue", "AOV"],
        markers=True,
        title="Revenue and Average Order Value",
    )
    st.plotly_chart(fig, use_container_width=True)

    c1, c2 = st.columns(2)

    with c1:
        day = (
            sales_df.groupby("DayName")["Revenue"]
            .sum()
            .reindex(
                ["Monday", "Tuesday", "Wednesday", "Thursday",
                 "Friday", "Saturday", "Sunday"]
            )
            .reset_index()
        )
        fig = px.bar(day, x="DayName", y="Revenue", title="Revenue by Day of Week")
        st.plotly_chart(fig, use_container_width=True)

    with c2:
        hour = sales_df.groupby("Hour")["Revenue"].sum().reset_index()
        fig = px.line(hour, x="Hour", y="Revenue", markers=True, title="Revenue by Hour")
        st.plotly_chart(fig, use_container_width=True)


# ============================================================
# DIAGNOSTIC ANALYSIS
# ============================================================
elif page == "Diagnostic Analysis":
    st.header("4. Diagnostic Analysis — Why Did It Happen?")

    st.subheader("Cancellation Risk")
    st.metric(
        "Cancellation Rate",
        f"{diagnostic['cancellation_rate']:.2f}%",
    )

    st.subheader("Revenue Concentration")
    c1, c2 = st.columns(2)
    c1.metric("Top 20% Products Revenue Share", f"{diagnostic['top_20_share']:.1f}%")
    c2.metric("Top 10 Customers Revenue Share", f"{diagnostic['top_10_customer_share']:.1f}%")

    st.subheader("Product Drivers")
    top_products = diagnostic["product_rev"].head(15)
    fig = px.bar(
        top_products.sort_values("Revenue"),
        x="Revenue",
        y="Description",
        orientation="h",
        title="Products Driving Revenue",
    )
    st.plotly_chart(fig, use_container_width=True)

    st.subheader("Country Drivers")
    top_countries = diagnostic["country_rev"].head(15)
    fig = px.bar(
        top_countries.sort_values("Revenue"),
        x="Revenue",
        y="Country",
        orientation="h",
        title="Countries Driving Revenue",
    )
    st.plotly_chart(fig, use_container_width=True)

    st.subheader("Metric Correlation")
    fig = px.imshow(
        diagnostic["corr"],
        text_auto=True,
        aspect="auto",
        title="Correlation: Quantity, Unit Price, Revenue",
    )
    st.plotly_chart(fig, use_container_width=True)

    st.info(
        "Diagnostic analysis identifies associations and business drivers. "
        "Correlation alone does not prove causation."
    )


# ============================================================
# PREDICTIVE ANALYSIS
# ============================================================
elif page == "Predictive Analysis":
    st.header("5. Predictive Analysis — What May Happen Next?")

    if forecast_df is None:
        st.warning(
            "There are not enough monthly observations to build a reliable lag-based forecast."
        )
    else:
        if model_metrics:
            c1, c2, c3 = st.columns(3)
            c1.metric("MAE", f"£{model_metrics['MAE']:,.0f}")
            c2.metric("RMSE", f"£{model_metrics['RMSE']:,.0f}")
            c3.metric(
                "R²",
                "N/A" if pd.isna(model_metrics["R2"]) else f"{model_metrics['R2']:.3f}",
            )

        fig = px.line(
            forecast_df,
            x="Date",
            y="Revenue",
            color="Type",
            markers=True,
            title="Actual Revenue and 3-Month Revenue Forecast",
        )
        st.plotly_chart(fig, use_container_width=True)

        future = forecast_df[forecast_df["Type"] == "Forecast"].copy()
        future["Month"] = future["Date"].dt.strftime("%Y-%m")

        st.subheader("Forecast Values")
        st.dataframe(
            future[["Month", "Revenue"]].rename(
                columns={"Revenue": "Forecast Revenue (£)"}
            ),
            use_container_width=True,
        )

        st.write("### Model features")
        st.write(
            "The Random Forest model uses calendar features and previous-month revenue "
            "lags (1, 2, 3 months) plus a 3-month rolling average."
        )

        st.warning(
            "This forecast is a planning estimate based only on historical transaction data. "
            "It does not include promotions, holidays, prices, inventory, or external economic variables."
        )


# ============================================================
# PRESCRIPTIVE ANALYSIS
# ============================================================
elif page == "Prescriptive Analysis":
    st.header("6. Prescriptive Analysis — What Should We Do?")

    st.write(
        "Prescriptive analytics converts the findings into possible actions. "
        "These are evidence-based business recommendations, not automatic decisions."
    )

    for i, recommendation in enumerate(recommendations, start=1):
        st.success(f"{i}. {recommendation}")

    st.subheader("Risk / Opportunity Framework")

    risk_opportunity = pd.DataFrame({
        "Area": [
            "Product concentration",
            "Customer concentration",
            "Cancellations",
            "Revenue trend",
            "Future forecast",
        ],
        "Risk": [
            "Overdependence on a small group of products",
            "High-value customers may have disproportionate impact",
            "Cancellations can reduce realized revenue",
            "Declining months may affect planning",
            "Forecast error can cause over/under planning",
        ],
        "Opportunity": [
            "Promote strong products and cross-sell related items",
            "Build retention and loyalty programs",
            "Reduce operational causes of cancellations",
            "Use strong months to identify successful products/markets",
            "Use forecast as an inventory and staffing planning signal",
        ],
    })

    st.dataframe(risk_opportunity, use_container_width=True)

    st.subheader("Decision Flow")
    st.markdown(
        """
        **Data → Information → Insight → Decision → Action**

        1. Measure the KPI.
        2. Identify the trend.
        3. Find the major product/customer/country driver.
        4. Identify risk or opportunity.
        5. Select a business action.
        6. Monitor the KPI after the action.
        """
    )


# ============================================================
# DETAILED TABLES
# ============================================================
elif page == "Detailed Tables":
    st.header("7. Detailed Analysis Tables")

    st.subheader("Top Products")
    st.dataframe(product_analysis.head(50), use_container_width=True)

    st.subheader("Country Analysis")
    st.dataframe(country_analysis, use_container_width=True)

    st.subheader("Monthly KPI Table")
    st.dataframe(monthly, use_container_width=True)

    st.subheader("Customer Revenue")
    customer_table = (
        sales_df.groupby("CustomerID")
        .agg(
            Revenue=("Revenue", "sum"),
            Orders=("InvoiceNo", "nunique"),
            Units=("Quantity", "sum"),
        )
        .sort_values("Revenue", ascending=False)
        .reset_index()
    )
    st.dataframe(customer_table.head(100), use_container_width=True)


# ============================================================
# METHODOLOGY
# ============================================================
else:
    st.header("8. Project Methodology")

    st.markdown(
        """
        ### Business Problem
        Analyze historical online retail transactions to understand revenue,
        orders, customers, products, markets, trends, drivers, risks and opportunities,
        then use historical patterns to forecast near-term revenue and propose actions.

        ### Analytics Framework

        **1. Descriptive Analytics**
        - What happened?
        - Revenue, orders, customers, products, units and AOV
        - Monthly, daily and hourly trends
        - Product and country performance

        **2. Diagnostic Analytics**
        - Why did it happen?
        - Product concentration
        - Customer concentration
        - Country drivers
        - Cancellation rate
        - Metric correlations

        **3. Predictive Analytics**
        - What may happen next?
        - Random Forest regression
        - Lagged monthly revenue features
        - Three-month forecast
        - MAE, RMSE and R² evaluation

        **4. Prescriptive Analytics**
        - What should we do?
        - Retain high-value customers
        - Investigate cancellations
        - Prioritize strong products
        - Review weak products
        - Use forecasts for planning

        ### KPI Definitions
        - Revenue = Quantity × Unit Price
        - Orders = unique InvoiceNo
        - Customers = unique CustomerID
        - Products = unique StockCode
        - AOV = Revenue / Orders
        - Units per Order = Units / Orders
        - Revenue per Customer = Revenue / Customers

        ### Important Limitation
        The source data represents historical transactions. Recommendations are
        analytical planning suggestions, not causal proof or guaranteed outcomes.
        """
    )

    st.subheader("Dataset Source")
    st.write(
        "UCI Machine Learning Repository — Online Retail dataset. "
        "The dataset contains UK-based non-store online retail transactions "
        "from 01/12/2010 to 09/12/2011."
    )

st.sidebar.divider()
st.sidebar.caption("Built with Python, Pandas, Plotly, Scikit-learn and Streamlit.")
