---
title: "SedonaDB: Why Yet Another Geospatial Database Engine?"
theme: default
layout: image
image: /cover.png
class: "!p-0"
fonts:
  sans: Nebula Sans
  mono: Fira Code
  weights: 600,900
  # Nebula Sans is self-hosted from ./fonts, so don't fetch it from Google Fonts
  local: Nebula Sans
mdc: true
favicon: /favicon.ico
# seoMeta:
#  ogImage: https://cover.sli.dev
---

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
layout: two-cols-header 
---

# My contributions

::left::

- SedonaDB (#8)

<img src="/src/github1.png" class="h-1/2" />

::right::

- DuckDB spatial (#3)

<img src="/src/github2.png" class="h-1/2" />

---

# 🐘PostGIS

<img src="/src/postgis-logo-horizontal.png" class="float-right -mt-16 ml-8 h-30 object-contain" />

<v-clicks>

- A PostgreSQL extension
- Pioneered the `ST_` function.
- Supports both vector and raster data
- Requires PostgreSQL server setup

</v-clicks>

---

# 🦆DuckDB

<img src="/src/DuckDB_icon-lightmode.svg" class="float-right -mt-16 ml-8 h-30 object-contain" />

<v-clicks>

- Highly portable
  - Single binary
  - Runs anywhere (older machines, web browsers, Python, R, etc)
- `spatial` extension provide spatial functions
- No native raster support (but a community extension is available)

</v-clicks>

---

# 🦆What’s New in DuckDB

<img src="/src/DuckDB_icon-lightmode.svg" class="float-right -mt-16 ml-8 h-30 object-contain" />

<v-clicks>

- Native `GEOMETRY` type (v1.5🎉)
  - We'll talk about this later!
- "DuckDB as a server" (v2.0, soon-ish)
  - Handle transactional workloads

</v-clicks>

---

# ⛰️SedonaDB

<img src="/src/sedona_logo_symbol.png" class="float-right -mt-16 ml-8 h-30 object-contain" />

<v-clicks>

- A query engine designed for GIS
- Primarily used from Python or R (a CLI is also available)
- Supports both vector and raster data (raster support is under development)

</v-clicks>

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
- Can read what GDAL can read
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

- In short, DuckDB can optimize the execution when the necessary statistics is available
- and it's not the case with Overture Maps' Parquet😢

---

# SedonaDB can do this

- We don't even need `&&`!

```sql
AND ST_Intersects(
  geometry,
  -- ST_MakeEnvelope() is not implemented yet
  ST_GeomFromWKT(
    'POLYGON((-75 40, -75 41, -73 41, -73 40, -75 40))'
  )
)
```

---

# But, why?

- Actually, SedonaDB does the same calculation as `bbox.xmin BETWEEN ...` internally for pruning.
- `bbox` column (or `covering` metadata) is defined by the GeoParquet 1.1 specification.

---

# DuckDB vs SedonaDB

- This was just a workaround because **DuckDB** doesn't automatucally check `bbox` column!

```sql
AND bbox.xmin BETWEEN -75 AND -73
AND bbox.ymin BETWEEN 40 AND 41
```

- On the other hand, **SedonaDB** does!🎉
```sql
AND ST_Intersects(
  geometry,
  ST_GeomFromWKT('POLYGON((-75 40, -75 41, -73 41, -73 40, -75 40))')
)
```

---

# DuckDB is not enough for GIS

- You can do almost everything in DuckDB with extensions.
- But, extensions have **little control over the planner and the optimizer**.


<v-click>
<div class="text-center pt-10 text-red-500 text-5xl">
We need a query engine<br/>designed for GIS big data!
</div>
</v-click>

---

# When does it matter?

- Small data is fine.
- Big data.
