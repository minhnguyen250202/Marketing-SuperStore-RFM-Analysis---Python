
# 📊  Marketing SuperStore RFM Analysis - Python 

<img width="952" alt="image" src="https://github.com/user-attachments/assets/df63e539-686b-4d71-a012-b1ad58fe16a9" />

Author: Nguyễn Thị Ánh Minh 

Date:  2025/04/10

Tools Used: Python

---

**📑 Table of Contents**

📌 Background & Overview

📂 Dataset Description & Data Structure

🔎 Final Conclusion & Recommendations

---

# 📌 Background & Overview

### Objective:

**📖 What is this project about?**

This project aims to build a Customer Segmentation System using the RFM (Recency, Frequency, Monetary) model for SuperStore, a global retail company. It focuses on processing large-scale customer data automatically using Python, replacing outdated Excel-based methods.

**What Business Question will it solve?**

- This analysis identifies efficient and meaningful ways to segment a large customer base, enabling personalized marketing and improved customer engagement. 

- It also determines which customers demonstrate the highest levels of loyalty and value, allowing the business to prioritize retention efforts and maximize customer lifetime value.


**👤 Who is this project for?**

- **Marketing Team:** Primary user; uses the segmented data for targeted campaigns.

- **Marketing Director:** Approves and interprets insights; wants simple visuals and impact assessment.

- **Data Analytics Team:** Implements the RFM segmentation system in Python; cares about scalability.

- **Company Leadership:** Observes overall campaign impact; interested in business-level performance.

---

# 📂 Dataset Description & Data Structure

**📌 Data Source**

Source: eCommerce Retail Dataset (likely from [Kaggle] or UCI Repository)

Size: Large dataset with hundreds of thousands of rows and 8 columns

Format: .xlsx (Excel Spreadsheet)

**📊 Data Structure & Relationships**

***1️⃣ Tables Used:***

2 table is used in the dataset.

***2️⃣ Table Schema & Data Snapshot***

- `ecommerce retail`

|Column Name|	Description|Detailed Description|
|:---|:---|:---|
|InvoiceNo|	Unique invoice ID for each transaction|Nominal, a 6-digit integral number uniquely assigned to each transaction. If this code starts with the letter 'C', it indicates a cancellation.
|StockCode|	Product/item code|Nominal, a 5-digit integral number uniquely assigned to each distinct product.|
|Description|	Product name|Nominal|
|Quantity|	Number of units sold|Numeric|
|InvoiceDate|	Date & time of purchase|Numeric, the day and time when each transaction was generated.|
|UnitPrice|	Price per unit (GBP)|Numeric, Product price per unit in sterling.|
|CustomerID|	Unique ID of the customer|Nominal, a 5-digit integral number uniquely assigned to each customer.|
|Country|	Country of the customer|Nominal, the name of the country where each customer resides.|

- `Segmentation` include 2 columns (Segment, RFM Score) 

---
# 🌈 Main Process

## ️🥇 PART 1: DATA PREPARATION 

The data preparation process began by installing and importing key libraries such as `pandas`, `numpy`, `matplotlib`, `seaborn`, `squarify`, and `ydata_profiling`. The dataset, **"ecommerce retail.xlsx"**, was loaded from Google Drive using pandas.read_excel().

A preliminary inspection was conducted using **`.info()` and `.describe()`** to **check the dataset’s structure and quality**. This revealed:

- Missing values in the Description and CustomerID columns.

- Suspicious values in Quantity and UnitPrice, including negative numbers, which are likely errors or require special handling (e.g., returns or cancellations).

To **gain a deeper understanding** of the dataset’s features, I used the **ydata_profiling library** to generate an automated profile report. This provided valuable insights into:

- Column data types (categorical, numerical, etc.)

- Missing value distribution

- Potential anomalies or duplicates

- Correlation between variables

This initial step laid the foundation for the cleaning and transformation processes that follow, ensuring that the dataset is accurate and analysis-ready.

**📌 Initial Observations:**

Columns `Description` and `CustomerID` contain missing values.
<img width="871" alt="image" src="https://github.com/user-attachments/assets/c219e387-e338-4fd0-9bb9-d4225a17e44d" />

`Quantity` and `UnitPrice` columns have negative values, which are unusual and need to be addressed.

<img width="836" alt="image" src="https://github.com/user-attachments/assets/1e488b04-01ac-4f4a-94cb-45957c0ffb94" />
<img width="833" alt="image" src="https://github.com/user-attachments/assets/ad20f7ad-54ea-4e42-ad26-ca6340527841" />

## 🔍 Data Cleaning – Detecting and Handling Invalid Entries

After the initial profiling, I moved on to **validate and clean suspicious values** found in the dataset, particularly in the 'CustomerID`,`Quantity` and `UnitPrice` columns.

**👤 Handling Missing CustomerID**

A significant portion of the dataset (over 20%) contains missing values in the `CustomerID` column. Since `CustomerID` is essential for customer-level analysis—such as segmentation, RFM analysis, and calculating customer lifetime value—these records would not provide reliable insights.

🧹 To maintain the integrity of customer-centric analytics, we decided to remove rows with null CustomerID using the following code:

```python
new_ecommerce_retail_2 = new_ecommerce_retail[new_ecommerce_retail["CustomerID"].notnull()]
```

This step ensures that all transactions retained in the dataset are associated with an identifiable customer, improving the accuracy of subsequent analysis.

**🛑Detecting Invalid Values**

Negative Quantity:

- Some transactions had negative quantities. To investigate further, we checked if these were cancellations by verifying whether the InvoiceNo started with the letter 'C'. This is a known business rule indicating a cancelled order.

-> **Most negative quantities were indeed cancellations.**

Negative UnitPrice:

- A few transactions with negative UnitPrice values, which are not realistic in a normal retail setting. These values may be due to data entry errors or incorrect system recordings.

**🧼 Handling Invalid Values**

For entries with negative quantities not marked as cancellations, further investigation or exclusion may be necessary depending on the business context.

Rows with negative or zero UnitPrice will be filtered out in the cleaning step, as they can distort revenue and customer value metrics.

---
## ️🥈 PART 2: DATA PROCESSING 

#### Step 1: Calculate Transaction Cost & Identify the Last Purchase Date

- Created a new column cost by multiplying Quantity and UnitPrice to calculate the total amount spent per transaction:

```python
new_ecommerce_retail_2["cost"] = new_ecommerce_retail_2["Quantity"] * new_ecommerce_retail_2["UnitPrice"]
```

- Identified the most recent transaction date to use in Recency calculations:
```python

last_day = new_ecommerce_retail_2['Day'].max()
```

#### Step 2: RFM Analysis

***Performed RFM analysis by grouping data by `CustomerID`:***

- **Recency**: Days since last purchase.

- **Frequency**: Total number of transactions.

- **Monetary**: Total amount spent.

- **Start_Day**: First transaction date for context.
```python

RFM = new_ecommerce_retail_2.groupby("CustomerID").agg(
    Recency=('Day', lambda x: last_day - x.max()),
    Frequency=('CustomerID', 'count'),
    Monetary=('cost', 'sum'),
    Start_Day=('Day', 'min')
).reset_index()
```

***Extracted days from Recency and derived Start_Month to support cohort analysis:***
```python

RFM["Recency"] = RFM["Recency"].dt.days.astype('int64')
RFM["Start_Day"] = pd.to_datetime(RFM["Start_Day"])
RFM["Start_Month"] = RFM["Start_Day"].apply(lambda x: x.replace(day=1))
```

<img width="530" alt="image" src="https://github.com/user-attachments/assets/54cb89c5-2036-4315-83d5-9bc75f5c0646" />

#### Step 3: RFM Scoring

**To assign RFM scores:**

- Reversed Recency (lower recency = higher score).

- Used pd.qcut to bin Recency, Frequency, and Monetary into 5 quantiles.

- Concatenated scores to form the RFM_Score.

```python
RFM["Recency_Reverse"] = -RFM["Recency"]
RFM["R"] = pd.qcut(RFM["Recency_Reverse"], 5, labels=range(1,6)).astype('int64')
RFM["F"] = pd.qcut(RFM["Frequency"], 5, labels=range(1,6))
RFM["M"] = pd.qcut(RFM["Monetary"], 5, labels=range(1,6))
RFM["RFM_Score"] = RFM["R"].astype(str) + RFM["F"].astype(str) + RFM["M"].astype(str)
```
<img width="795" alt="image" src="https://github.com/user-attachments/assets/83f7c874-32f0-426b-b386-ebfe72dae1d5" />

#### Step 4: Customer Segmentation

We imported a predefined segmentation mapping based on RFM scores:

```python
Segmentation = pd.read_excel(path + 'ecommerce retail.xlsx', sheet_name='Segmentation')
Segmentation["RFM Score"] = Segmentation["RFM Score"].str.split(',')
Segmentation = Segmentation.explode("RFM Score").reset_index(drop=True)
```
<img width="217" alt="image" src="https://github.com/user-attachments/assets/77e9a905-9c62-487b-91e8-9c1231d3a94f" />

Then merged the RFM table with the segmentation table to assign customer segments:

```python

RFM_final = RFM.merge(Segmentation, left_on='RFM_Score', right_on='RFM Score')
```

<img width="999" alt="image" src="https://github.com/user-attachments/assets/b7e94165-1a8c-4cef-9e68-fea9e2b3d8ab" />

---
## ️🥉 PART 3: DATA VISUALIZATION 

**🎯 Visualizing Customer Segments Based on RFM**

Used a horizontal count plot to visualize the distribution of customers across different RFM segments:

```python
sns.countplot(
    y=RFM_final["Segment"],
    order=RFM_final["Segment"].value_counts().index,
    palette="viridis"
)
plt.xlabel("Number of Customers")
plt.ylabel("Customer Segments")
plt.title("Customer Distribution by RFM Segment")
plt.show()
```

<img width="777" alt="image" src="https://github.com/user-attachments/assets/0a31db68-6490-4bf4-ba2a-a114b0147842" />

**🔍 Key Insights:**

**Champions and Lost Customers** are the two largest segments, indicating a strong group of high-value active customers and a significant number of previously valuable but now inactive ones.

Segments like **Loyal, Promising, and At Risk **also show noticeable volume, highlighting opportunities for retention and engagement strategies.

Smaller segments such as **New Customers, Need Attention, and Cannot Lose Them** can be targeted for specific marketing campaigns to increase lifetime value.

--- 
# ✅ Conclusions & Recommendations

## 🔍 Conclusion

The RFM analysis reveals clear segmentation of customers based on Recency, Frequency, and Monetary value.

Key highlights include:

- **Champions represent the largest segment**, indicating a loyal customer base that spends highly and frequently returns.

- **Lost Customers** also form a significant group, reflecting customers who once contributed substantial value but have since become inactive.

- Segments such as **At Risk, About to Sleep, and Cannot Lose Them require prioritized retention strategies**.

## 💡 Recommendation 

|Recommended| Strategy|
|:---|:---|
|Champions|	Reward points, special offers, or invitations to try new products to maintain loyalty and turn them into brand ambassadors.|
|Loyal|	Upsell premium products, suggest items based on purchase history, and request reviews to increase engagement.|
|Potential Loyalist|	Implement membership programs or discount offers to convert them into loyal customers.|
|New Customers|	Provide product/service usage guidance and create a welcoming experience to improve retention.|
|Promising|	Build brand awareness and offer free trials to encourage repeat purchases.|
|Need Attention|	Offer time-limited promotions or discounts based on purchase history to stimulate return.|
|About to Sleep|	Send emails with helpful resources or special offers as reminders.|
|At Risk|	Personalize care emails and provide incentives to win them back.|
|Cannot Lose Them|	Engage directly via calls or meetings, introduce new products or exclusive offers to retain them.|
|Hibernating|	Rekindle brand value through exclusive promotions or product recommendations.|
|Lost Customers|	Run retargeting marketing campaigns; if ineffective, consider excluding them from main strategies.|


