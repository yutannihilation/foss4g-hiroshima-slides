---
title: "SedonaDB: Why Yet Another Geospatial Database Engine?"
theme: default
background: /background.png
fonts:
  sans: Nebula Sans
  mono: Fira Code
  weights: 600,900
  # Nebula Sans is self-hosted from ./fonts, so don't fetch it from Google Fonts
  local: Nebula Sans
class: text-center
mdc: true
favicon: /favicon.ico
# seoMeta:
#  ogImage: https://cover.sli.dev
---

# SedonaDB:<br>Why Yet Another<br>Geospatial Database Engine?

2026/09/01

Hiroaki Yutani

---

# So you want SQL on GIS data?

<div class="flex justify-center items-center gap-16 h-4/5 text-8xl">
<v-clicks>
    <div class="h-40 w-40">🐘</div>
    <div class="h-40 w-40">🦆</div>
    <div class="h-40 w-40">⛰️</div>
</v-clicks>
</div>

---

# So you want SQL on GIS data?

<div class="flex justify-center items-center gap-16 h-4/5">
    <img src="/src/postgis-logo-horizontal.png" class="h-40 object-contain" />
    <img src="/src/DuckDB_icon-lightmode.svg" class="h-40 object-contain" />
    <img src="/src/sedona_logo_symbol.png" class="h-40 object-contain" />
</div>

---

# 🐘PostGIS

<img src="/src/postgis-logo-horizontal.png" class="float-right -mt-16 ml-8 h-30 object-contain" />

- A PostgreSQL extension
- Versatile spatial functions!
  - This is the de facto standard...
- Supports both vector and raster data
- Requires PostgreSQL server setup

---

# 🦆DuckDB

<img src="/src/DuckDB_icon-lightmode.svg" class="float-right -mt-16 ml-8 h-30 object-contain" />

- Highly portable
  - Single binary
  - Runs everywhere (older machines, web browsers, Python, R, etc)
- `spatial` extension provide spatial functions
- No native raster support (but a community extension is available)

---

# 🦆What’s New in DuckDB

<img src="/src/DuckDB_icon-lightmode.svg" class="float-right -mt-16 ml-8 h-30 object-contain" />

- Native `GEOMETRY` type (v1.5🎉)
  - We'll talk about this later!
- "DuckDB as a server" (v2.0, soon-ish)
  - Handle transactional workloads

---

# ⛰️SedonaDB

<img src="/src/sedona_logo_symbol.png" class="float-right -mt-16 ml-8 h-30 object-contain" />

- A query engine designed for GIS
- Primarily used from Python or R (a CLI is also available)
- Supports both vector and raster data (raster support is under development)

---

# ⛰️SedonaDB vs Sedona

<img src="/src/sedona_logo_symbol.png" class="float-right -mt-16 ml-8 h-30 object-contain" />

- **Sedona:** Distributed geospatial processing on Apache Spark
- **SedonaDB:** The same API, optimized for a single node
- Prototype locally, then scale in the cloud

---

# Summary

|   | Vector | Raster | w/o server |
|:--|:------:|:------:|:----------:|
| PostGIS | ✅ | ✅  |   |
| DuckDB | ✅ | ⚠️  | ✅  |
| SedonaDB | ✅ | ✅  | ✅  |

<small class="block mt-2 text-left text-xl text-gray-500 font-semibold">
⚠️ Community extension
</small>

---
clicks: 1
---

# So, which DB should I use??

<div class="flex justify-center items-center h-4/5">
  <EmojiRoulette :emojis="['🐘', '🦆', '⛰️']" final="🦆" />
</div>

---

# DuckDB is a safe bet

- Write once, run anywhere
  - No server, just download the CLI!
- `spatial` extension provides most of the vector operations
- Very performant on Parquet

---

# But...

---

# Overture Maps' official document

![](/src/screenshot1.png)

---

# Overture Maps' official document

![](/src/screenshot2.png)

---

# What's this...?

```sql {7-8}
...snip...
 
  FROM
    read_parquet('s3://overturemaps-us-west-2/release/2026-08-19.0/theme=places/type=place/*', filename=true, hive_partitioning=1)
  WHERE
    categories.primary = 'pizza_restaurant'
    AND bbox.xmin BETWEEN -75 AND -73       -- Only use the bbox min values
    AND bbox.ymin BETWEEN 40 AND 41         -- because they are point geometries.
) TO 'nyc_pizza.geojson' WITH (FORMAT GDAL, DRIVER 'GeoJSON');
```

---

# Equivalent SQL in PostGIS

```sql
-- filter by bbox
AND geometry &&
  ST_MakeEnvelope(-75, 40, -73, 41, 4326)
-- check intersection precisely
AND ST_Intersects(
  geometry,
  ST_MakeEnvelope(-75, 40, -73, 41, 4326)
)
```

---

# Good news

- As of DuckDB v1.5, `&&` does filter pruning!
  - `&&` is an alias to `ST_Intersects_Extent()`
- So, we can use `&&` instead of the conditions on `bbox` column, in theory...?


---

# Bad news

- If we change the SQL to `&&`, it takes forever...

<img src="/src/screenshot3.png" class="mx-auto mt-6 w-4/5 max-h-72 object-contain" />

---

# Why?

- Overture Maps' GeoParquet does not provide row-group statistics on `GEOMETRY` column yet.
- 