# Customer-Segmentation-with-RFM-Analysis

## Introduction

This script performs customer segmentation analysis on a dataset using Python and the Pandas library. It aims to categorize customers into different segments based on their purchase behavior.

## Prerequisites

Before running the script, make sure you have the following Python libraries installed:

- Pandas
- NumPy
- Matplotlib (for visualizations)
- Seaborn (for visualizations)
- Plotly (for interactive visualizations)

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
