Hi, I'm Hiroaki Yutani. I'm a software engineer at MIERUNE, Inc. MIERUNE is a GIS company in the Northern part of Japan.

Today, I talk about elephant, duck, and mountain. Or, more precisely, these three GIS databases; PostGIS, DuckDB, and SedonaDB.

The background is that I've been contributing to SeodnaDB and DuckDB spatial extension. But, I myself have been wondering which one is the best.

First, let's talk about the overviews.

PostGIS. PostGIS is a PostgreSQL extension. Since PostGIS is the pioneer of the `ST_` functions, it provides more versatile functions than the others. It can handle both vector and raster.

The downside is that we need to set up a server and import data before doing anything. While having a server is a good thing for transactional workloads, PostGIS is not lightweight as DuckDB and SedonaDB.

Next, DuckDB. DuckDB is extremely portable. The CLI is a single binary, and runs anywhere. macOS, Windows, Linux, and web browsers.
