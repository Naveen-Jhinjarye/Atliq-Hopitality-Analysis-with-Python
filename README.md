# Atliq-Hopitality-Analysis-with-Python
# 🏨 AtliQ Hotels Data Analysis Project

## 📌 Project Overview
AtliQ Grands ek multi-city luxury hotel chain hai jo India me 20+ saal se operate kar rahi hai.  
Recent time me competitors ki wajah se inka market share aur revenue decrease ho raha hai.

Is project ka goal hai:
- Historical data analyze karna
- Business insights nikalna
- Data-driven decisions lene me help karna

---

## 🎯 Objectives
- Occupancy rate analyze karna
- Revenue trends identify karna
- City-wise performance dekhna
- Booking platforms ka impact samajhna
- Weekday vs Weekend analysis

---

## 📂 Dataset Information

Project me 5 datasets use hue hain:

- `dim_date.csv`
- `dim_hotels.csv`
- `dim_rooms.csv`
- `fact_bookings.csv`
- `fact_aggregated_bookings.csv`

---

## 🛠️ Tools & Technologies

- Python 🐍
- Pandas
- NumPy
- Matplotlib
- Jupyter Notebook

---

## 🔍 Data Cleaning Steps

- Invalid guests remove kiye (`no_guests <= 0`)
- Revenue outliers remove kiye (3 standard deviation rule)
- Missing values handle kiye
- Capacity null values median se fill ki

---

## 🔄 Data Transformation

- Occupancy percentage (`occ_pct`) calculate kiya:
  
