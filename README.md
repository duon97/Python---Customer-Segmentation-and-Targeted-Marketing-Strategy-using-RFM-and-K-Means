# Customer Segmentation: RFM and K-means

## I. Introduction

This project segments the customers of an online retailer using their transaction history. The goal is to help the company apply a different strategy to each customer group, based on how recently, how often, and how much each customer buys.

## II. The Goal of Creating This Project

This project aims to boost business performance by:
- Segmenting customers into meaningful groups from purchase behavior
- Identifying which customers to reward, upsell, or re-engage
- Supporting targeted marketing instead of one uniform strategy

## III. Dataset description table

The data contains sales history only, with no demographic information about customers, so segmentation uses the RFM model.

### transactions

| Column Name | Description |
|---|---|
| InvoiceNo | Invoice ID |
| StockCode | ID of the purchased item |
| Description | Item description |
| Quantity | Quantity of the item purchased |
| InvoiceDate | Purchase date |
| UnitPrice | Item price (GBP) |
| CustomerID | ID of the purchasing customer |
| Country | Customer's country |

### RFM attributes (calculated per customer)

| Attribute | Description |
|---|---|
| Recency | Days since the customer's last purchase |
| Frequency | Number of distinct invoices of the customer |
| Monetary (Amount) | Total amount (GBP) of all the customer's transactions |

## IV. Explore the Dataset

### Step 1: Collect, clean, and preprocess the transaction data

```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score

df = pd.read_csv("data/transactions.csv", encoding="ISO-8859-1")
df["InvoiceDate"] = pd.to_datetime(df["InvoiceDate"])

# Remove rows without a customer, cancelled invoices, and invalid quantity or price
df = df.dropna(subset=["CustomerID"])
df = df[~df["InvoiceNo"].astype(str).str.startswith("C")]
df = df[(df["Quantity"] > 0) & (df["UnitPrice"] > 0)]

df["CustomerID"] = df["CustomerID"].astype(int)
df["Amount"] = df["Quantity"] * df["UnitPrice"]
```

### Step 2: Calculate RFM

```python
snapshot_date = df["InvoiceDate"].max() + pd.Timedelta(days=1)

rfm = df.groupby("CustomerID").agg(
    Recency=("InvoiceDate", lambda x: (snapshot_date - x.max()).days),
    Frequency=("InvoiceNo", "nunique"),
    Amount=("Amount", "sum"),
).reset_index()
```

Remove outliers with the IQR rule, then standardize the RFM metrics.

```python
def remove_outliers(data, cols):
    for c in cols:
        q1, q3 = data[c].quantile([0.25, 0.75])
        iqr = q3 - q1
        data = data[(data[c] >= q1 - 1.5 * iqr) & (data[c] <= q3 + 1.5 * iqr)]
    return data

rfm_clean = remove_outliers(rfm, ["Recency", "Frequency", "Amount"])

scaler = StandardScaler()
rfm_scaled = scaler.fit_transform(rfm_clean[["Recency", "Frequency", "Amount"]])
```

### Step 3: Choose K and cluster customers with K-means

```python
inertia, sil = [], []
k_range = range(2, 9)
for k in k_range:
    km = KMeans(n_clusters=k, random_state=42, n_init=10).fit(rfm_scaled)
    inertia.append(km.inertia_)
    sil.append(silhouette_score(rfm_scaled, km.labels_))

fig, ax = plt.subplots(1, 2, figsize=(12, 4))
ax[0].plot(k_range, inertia, marker="o")
ax[0].set_title("Elbow method")
ax[0].set_xlabel("K")
ax[1].plot(k_range, sil, marker="o")
ax[1].set_title("Silhouette score")
ax[1].set_xlabel("K")
plt.show()
```

```python
kmeans = KMeans(n_clusters=3, random_state=42, n_init=10)
rfm_clean = rfm_clean.copy()
rfm_clean["Cluster"] = kmeans.fit_predict(rfm_scaled)

rfm_clean.groupby("Cluster").agg(
    Customers=("CustomerID", "count"),
    Recency=("Recency", "mean"),
    Frequency=("Frequency", "mean"),
    Amount=("Amount", "mean"),
).round(1)
```

### Step 4: Compare clusters

```python
sns.set_theme()
fig, ax = plt.subplots(figsize=(8, 5))
sns.boxplot(x="Cluster", y="Amount", data=rfm_clean, ax=ax)
ax.set_title("Số tiền khách hàng đã mua hàng trong từng cụm với K = 3")
plt.show()
```

<img width="1131" height="571" alt="Amount by cluster, K = 3" src="images/boxplot_amount_k3.png" />

Insights (values read from the chart, approximate)

- Cluster 2 has the highest spend: median about £2,700, middle half of customers between about £1,900 and £3,900, and no outliers.
- Cluster 0 is the middle group: median about £1,000, middle half between about £650 and £1,600. It has many high outliers up to about £5,300, which overlap the range of Cluster 2.
- Cluster 1 has the lowest spend: median about £300, middle half between about £170 and £430, and almost all customers below about £850.
- Median spend separates the three clusters clearly (about £300, £1,000, £2,700), but Clusters 0 and 2 overlap at the top end, so Monetary alone does not fully separate them.

## V. Insight and Recommendation

| Insight | Recommendation |
|---|---|
| Cluster 2 (High-Value) is made up of frequent, recent, high spenders. Median spend is about £2,700 with no outliers, so spend is consistently high. | Reward and retain with loyalty benefits and priority service, since losing these customers costs the most revenue. |
| Cluster 0 (Medium-Value) has moderate activity, with median spend about £1,000. High outliers reach the spend range of Cluster 2. | Upsell with bundles and cross-sell offers, and track the high-spend outliers as candidates to move into High-Value. |
| Cluster 1 (Low-Value) is inactive or low-spending, with median spend about £300 and almost all customers below about £850. | Run engagement campaigns to drive repeat purchases, and measure reactivation before increasing spend on this group. |
| Median spend separates the three clusters, but Clusters 0 and 2 overlap at the top end. | Define segment rules on Recency and Frequency as well as Monetary, so that high-spend outliers in Cluster 0 are not treated as High-Value by mistake. |

## Tech Stack

- Python: pandas, numpy, scikit-learn (StandardScaler, KMeans, silhouette_score), seaborn, matplotlib
- RFM model and K-means (K = 3)

## Repository Structure

```
.
├── README.md
├── customer_segmentation.ipynb
├── data/
│   └── transactions.csv
└── images/
    └── boxplot_amount_k3.png
```
