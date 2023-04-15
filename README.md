# Customer-Segmentation-with-RFM-Analysis

1) Data Preparation
Loading the dataset and continuing by creating a copy to avoid making changes to the main dataset.
df_ = pd.read_csv("flo_data_20k.csv")
df = df_.copy()
Here are some observations to provide an overview:
First 10 observations, variable names, size, descriptive statistics, null values, variable types, respectively.
df.head(10)
df.columns
df.shape
df.describe().T
df.isnull().sum()
df.info()

The columns we will handle are ([‘master_id’, ‘order_channel’, ‘last_order_channel’, ‘first_order_date’, ‘last_order_date’, ‘last_order_date_online’, ‘last_order_date_offline’, ‘order_num_total_ever_online’, ‘order_num_total_ever_offline’, ‘customer_value_total_ever_offline’, ‘customer_value_total_ever_online’, ‘interested_in_categories_12’, ‘order_num_total’, ‘customer_value_total’])
Created new variables for each customer’s total purchases and spending:
df["order_num_total"] = df["order_num_total_ever_online"] + df["order_num_total_ever_offline"]
df["customer_value_total"] = df["customer_value_total_ever_offline"] + df["customer_value_total_ever_online"]
Type of variables expressing date are converted to date:
date_columns = df.columns[df.columns.str.contains("date")]
df[date_columns] = df[date_columns].apply(pd.to_datetime)
The distribution of the number of customers in the shopping channels, the total number of products purchased and the total expenditures were examined:
df.groupby("order_channel").agg({"master_id":"count",
                                 "order_num_total":"sum",
                                 "customer_value_total":"sum"})

The top 10 customers with the most profits and the most orders are:
df.sort_values("customer_value_total", ascending=False)[:10]
df.sort_values("order_num_total", ascending=False)[:10]
Now the entire data preparation process up to this point has been functionalized.
def data_prep(dataframe):
    dataframe["order_num_total"] = dataframe["order_num_total_ever_online"] + dataframe["order_num_total_ever_offline"]
    dataframe["customer_value_total"] = dataframe["customer_value_total_ever_offline"] + dataframe["customer_value_total_ever_online"]
    date_columns = dataframe.columns[dataframe.columns.str.contains("date")]
    dataframe[date_columns] = dataframe[date_columns].apply(pd.to_datetime)
    return df
2) Calculating RFM Metrics
Two days after the date of the last purchase in the data set was accepted as the analysis date, as it was a data of a past date.
A new rfm dataframe was created with customer_id, recency, frequency and monetary values.
rfm = pd.DataFrame()
rfm["customer_id"] = df["master_id"]
rfm["recency"] = (analysis_date - df["last_order_date"]).astype('timedelta64[D]')
rfm["frequency"] = df["order_num_total"]
rfm["monetary"] = df["customer_value_total"]

rfm.head()

3) Calculating RF and RFM Scores
Recency, Frequency and Monetary metrics were converted to scores between 1–5 with the help of qcut and these scores recorded as recency_score, frequency_score and monetary_score.
rfm["recency_score"] = pd.qcut(rfm['recency'], 5, labels=[5, 4, 3, 2, 1])
rfm["frequency_score"] = pd.qcut(rfm['frequency'].rank(method="first"), 5, labels=[1, 2, 3, 4, 5])
rfm["monetary_score"] = pd.qcut(rfm['monetary'], 5, labels=[1, 2, 3, 4, 5])

rfm.head()

Expressing recency_score and frequency_score as a single variable and saving as RF_SCORE and expressing recency_score and frequency_score and monetary_score as a single variable and saving as RFM_SCORE is as follows:
rfm["RF_SCORE"] = (rfm['recency_score'].astype(str) + rfm['frequency_score'].astype(str))
rfm["RFM_SCORE"] = (rfm['recency_score'].astype(str) + rfm['frequency_score'].astype(str) + rfm['monetary_score'].astype(str))
rfm.head()
4) Definition of RF Scores as Segments
In order to make the generated RFM scores more clear, RF_SCORE was converted to segments as follows with the help of segment definition and defined seg_map.

RFM Analysis Table
seg_map = {
    r'[1-2][1-2]': 'hibernating',
    r'[1-2][3-4]': 'at_Risk',
    r'[1-2]5': 'cant_loose',
    r'3[1-2]': 'about_to_sleep',
    r'33': 'need_attention',
    r'[3-4][4-5]': 'loyal_customers',
    r'41': 'promising',
    r'51': 'new_customers',
    r'[4-5][2-3]': 'potential_loyalists',
    r'5[4-5]': 'champions'
}

rfm['segment'] = rfm['RF_SCORE'].replace(seg_map, regex=True)

rfm.head()

5) It’s action time!
The recency, frequency and monetary averages of the segments were examined.
rfm[["segment", "recency", "frequency", "monetary"]].groupby("segment").agg(["mean", "count"])

Case 1: FLO includes a new women’s shoe brand. The product prices of the brand it includes are above the general customer preferences. For this reason, customers in the profile who will be interested in the promotion of the brand and product sales are requested to be contacted privately. These customers were planned to be loyal and female shoppers. Saved the id numbers of the customers in the csv file as new_brand_trgt_cstmr_id.cvs.
target_segments_customer_ids = rfm[rfm["segment"].isin(["champions","loyal_customers"])]["customer_id"]
cust_ids = df[(df["master_id"].isin(target_segments_customer_ids)) &(df["interested_in_categories_12"].str.contains("WOMEN"))]["master_id"]
cust_ids.to_csv("new_brand_trgt_cstmr_id.cvs", index=False)
Case 2: Up to 40% discount is planned for Men’s and Children’s products. We want to specifically target customers who are good customers in the past who are interested in categories related to this discount, but have not shopped for a long time and new customers. Saved the ids of the customers in the appropriate profile to the csv file as discount_target_customer_ids.csv.
target_segments_customer_ids = rfm[rfm["segment"].isin(["cant_loose","hibernating","new_customers"])]["customer_id"]
cust_ids = df[(df["master_id"].isin(target_segments_customer_ids)) & ((df["interested_in_categories_12"].str.contains("MEN"))|(df["interested_in_categories_12"].str.contains("KIDS")))]["master_id"]
cust_ids.to_csv("discount_target_customer_ids.csv", index=False)
The functionalized version of all these processes is below.
def create_rfm(dataframe):
    # Data preparation
    dataframe["order_num_total"] = dataframe["order_num_total_ever_online"] + dataframe["order_num_total_ever_offline"]
    dataframe["customer_value_total"] = dataframe["customer_value_total_ever_offline"] + dataframe["customer_value_total_ever_online"]
    date_columns = dataframe.columns[dataframe.columns.str.contains("date")]
    dataframe[date_columns] = dataframe[date_columns].apply(pd.to_datetime)

    # Calculating RFM metrics
    dataframe["last_order_date"].max()  # 2021-05-30
    analysis_date = dt.datetime(2021, 6, 1)
    rfm = pd.DataFrame()
    rfm["customer_id"] = dataframe["master_id"]
    rfm["recency"] = (analysis_date - dataframe["last_order_date"]).astype('timedelta64[D]')
    rfm["frequency"] = dataframe["order_num_total"]
    rfm["monetary"] = dataframe["customer_value_total"]

    # Calculating RF and RFM Scores
    rfm["recency_score"] = pd.qcut(rfm['recency'], 5, labels=[5, 4, 3, 2, 1])
    rfm["frequency_score"] = pd.qcut(rfm['frequency'].rank(method="first"), 5, labels=[1, 2, 3, 4, 5])
    rfm["monetary_score"] = pd.qcut(rfm['monetary'], 5, labels=[1, 2, 3, 4, 5])
    rfm["RF_SCORE"] = (rfm['recency_score'].astype(str) + rfm['frequency_score'].astype(str))
    rfm["RFM_SCORE"] = (rfm['recency_score'].astype(str) + rfm['frequency_score'].astype(str) + rfm['monetary_score'].astype(str))

    # Naming of segments
    seg_map = {
        r'[1-2][1-2]': 'hibernating',
        r'[1-2][3-4]': 'at_Risk',
        r'[1-2]5': 'cant_loose',
        r'3[1-2]': 'about_to_sleep',
        r'33': 'need_attention',
        r'[3-4][4-5]': 'loyal_customers',
        r'41': 'promising',
        r'51': 'new_customers',
        r'[4-5][2-3]': 'potential_loyalists',
        r'5[4-5]': 'champions'
    }
    rfm['segment'] = rfm['RF_SCORE'].replace(seg_map, regex=True)

    return rfm[["customer_id", "recency","frequency","monetary","RF_SCORE","RFM_SCORE","segment"]]

rfm_df = create_rfm(df)
In conclusion, the most important benefit of RFM analysis for a company is that it can increase sales, improve customer loyalty, and prevent customer churn by taking the right action at the right time.
