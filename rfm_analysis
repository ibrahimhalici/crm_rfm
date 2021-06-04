# importing libraries
import datetime as dt
import pandas as pd

# pandas display settings
pd.set_option('display.max_columns',None)
pd.set_option('display.max_rows', 20)
pd.set_option('display.float_format', lambda x: '%.3f' % x)

# importing and first look to data
df_ = pd.read_excel("datasets/online_retail_II.xlsx", sheet_name="Year 2010-2011")
df = df_.copy()
df.head()
df.shape


# descriptive statictics for dataset
df.shape
df.info()
df.quantile([0, 0.05, 0.50, 0.95, 0.99, 1]).T
df.describe().T


# are there any null values in data? if there how many? 
df.isnull().values.any()
df.isnull().sum()


# drop the null values.
df.dropna(inplace = True)
df.isnull().sum()


# how many unique products in data?
df['StockCode'].nunique() # 3684

# what is the sale number for every product? 
df['StockCode'].value_counts().head()

# most ordered 5 product
df['StockCode'].value_counts().sort_values(ascending=False).head()

# drop the 'Canceled' orders.
df = df[~df["Invoice"].str.contains("C", na=False)]

# Create a Variable named TotalPrice to show the exact monetary value for each invoice
df["TotalPrice"] = df["Quantity"] * df["Price"]
df.head()

######################
# creating RFM Metrics
######################

# Recency
# Frequency
# Monetary

# target_date for analyse
df["InvoiceDate"].max()
target_date = dt.datetime(2011, 12, 11)

# creating Recency, Frequency and Monetary Metrics for every customer.
rfm = df.groupby('Customer ID').agg({'InvoiceDate': lambda x: (target_date - x.max()).days,
                                     'Invoice': lambda x: x.nunique(),
                                     'TotalPrice': lambda x: x.sum()})

# changing the column names of dataframe
rfm.columns = ['recency', 'frequency', 'monetary']

# dropping values with monetary value below 0
rfm = rfm[rfm["monetary"] > 0]

# checking data
rfm.head()
rfm.describe().T

####################
# creating RFM Scores
#####################


# creating scores and score variables for RFM metrics
rfm["recency_score"] = pd.qcut(rfm['recency'], 5, labels=[5, 4, 3, 2, 1])
rfm["frequency_score"] = pd.qcut(rfm['frequency'].rank(method="first"), 5, labels=[1, 2, 3, 4, 5])
rfm["monetary_score"] = pd.qcut(rfm['monetary'], 5, labels=[1, 2, 3, 4, 5])



# creating RFM SCORE with Recency and Frequency Score. 
rfm["RFM_SCORE"] = (rfm['recency_score'].astype(str) +
                    rfm['frequency_score'].astype(str))


rfm.describe().T

##############################
# segmentation with RFM Scores
##############################

# segmentation dict for RF scores.
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

# creating segments with rfm scores using regex.
rfm['segment'] = rfm['RFM_SCORE'].replace(seg_map, regex=True)
rfm.head()

# checking the distribution of segments and values.
rfm[["segment", "recency", "frequency", "monetary"]].groupby("segment").agg(["mean", "count"])


####################################
# segment analyzing- Decision Making
####################################

# creating special dataframes with some segments to take decision and make analyze
cant_loose = rfm[rfm['segment'] =='cant_loose']
new_customers = rfm[rfm['segment']=='new_customers']
need_attention = rfm[rfm['segment']=='need_attention']


# 63 customers. averagely. 8 orders in 132 days. 2796 monetary value (avg).
# high frequency but they not recent. we should gain these customers again with push notifications, special and individual discounts
# we should apply special marketing strategies for them,their monetary score is 5 which is max.
cant_loose['recency_score'].value_counts()
cant_loose['frequency_score'].value_counts()
cant_loose['monetary_score'].value_counts()
cant_loose.info()
cant_loose.describe().T
cant_loose[["recency", "frequency", "monetary",]].agg(["mean", "count"])


# 42 customers. avg. 1 order in 7 days. 388 monetary.
# new customer. that means we are with fresh corporate looking and image. they are neutral to us. we should do what we best
# and show them what we do. they can be our loyal customers with the right marketing strategy. they should not see our weaknesses
# we should take action on shipping conditions, shipping time and packaging which cause potential loose of customers.
new_customers.shape
new_customers[["recency", "frequency", "monetary",]].agg(["mean", "count"])

# 172 customers. avg. 2 orders in 52days. 897 monetary.
# `many a little makes a mickle` "DROPS BECOME LAKES :)" by looking their consuming behavior we can define periodical discounts to increase 
# their recency score which will make them our loyal customers. we make them order in 25 instead of 50.
# every 25 days we can send them mails, and notifications, maybe the products that is already in discount.
need_attention.shape
need_attention[["recency", "frequency", "monetary",]].agg(["mean", "count"])


# creating and exporting loyal customers dataframe with customer id as excel file
loyal_df = pd.DataFrame()
loyal_df["loyal_customers_id"] = rfm[rfm["segment"] == "loyal_customers"].index
loyal_df.to_excel("loyal_customers.xlsx",
                  sheet_name='Loyal Customers Index')










