# 🏨 AtliQ Hotels Data Analysis Project

## 📌 Project Overview

AtliQ Grands is a premium 5-star hotel chain operating across 4 major cities in India — **Delhi, Mumbai, Bangalore, and Hyderabad**. Despite being in the hospitality industry for over 20 years, the company has been losing its market share and revenue due to competitors' strategic moves and ineffective decision-making by management.

To tackle this, the managing director decided to incorporate **Business & Data Intelligence** by hiring a 3rd party data analyst to extract meaningful insights from their historical booking data.

**This project covers:**
- Exploratory Data Analysis (EDA) on 1,34,590 booking records
- Data Cleaning — handling invalid entries, outliers, and null values
- Data Transformation — creating occupancy percentage metric
- Insight Generation — revenue, occupancy, ratings across cities, hotels, and platforms
---

## 🎯 Project Objective

- Analyze historical booking data of AtliQ Grands hotels
- Identify key factors affecting **revenue and occupancy rate**
- Compare performance across **cities, room types, and booking platforms**
- Find difference between **weekday vs weekend** booking trends
- Generate actionable insights to help management take **data-driven decisions**
- Help AtliQ Grands **regain its market share** in luxury/business hotel segment
---

## 🗂️ Dataset Information

This project contains **5 CSV files:**

| File | Rows | Description |
|------|------|-------------|
| `dim_hotels.csv` | 25 | Property ID, hotel name, category (Luxury/Business), city |
| `dim_rooms.csv` | — | Room ID and room class (Standard, Double, Premium, Presidential) |
| `dim_date.csv` | 92 | Date, month, week number, day type (weekday/weekend) |
| `fact_bookings.csv` | 1,34,590 | Booking ID, check-in/out dates, guests, platform, status, revenue |
| `fact_aggregated_bookings.csv` | 9,200 | Property ID, room category, successful bookings, capacity |

**Cities Covered:** Delhi, Mumbai, Bangalore, Hyderabad  
**Hotel Categories:** Luxury (16 properties), Business (9 properties)  
**Room Types:** RT1 (Standard), RT2 (Double), RT3 (Premium), RT4 (Presidential)  
**Time Period:** May 2022 — July 2022
---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| Python 3.10+ | Core programming language |
| Pandas | Data manipulation and analysis |
| Matplotlib | Data visualization and charts |
| NumPy | Numerical computation |
| Jupyter Notebook | Interactive development environment |
---

## 🧹 Data Cleaning Steps

**1. Invalid Guests Removal**
- Removed records where `no_guests <= 0` as these were data entry errors
- Records removed: 12
```python
df_fbooking = df_fbooking[df_fbooking.no_guests > 0]
```

**2. Revenue Outlier Removal**
- Used **Mean ± 3×Standard Deviation** method to find upper limit
- Upper limit came out to be ≈ ₹2,94,499
- Removed 5 extreme outlier records
```python
higher_limit = df_fbooking.revenue_generated.mean() + 3 * df_fbooking.revenue_generated.std()
df_bookings = df_fbooking[df_fbooking.revenue_generated <= higher_limit]
```

**3. Null Values Treatment**
- `capacity` column in aggregated bookings had 2 null values
- Filled with **median value (25.0)**
```python
df_agg_bookings['capacity'] = df_agg_bookings['capacity'].fillna(
    df_agg_bookings['capacity'].median()
)
```

**4. Over-Capacity Records Removal**
- Removed 6 records where `successful_bookings > capacity` as these were invalid entries
```python
df_agg_bookings = df_agg_bookings[
    df_agg_bookings.successful_bookings <= df_agg_bookings.capacity
]
```

**5. Ratings Column**
- `ratings_given` column had 77,897 null values
- These were **not removed or filled** as most cancelled/no-show bookings naturally have no rating

## 🔄 Data Transformation

**1. Occupancy Percentage Column**
- Created new column `occ_pct` to measure how much capacity is being utilized
- Formula: `successful_bookings / capacity × 100`
```python
df_agg_bookings['occ_pct'] = (
    df_agg_bookings['successful_bookings'] / df_agg_bookings['capacity']
).apply(lambda x: round(x * 100, 2))
```

**2. Merging Datasets**
- Merged aggregated bookings with hotels data to get city and category info
```python
df_hotel = pd.merge(df_agg_bookings, df_hotels, on="property_id")
```
- Merged with date table to get month and day type info
```python
df_again = pd.merge(df_hotel, df_date, left_on="check_in_date", right_on="date")
```
- Merged fact bookings with hotels data for revenue analysis
```python
df_bookings_all = pd.merge(df_bookings, df_hotels, on="property_id")
```

**3. Date Column Conversion**
- Converted date columns from string to datetime format for proper merging
```python
df_date["date"] = pd.to_datetime(df_date["date"])
df_bookings_all["check_in_date"] = pd.to_datetime(
    df_bookings_all["check_in_date"], dayfirst=True, format="mixed"
)
```

**4. New Data Append**
- Appended August 2022 new data to existing dataframe
```python
latest_df = pd.concat([df_again, df_august], ignore_index=True, axis=0)
```
## 💡 Business Questions & Insights

---

**Q1. What is the average occupancy rate in each room category?**

```python
df_agg_bookings.groupby("room_category")["occ_pct"].mean()
```

| Room Type | Description | Avg Occupancy % |
|-----------|-------------|----------------|
| RT1 | Standard | 57.89% |
| RT2 | Double | 58.01% |
| RT3 | Premium | 58.03% |
| RT4 | Presidential | **59.28%** |

> 📌 Presidential suites (RT4) have the highest occupancy rate across all room types.

---

**Q2. What is the average occupancy rate per city?**

```python
df_hotel = pd.merge(df_agg_bookings, df_hotels, on="property_id")
df_hotel.groupby("city")["occ_pct"].mean()
```

| City | Avg Occupancy % |
|------|----------------|
| **Delhi** | **61.51%** |
| Hyderabad | 58.12% |
| Mumbai | 57.91% |
| Bangalore | 56.33% |

> 📌 Delhi has the highest occupancy rate among all cities.

---

**Q3. When was the occupancy better — Weekday or Weekend?**

```python
df_again.groupby("day_type")["occ_pct"].mean().round(2)
```

| Day Type | Avg Occupancy % |
|----------|----------------|
| **Weekend** | **72.34%** |
| Weekday | 50.88% |

> 📌 Weekend occupancy is 42% higher than weekdays. Dynamic pricing strategy should be applied on weekends.

---

**Q4. In the month of June, what is the occupancy for different cities?**

```python
df_june_22 = df_again[df_again["mmm yy"] == "Jun 22"]
df_june_22.groupby('city')['occ_pct'].mean().round(2).sort_values(ascending=False)
```

| City | June Occupancy % |
|------|-----------------|
| **Delhi** | **62.47%** |
| Hyderabad | 58.46% |
| Mumbai | 58.38% |
| Bangalore | 56.44% |

> 📌 Delhi leads in June occupancy as well, while Bangalore remains the lowest.

---

**Q5. What is the revenue realized per city?**

```python
df_bookings_all = pd.merge(df_bookings, df_hotels, on="property_id")
df_bookings_all.groupby("city")["revenue_realized"].sum()
```

| City | Revenue Realized |
|------|-----------------|
| **Mumbai** | **₹66.86 Cr** |
| Bangalore | ₹42.04 Cr |
| Hyderabad | ₹32.52 Cr |
| Delhi | ₹29.44 Cr |

> 📌 Mumbai generates the highest revenue despite not having the highest occupancy rate.

---

**Q6. What is the month by month revenue?**

```python
df_bookings_all_rev.groupby("mmm yy")["revenue_realized"].sum()
```

| Month | Revenue |
|-------|---------|
| May 22 | ₹58.18 Cr |
| Jun 22 | ₹55.39 Cr |
| Jul 22 | ₹57.28 Cr |
| **Total** | **₹170.85 Cr** |

> 📌 May 2022 was the highest revenue month. June saw a slight dip but July recovered.

---

**Q7. What is the revenue realized per hotel?**

```python
df_bookings_all_rev.groupby("property_name")["revenue_realized"].sum().sort_values(ascending=False)
```

| Hotel | Revenue Realized |
|-------|-----------------|
| **Atliq Exotica** | **₹32.03 Cr** |
| Atliq Palace | ₹30.41 Cr |
| Atliq City | ₹28.58 Cr |
| Atliq Blu | ₹26.09 Cr |
| Atliq Bay | ₹26.00 Cr |
| Atliq Grands | ₹21.15 Cr |
| Atliq Seasons | ₹6.61 Cr |

> 📌 Atliq Exotica is the top revenue generating hotel. Atliq Seasons needs serious attention as it is the lowest performer.

---

**Q8. What is the average rating per city?**

```python
df_bookings_all_rev.groupby("city")["ratings_given"].mean().round(2)
```

| City | Avg Rating |
|------|-----------|
| **Delhi** | **3.78** |
| Hyderabad | 3.66 |
| Mumbai | 3.65 |
| Bangalore | 3.41 |

> 📌 Delhi has the best customer ratings. Bangalore has the lowest ratings which may be contributing to its lower occupancy.

---

**Q9. What is the revenue contribution per booking platform?**

```python
df_bookings_all_rev.groupby("booking_platform")["revenue_realized"].sum()
```

| Platform | Revenue |
|----------|---------|
| others | ₹69.93 Cr |
| makeyourtrip | ₹34.08 Cr |
| logtrip | ₹18.75 Cr |
| direct online | ₹16.89 Cr |
| tripster | ₹12.31 Cr |
| journey | ₹10.25 Cr |
| direct offline | ₹8.64 Cr |

> 📌 "Others" platform contributes the highest revenue. Direct offline is the lowest — digital presence needs improvement.

## 📊 Conclusion

This project successfully analyzed AtliQ Grands historical booking data from **May 2022 to July 2022** covering **7 hotels across 4 cities** with over **1.34 lakh booking records**.

**Key Findings:**

- **Delhi** is the best performing city with highest occupancy (61.51%) and highest customer ratings (3.78)
- **Mumbai** generates the highest revenue (₹66.86 Cr) despite not having the highest occupancy rate
- **Weekend occupancy (72.34%)** is significantly higher than weekday occupancy (50.88%) — a 42% difference
- **RT4 Presidential Suite** has the highest occupancy rate (59.28%) among all room types
- **Atliq Exotica** is the top revenue generating hotel (₹32.03 Cr) while **Atliq Seasons** is the lowest (₹6.61 Cr)
- **Bangalore** has the lowest occupancy and lowest customer ratings (3.41) — needs immediate attention
- Total revenue generated in 3 months: **₹170.85 Crore**

**Business Recommendations:**

- Implement **dynamic pricing on weekends** to maximize revenue during high demand days
- Focus on **improving service quality in Bangalore** to boost ratings and occupancy
- Investigate why **Atliq Seasons** is underperforming and take corrective actions
- Strengthen **direct booking channels** as direct offline contributes the least revenue
- Deep dive into **"Others" booking platform** to identify and strengthen that revenue stream
