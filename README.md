weather data analyser 

import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from io import StringIO
import os

# ==========================================
# STEP 1: LOAD WEATHER DATA
# ==========================================

file_path = r"D:\code\PYTHON\__pycache__\sql\weather_analysis\weather_data.csv"

if not os.path.exists(file_path):
    print("ERROR: CSV file not found!")
    print("Please check the file path:", file_path)
    raise SystemExit

with open(file_path, "r", encoding="utf-8-sig") as file:
    content = file.read()

# Handle literal \t characters in the file
content = content.replace(r"\t", "\t")

# Read the dataset
df = pd.read_csv(StringIO(content), sep="\t")

# If data is comma-separated instead
if len(df.columns) == 1:
    df = pd.read_csv(StringIO(content), sep=",")

# Clean column names
df.columns = df.columns.str.strip()

# Remove extra spaces from text data
for col in df.select_dtypes(include="object").columns:
    df[col] = df[col].str.strip()

# Convert columns to correct data types
df["Date"] = pd.to_datetime(
    df["Date"], errors="coerce"
)

numeric_cols = [
    "Temperature",
    "Humidity",
    "WindSpeed",
    "Precipitation"
]

for col in numeric_cols:
    df[col] = pd.to_numeric(df[col], errors="coerce")

# ==========================================
# STEP 2: DISPLAY ORIGINAL DATA
# ==========================================

print("\n========== WEATHER ANALYSER ==========")

print("\n1. ORIGINAL WEATHER DATA")
print(df.to_string(index=False))

# ==========================================
# STEP 3: DATASET INFORMATION
# ==========================================

print("\n2. DATASET INFORMATION")
df.info()

print("\n3. STATISTICAL SUMMARY")
print(df.describe().round(2))

print("\n4. MISSING VALUES")
print(df.isnull().sum())

# ==========================================
# STEP 4: WEATHER ANALYSIS
# ==========================================

print("\n5. WEATHER ANALYSIS")

print("Average Temperature:",
      round(df["Temperature"].mean(), 2))

print("Maximum Temperature:",
      df["Temperature"].max())

print("Minimum Temperature:",
      df["Temperature"].min())

print("Average Humidity:",
      round(df["Humidity"].mean(), 2))

print("Average Wind Speed:",
      round(df["WindSpeed"].mean(), 2))

print("Total Recorded Precipitation:",
      round(df["Precipitation"].sum(), 2))

print("\nWeather Condition Counts:")
print(df["Condition"].value_counts())

# Hottest day
if df["Temperature"].notna().any():
    hottest = df.loc[df["Temperature"].idxmax()]
    print("\nHottest Day:")
    print("Date:", hottest["Date"].date())
    print("Temperature:", hottest["Temperature"])

# ==========================================
# STEP 5: PREPARE GRAPHS
# ==========================================

sns.set_theme(style="whitegrid")

plot_df = df.dropna(subset=["Date"]).sort_values("Date")

fig, axes = plt.subplots(2, 2, figsize=(15, 10))

# Graph 1: Temperature Trend
sns.lineplot(
    data=plot_df,
    x="Date",
    y="Temperature",
    marker="o",
    ax=axes[0, 0]
)

axes[0, 0].set_title("Daily Temperature Trend")
axes[0, 0].set_xlabel("Date")
axes[0, 0].set_ylabel("Temperature")
axes[0, 0].tick_params(axis="x", rotation=45)

# Graph 2: Humidity Trend
sns.lineplot(
    data=plot_df,
    x="Date",
    y="Humidity",
    marker="o",
    ax=axes[0, 1]
)

axes[0, 1].set_title("Daily Humidity Trend")
axes[0, 1].set_xlabel("Date")
axes[0, 1].set_ylabel("Humidity (%)")
axes[0, 1].tick_params(axis="x", rotation=45)

# Graph 3: Daily Precipitation
sns.barplot(
    data=plot_df,
    x="Date",
    y="Precipitation",
    ax=axes[1, 0]
)

axes[1, 0].set_title("Daily Precipitation")
axes[1, 0].set_xlabel("Date")
axes[1, 0].set_ylabel("Precipitation")
axes[1, 0].tick_params(axis="x", rotation=90)

# Graph 4: Weather Conditions
sns.countplot(
    data=df,
    x="Condition",
    order=df["Condition"].value_counts().index,
    ax=axes[1, 1]
)

axes[1, 1].set_title("Weather Condition Distribution")
axes[1, 1].set_xlabel("Weather Condition")
axes[1, 1].set_ylabel("Number of Days")

# ==========================================
# STEP 6: SAVE AND DISPLAY GRAPHS
# ==========================================

plt.tight_layout()

output_path = os.path.join(
    os.path.dirname(file_path),
    "weather_analysis_graphs.png"
)

plt.savefig(output_path, dpi=300, bbox_inches="tight")

print("\nGraphs saved successfully at:")
print(output_path)

print("\nDisplaying graphs...")

plt.show()
