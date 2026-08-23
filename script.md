Hi, I'm Hiroaki Yutani. I'm a software engineer at MIERUNE, Inc. MIERUNE is a GIS company in the Northern part of Japan.

Today, I talk about elephant, duck, and mountain. Or, more precisely, these three GIS databases; PostGIS, DuckDB, and SedonaDB.

The background is that I've been contributing to SeodnaDB and DuckDB spatial extension. But, I myself have been wondering which one is the best.

First, let's talk about the overview.

PostGIS. PostGIS is a PostgreSQL extension. Since PostGIS is the pioneer of the `ST_` functions, it provides more versatile functions than the others. It can handle both vector and raster.

The downside is that we need to set up a server and import data before doing anything. While having a server is a good thing for transactional workloads, PostGIS is not lightweight as DuckDB and SedonaDB.

Next, DuckDB. DuckDB is extremely portable. The CLI is a single binary. We can run it right after downloading it, without installation. It runs anywhere. macOS, Windows, Linux, and even web browsers. There are bindings for various programming languages such as Python and R.

While DuckDB is a general-purpose tool, the official `spatial` extension provides various spatial functions.

It can handle only vector data. There are community extensions for raster data, but it's not integrated with the core.

Last one, SedonaDB.
