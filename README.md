# Snowflake Weather Data Pipeline

This repository documents what I learned while building a simple data ingestion pipeline in **Snowflake**, using an external stage pointed to an **S3 bucket**, ingesting **JSON weather data**, transforming it, and loading it into a structured table.

---

## 📌 Overview

This project demonstrates:

* Creating a **Snowflake stage**
* Loading raw JSON files into a **VARIANT** column table
* Querying semi‑structured JSON data using the `:` operator
* Transforming JSON into structured columns
* Copying transformed results into a final analytics‑ready table

---

## 🏗️ 1. Create a Stage

A stage stores references to external files. Here, we connect Snowflake to an S3 bucket containing NYC weather data.

```sql
CREATE OR REPLACE STAGE DEMO.DEMO_SCHEMA.weather_stage
  URL = 's3://snowflake-workshop-lab/weather-nyc'
  FILE_FORMAT = (TYPE='json');
```

---

## 🧱 2. Create Raw Weather Table

We create a table with a single `VARIANT` column to hold raw semi‑structured JSON.

```sql
CREATE OR REPLACE TABLE DEMO.DEMO_SCHEMA.WEATHERTABLE (
  data VARIANT
);
```

---

## 📥 3. Load JSON Data Into the Raw Table

```sql
COPY INTO DEMO.DEMO_SCHEMA.WEATHERTABLE
FROM @DEMO.DEMO_SCHEMA.weather_stage;
```

Check that data loaded correctly:

```sql
SELECT *
FROM DEMO.DEMO_SCHEMA.WEATHERTABLE;
```

---

## 🔍 4. Querying JSON Fields

Using Snowflake's semi‑structured data functions, we can extract nested fields:

```sql
SELECT
  data:city:findname,
  data:city:coord:lat,
  data:city:coord:lon,
  data:clouds:all,
  data:main:humidity,
  data:main:pressure,
  data:main:temp,
  data:time,
  data:weather[0]:main
FROM WEATHERTABLE;
```

---

## 📊 5. Create Final Structured WEATHER Table

We now create an analytics‑ready table with typed columns.

```sql
CREATE OR REPLACE TABLE DEMO.DEMO_SCHEMA.WEATHER (
  CITYNAME STRING,
  LAT FLOAT,
  LON FLOAT,
  CLOUDS INTEGER,
  HUMIDITY INTEGER,
  PRESSURE FLOAT,
  TEMP FLOAT,
  TIME TIMESTAMP,
  WEATHER STRING
);
```

---

## 🚀 6. Load Transformed Data into Final Table

We extract specific JSON fields and load them directly into the final table.

```sql
COPY INTO DEMO.DEMO_SCHEMA.WEATHER
FROM (
  SELECT
    t.$1:city:findname,
    t.$1:city:coord:lat,
    t.$1:city:coord:lon,
    t.$1:clouds:all,
    t.$1:main:humidity,
    t.$1:main:pressure,
    t.$1:main:temp,
    t.$1:time,
    t.$1:weather[0]:main
  FROM @DEMO.DEMO_SCHEMA.weather_stage t
);
```

View final structured table:

```sql
SELECT *
FROM DEMO.DEMO_SCHEMA.WEATHER;
```

---

## 📚 Summary of What I Learned

* How to configure **Snowflake stages** for external storage (S3)
* Working with **semi‑structured JSON** using the `VARIANT` type
* Extracting nested fields with Snowflake's `:` and array indexing
* Using `COPY INTO` to load both raw and transformed data
* Converting raw JSON into a **clean analytical model**
* Understanding Snowflake's flexible schema handling for data engineering

---

## 🚀 Next Steps

* Automate ingestion with **Snowpipe**
* Add **tasks** & **streams** for incremental loads
* Build a dashboard using Tableau or Power BI

---

Feel free to clone or build on this project! Let me know if you want help expanding this pipeline.
