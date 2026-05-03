I cleaned a transactional dataset by handling missing and invalid values, standardizing data types, enforcing business rules, and validating the final dataset for accuracy.

import pandas as pd
import numpy as np

df = pd.read_csv("dirty_cafe_sales.csv")

df.replace(["ERROR", "UNKNOWN", ""], np.nan, inplace=True)

df["Quantity"] = pd.to_numeric(df["Quantity"], errors='coerce')
df["Price Per Unit"] = pd.to_numeric(df["Price Per Unit"], errors='coerce')
df["Total Spent"] = pd.to_numeric(df["Total Spent"], errors='coerce')
df["Transaction Date"] = pd.to_datetime(df["Transaction Date"], errors='coerce')

df["Quantity"] = df["Quantity"].fillna(df["Quantity"].median())
df["Price Per Unit"] = df["Price Per Unit"].fillna(df["Price Per Unit"].median())

df["Item"] = df["Item"].fillna(df["Item"].mode()[0])
df["Payment Method"] = df["Payment Method"].fillna("Unknown")
df["Location"] = df["Location"].fillna("Unknown")

price_map = {
    "Coffee": 2, "Tea": 1.5, "Sandwich": 4, "Salad": 5,
    "Cake": 3, "Cookie": 1, "Smoothie": 4, "Juice": 3
}

df["Price Per Unit"] = df["Item"].map(price_map)
df["Price Per Unit"] = df["Price Per Unit"].fillna(df["Price Per Unit"].median())

df["Total Spent"] = df["Quantity"] * df["Price Per Unit"]

df["Transaction Date"] = df["Transaction Date"].ffill()

df["Day"] = df["Transaction Date"].dt.day_name()
df["Month"] = df["Transaction Date"].dt.month

df.drop_duplicates(inplace=True)

print(df.isnull().sum())
print(df.duplicated().sum())

df.to_csv("cleaned_data.csv", index=False)
print((df["Quantity"] * df["Price Per Unit"] == df["Total Spent"]).all())

