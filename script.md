Hi, I'm Hiroaki Yutani. I'm a software engineer at MIERUNE, Inc. MIERUNE is a GIS company in the Northern part of Japan.

Today, I talk about elephant, duck, and mountain. Or, more precisely, these three GIS databases; PostGIS, DuckDB, and SedonaDB.

The background is that I've been contributing to SeodnaDB and DuckDB spatial extension. But, I myself have been wondering which one is the best.

First, let's talk about the overview.

PostGIS. PostGIS is a PostgreSQL extension. Since PostGIS is the pioneer of the `ST_` functions, it provides more versatile functions than the others. It can handle both vector and raster.

The downside is that we need to set up a server and import data before doing anything. While having a server is a good thing for transactional workloads, PostGIS is not lightweight as DuckDB and SedonaDB.

Next, DuckDB. DuckDB is extremely portable. The CLI is a single binary. We can run it right after downloading it, without installation. It runs anywhere. macOS, Windows, Linux, and even web browsers. There are bindings for various programming languages such as Python and R.

While DuckDB is a general-purpose tool, the official `spatial` extension provides various spatial functions.

One thing to note is that the spatial extension can handle only vector data. There are community extensions for raster data, but it's not integrated with the core.

Here's two hot news about DuckDB. First, version 1.5 was released a while ago, and DuckDB now has native `GEOMETRY` type! I won't go into the details, but this is huge. Let's revisit here later.

Second, DuckDB is announced to be tuned for server usages. This means we'll be able to use DuckDB both for analytical usages and transactional usages. But, version 2.0 is not released yet.

Last one, SedonaDB. SedonaDB is a query engine designed for GIS. It is available mainly in the form of a Python module and an R package. SedonaDB supports both vector and raster, although raster support is under development!

Before SedonaDB, there's Sedona, a distributed geospatial processing on Apache Spark. SedonaDB is developped as a single node version of Sedona, so that we can get familiar with the API. Having the same API means we can prototype locally and then scale in the cloud.

So, here's the short summary. They all have support for raster data, but DuckDB's one is not integrated with the core. DuckDB and SedonaDB can run without the server setup. If we want a server, in other words, for transactional workloads, PostGIS is a great choice.

So, which one should I choose? I would recommend... DuckDB!

Yes, I think DuckDB is a safe bet. What I think makes DuckDB different is its portability. We can develop the query locally, and execute it almost anywhere. No server is needed. No installation is needed. Just download the server and run the same SQL.

Also, its spatial extension has enough functions to handle almost any vector operations.

Internally, it uses GDAL. So, it can read any data format that GDAL can read.

Needless to say, DuckDB delivers excellent performance, especially with Parquet files.

But..., DuckDB is not almighty. Let me pick one example.

Overture Maps' official document shows how to access their data with DuckDB. Here's the SQL.

In this SQL, have you ever seen such conditions? What's this `bbox` column?

If you are familiar with PostGIS, you would expect a condition on `geometry` column like this. `&&`(double ampersand) is the trick to use the spatial index. By putting this before the actual `ST_Intersects()` condition, we can speed up the query by pruning the features that are apparently outside the bbox.

So, why does the DuckDB SQL look very different? Can't we do the same thing? 

Good news! As I talked in the previous slide, DuckDB version 1.5 acquired the functionality! Now that the `GEOMETRY` type is native in DuckDB, DuckDB can do filter pruning more efficiently.

We can use `&&` in the same way as PostGIS. Internally, it is an alias to `ST_Intersects_Extent()`, which is a lightweight version of `ST_Intersects()`. By using this, theoretically, we should be able to replace the `bbox` condition with double ampersand.

But, here's a bad news. It doesn't work. If we remove the `bbox` condition, DuckDB scans the full data.

This is a bit complicated issue, but, in short, it is that DuckDB can optimize the execution only when the necessary statistics is available, and Overture Maps' Parquet files don't provide the statistics.

On the other hand, SedonaDB can do! It doesn't even need the double ampersand. We can just use `ST_Intersects()`, then SedonaDB can take care of the rest.

But, why does SedonaDB work while DuckDB fails? Actually, SedonaDB does the same calculation as the `bbox` condition internally. `bbox` column, or `convering` metadata is defined by GeoParquet 1.1 specification. The `covering` metadata refers to a column name that contains the bounding box information, and the column is `bbox` in this specific case.

So, this bbox condition was just a workaround because DuckDB doesn't automatically check `bbox` column! On the other hand, SedonaDB knows this.

To be clear, this should be just a temporary problem until the new version of the GeoParquet specification. Once it's standardized, Overture Maps can provide the data with row group statistics. Then, DuckDB should have no problem with utilizing it for filter pruning.

But, what I would emphasize is, there are cases when the engine itself needs to be aware of the data format designed for geospatial.
