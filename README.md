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
| Recency | How recently the customer last purchased |
| Frequency | How often the customer purchases |
| Monetary (Amount) | Total amount (GBP) of all the customer's transactions |

## IV. Methodology

### 1. Prepare the data
Collect, clean, and preprocess the transaction data, then aggregate it into one row per customer with Recency, Frequency, and Monetary. The three metrics are standardized so that no single metric dominates the distance calculation.

### 2. K-means clustering
K-means groups customers so that customers in the same cluster are close to each other in RFM space and far from customers in other clusters. The number of clusters K must be chosen in advance, so the model is run with K = 3, 5, and 7.

### 3. Silhouette score
The silhouette score evaluates how well each customer fits its own cluster compared with the nearest other cluster. It ranges from -1 to 1: values close to 1 mean clusters are compact and well separated, values near 0 mean clusters overlap, and negative values mean customers are likely in the wrong cluster. It is used to compare different values of K.

| K | Silhouette score |
|---|---|
| 3 | 0.34 |
| 5 | 0.30 |
| 7 | 0.31 |

K = 3 has the highest score (0.34), against 0.31 for K = 7 and 0.30 for K = 5. All three fall in the moderate range (0.25 to 0.5), so the clusters have a reasonable structure but are not sharply separated. The differences between the three scores are small (0.04), so the score alone does not decide the choice.
### 4. Interpret each cluster
Each cluster is described by its RFM profile, using boxplots of Amount (Monetary) per cluster to compare the median, the spread, and the outliers, then labeled by business value.

## V. Cluster Analysis

Values below are read from the charts and are approximate.

### K = 3 (silhouette 0.34)

<img width="997" height="570" alt="image" src="https://github.com/user-attachments/assets/37b5b206-4db7-4eb4-9e1d-42c2c38c562e" />

- Cluster 2: highest spend. Median about £2,700, middle half between about £1,900 and £3,900, no outliers.
- Cluster 0: middle spend. Median about £1,000, middle half between about £650 and £1,600. Many high outliers up to about £5,300 reach the range of Cluster 2.
- Cluster 1: lowest spend. Median about £300, middle half between about £170 and £430, almost all customers below about £850.

### K = 5 (silhouette 0.30)

<img width="1123" height="543" alt="image" src="https://github.com/user-attachments/assets/ef27d83b-4dde-4e3e-8fd7-79fd2a5ee5db" />

- Cluster 1: highest spend. Median about £3,100, middle half between about £2,100 and £4,250.
- Cluster 3: upper-middle spend. Median about £1,750, middle half between about £1,170 and £2,580, with high outliers up to about £5,700.
- Cluster 0: middle spend. Median about £800, middle half between about £580 and £1,130, with outliers up to about £5,000.
- Cluster 4: low spend. Median about £390, middle half between about £250 and £600.
- Cluster 2: lowest spend. Median about £230, middle half between about £150 and £330, maximum about £770.

### K = 7 (silhouette 0.31)

<img width="1047" height="581" alt="image" src="https://github.com/user-attachments/assets/b7338a7a-9367-406b-bab5-b40081c25d7c" />

- Cluster 3: highest spend. Median about £4,750, middle half between about £4,120 and £5,260.
- Clusters 1 and 5: upper-middle spend. Medians about £2,150 and £2,100, middle halves between about £1,550 and £3,080 (Cluster 1) and £1,450 and £2,890 (Cluster 5).
- Clusters 2 and 6: middle spend. Medians about £700 and £790, middle halves between about £490 and £950 (Cluster 2) and £580 and £1,130 (Cluster 6). Cluster 6 has many high outliers up to about £4,970.
- Clusters 0 and 4: lowest spend. Medians about £220 and £270, maximum about £850 and £1,020.
- Three pairs of clusters (1 and 5, 2 and 6, 0 and 4) have almost the same Amount distribution, so they can only be told apart by Recency or Frequency.

## VI. Insight and Recommendation

| Insight | Recommendation |
|---|---|
| K = 3 has the highest silhouette score (0.34), against 0.31 for K = 7 and 0.30 for K = 5. All scores are in the moderate range and close together, and the Amount boxplot at K = 3 shows clearly separated clusters, with overlap only through the high outliers of Cluster 0. | Use K = 3 as the working segmentation, and review the high-spend customers of Cluster 0 individually, since they are the ones most likely to sit near the boundary with Cluster 2. |
| With K = 3, Cluster 2 (High-Value) is made up of frequent, recent, high spenders, with median spend about £2,700 and no outliers. | Reward and retain with loyalty benefits and priority service, since losing these customers costs the most revenue. |
| With K = 3, Cluster 0 (Medium-Value) has moderate activity, with median spend about £1,000 and high outliers that reach the range of Cluster 2. | Upsell with bundles and cross-sell offers, and track the high-spend outliers as candidates to move into High-Value. |
| With K = 3, Cluster 1 (Low-Value) is inactive or low-spending, with median spend about £300 and almost all customers below about £850. | Run engagement campaigns to drive repeat purchases, and measure reactivation before increasing spend on this group. |
| With K = 5, the upper-middle group (Cluster 3, median about £1,750) is separated from the top group (Cluster 1, median about £3,100), and low-value customers are split into Cluster 4 (about £390) and Cluster 2 (about £230). The silhouette score is the lowest of the three (0.30). | Move to K = 5 only if the company can run more than three distinct strategies and a test shows that the extra clusters respond differently. |
| With K = 7, three pairs of clusters have almost the same Amount distribution, and the silhouette score (0.31) is not better than K = 3. | Do not use K = 7 for strategy, since the extra clusters add complexity without a clear difference in spend or a better fit. |
| High outliers appear in the middle clusters at every K (Cluster 0 at K = 3, Clusters 0 and 3 at K = 5, Cluster 6 at K = 7). | Review these customers individually, since they spend like a higher segment but are grouped with a lower one by the other RFM metrics. |

## Tech Stack

- Python: pandas, scikit-learn (K-means, silhouette score), seaborn, matplotlib
- RFM model, K-means, silhouette score
