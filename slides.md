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

<img src="/src/postgis-logo-horizontal.png" class="absolute top-8 right-8 h-30 object-contain" />

- PostgreSQL で GIS データを扱える<br/>ようにする拡張
- 関数が豊富
  - PostGIS の挙動がデファクトスタンダード
- ベクターもラスターも扱える

---

# 🦆DuckDB

<img src="/src/DuckDB_icon-lightmode.svg" class="absolute top-8 right-8 h-30 object-contain" />

- ポータビリティに優れている
  - シングルバイナリ
  - ブラウザでも動く
  - 古いマシンでも動く
  - もちろん Python や R からも使える
- GIS 専用ではないが `spatial` 拡張がある
- v1.5 で`GEOMETRY`型が本体に入った

---

# ⛰️SedonaDB

<img src="/src/sedona_logo_symbol.png" class="absolute top-8 right-8 h-30 object-contain" />

- GIS のために設計されたクエリエンジン
- Python や R から使うのがメイン（CLI もある）
- Sedona（分散処理エンジン）と同じ API
  - ローカルでは重すぎる処理はクラウド上にスケールしたりできる
- ベクターもラスター（開発中）も扱える
- Wherobots が開発

---

# What is SedonaDB?

- Query engine designed for GIS

---

# DuckDB

|   | Vector | Raster | w/o server |
|:--|:------:|:------:|:----------:|
| PostGIS | ✅ | ✅  |   |
| DuckDB | ✅ |   | ✅  |
| SedonaDB | ✅ | ✅  | ✅  |

---

# a
