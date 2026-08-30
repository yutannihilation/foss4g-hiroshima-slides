Hi, I'm Hiroaki Yutani. I'm a software engineer at MIERUNE, Inc. MIERUNE is a GIS company in the northern part of Japan.

Today, I'll talk about an elephant, a duck, and a mountain. Or, more precisely, these three GIS databases: PostGIS, DuckDB, and SedonaDB.

Let me talk a bit about the background. I've been contributing to SedonaDB, but I'm also a contributor to the DuckDB spatial extension. I'm the eighth contributor in SedonaDB's repo, and the third in DuckDB's repo. But, while contributing to multiple projects, I myself was wondering if it's really meaningful to have multiple geospatial databases. Isn't it enough to have the most powerful one? Why do we need yet another geospatial database engine? This is the question I've been thinking about for a year. Honestly, I don't have a clear answer yet, but let me share my thoughts.

Okay, let's start with brief introductions to the three databases I showed.

First, PostGIS. PostGIS is a PostgreSQL extension. Since PostGIS is the pioneer of the `ST_` functions, it provides more versatile functions than the others. PostGIS is the standard for SQL on GIS data. It can handle both vector and raster.

The downside is that we need to set up a server before doing anything. While having a server is a good thing for transactional workloads, PostGIS is not as lightweight as DuckDB and SedonaDB. In addition, we need to import the data into the server. If the data is small enough, it's not a problem, but it's impossible if the data is larger than the available disk space.

Next, DuckDB. DuckDB is extremely portable. The CLI is a single binary. We can run it right after downloading it, without installation. It runs anywhere. macOS, Windows, Linux, and even web browsers. There are also bindings for various programming languages such as Python and R.

While DuckDB is a general-purpose database, the `spatial` extension turns it into a versatile tool for geospatial operations. It implements most of the `ST_` functions that PostGIS provides.

One thing to note is that the spatial extension can handle only vector data. There are community extensions for raster data, but they're not really integrated with the core.

Let me talk a bit about DuckDB's recent updates. First, DuckDB now has a native `GEOMETRY` type. This happened in version 1.5, which was released in May. Before this, the `GEOMETRY` type was an opaque binary type to the core, but today, the core understands `GEOMETRY`. I won't go into the details, but this is huge. Let's revisit this later.

The second piece of news is that DuckDB 2.0 will significantly improve the ergonomics of server-side use. This means we'll be able to use DuckDB not only for analytical workloads but also for transactional workloads. DuckDB 2.0 is scheduled to be released this fall.

Last one, SedonaDB. How many of you know SedonaDB already? SedonaDB is a query engine designed for GIS. It is available mainly in the form of a Python module and an R package. At its core, SedonaDB is implemented in the Rust programming language, so it should be available in other forms as well.

SedonaDB supports both vector and raster, although raster support is under development!

Before SedonaDB, there was already Sedona, a distributed geospatial processing system on Apache Spark. SedonaDB is developed as a single-node version of Sedona, so that we can get familiar with the API. Having the same API means we can prototype locally and then scale in the cloud.

So, here's the short summary. They all have support for raster data, but DuckDB's raster support is not integrated with the core. DuckDB and SedonaDB can run without setting up a server. If we want a server, in other words, for transactional workloads, PostGIS is a great choice. But, DuckDB 2.0 might be a game changer!

So, which one should I choose? I would recommend... DuckDB!

Yes, I think DuckDB is a safe bet. What I think makes DuckDB different is its portability. We can develop the query locally, and execute it almost anywhere. No server is needed. No installation is needed. Just download the binary and run the same SQL.

Also, its spatial extension has enough functions to handle almost any vector operation.

Internally, it uses GDAL. So, it can read any data format that GDAL can read.

Needless to say, DuckDB delivers excellent performance, especially with Parquet files.

But..., DuckDB is not almighty. Let me share one minor problem, which was too tricky for me to understand.

Do you know Overture Maps? It's a collaborative project that provides free and open map data for the entire world. It combines data from multiple sources and includes buildings, places, addresses, transportation networks, and more.
The official Overture Maps documentation shows how to access the data using DuckDB. Here's the SQL query.

In this SQL, have you ever seen such conditions? What's this `bbox` column?

If you are familiar with PostGIS, you would expect a condition on the `geometry` column like this. `&&` (double ampersand) is the trick to use the spatial index. By putting this before the actual `ST_Intersects()` condition, we can speed up the query by pruning the features that are apparently outside the area of interest.

So, why does the DuckDB SQL look very different? Can't we do the same thing? 

Good news! As I mentioned on the previous slide, DuckDB version 1.5 acquired the functionality! Now that the `GEOMETRY` type is native in DuckDB, DuckDB can do filter pruning more efficiently.

We can use `&&` in the same way as PostGIS. Internally, it is an alias to function `ST_Intersects_Extent()`, which is a lightweight version of `ST_Intersects()`. By using this, theoretically, we should be able to replace the `bbox` condition with double ampersand.

But, here's a bad news. It doesn't work. If we remove the `bbox` condition, DuckDB scans the full data. No filter pruning happens here.

This is a bit of a complicated issue, but, in short, DuckDB can optimize the execution only when the necessary statistics are available, and Overture Maps' Parquet files don't provide the statistics.

On the other hand, SedonaDB can handle this nicely! We don't even need to write the double ampersand condition. We can just use `ST_Intersects()`, and then SedonaDB can take care of the rest.

You can also use the data frame API instead of SQL.

But, why does SedonaDB work while DuckDB fails? Actually, SedonaDB does the same calculation as the `bbox` condition internally. The `bbox` column, or `covering` metadata, is defined by the GeoParquet 1.1 specification. The `covering` metadata refers to a column name that contains the bounding box information, and the column name is `bbox` in this specific case.

This is very tricky so I'm not sure if I can explain this well, but let me try anyway...!

In a Parquet file, the table data is split into many chunks called "row groups."

Each row group has some metadata, including statistics about the values in each column in the chunk. For example, the maximum value and the minimum value. So, if a database engine is smart enough, it can use the statistics to determine if the chunk is worth reading.

To be clear, this figure just shows the concept and doesn't reflect the actual data layout. In reality, the metadata section is located at the end of the file.

Anyway, this is how filter pruning works.

However, the GEOMETRY type in GeoParquet v1.1 doesn't provide the statistics. This is because, from the viewpoint of the Parquet specification, this GEOMETRY is just an opaque binary.

So, we are in trouble. If the statistics are not available, the engine cannot perform filter pruning. What should we do?

Here's the trick. We can add a column that provides the statistics for the geometry column. We are not interested in the contents of the column. Only the statistics are used.

Yes, this is the bbox column. I'm not sure if this is the original intention of the bbox column, but it seems people find the metadata useful instead of the content.

So, now we can understand what was happening in DuckDB's case. This bbox condition was just a workaround. Since DuckDB doesn't automatically check the `bbox` column, we need to check it manually! On the other hand, SedonaDB knows what to do with it.

To be clear, this should be just a temporary problem until the new version, 2.0, of the GeoParquet specification. Once it's standardized, Overture Maps can adopt it and provide the data with row group statistics. Then, DuckDB should have no problem with utilizing it for filter pruning.

But, what I would emphasize is, sometimes there are cases like this when the engine itself needs to be aware of the data format, needs to know the geospatial things.

Extensions are powerful, but they don't help much here. They have little control over the planner and the optimizer because this usually happens before the extension reads and processes the data. Again, the engine itself needs to address the geospatial things.

That's why we need a query engine designed for GIS big data.

To be honest, this is beyond my knowledge, and small data is usually fine. But I think this matters when filtering a large GIS dataset or joining multiple large GIS datasets. Spatial joins are especially challenging because they need to compare two large sets of geometries.

SedonaDB is working on this problem at the engine level. Version 0.3 introduced out-of-core spatial joins, which can process data larger than memory. Version 0.4 added GPU-accelerated spatial joins. Version 0.4.1 added index-optimized raster–vector spatial joins for `RS_Intersects`, `RS_Contains`, and `RS_Within`. This is another good example of why engine-level support matters. DuckDB's raster support is provided by a community extension and is not integrated into the core, so it is difficult for the engine to understand and optimize a join between vector and raster data. In SedonaDB, both vector and raster are part of the same query engine, so the engine can optimize the join directly. The feature is still new, and some difficult cross-CRS cases are being discussed, but I think these are good examples of why a query engine designed for GIS can make a difference.

So, let me come back to the question in the title. Why do we need yet another geospatial database engine? DuckDB is still a safe bet for most GIS workloads, and extensions can provide almost all the spatial functions we need. But extensions have limited control over query planning and optimization. For GIS big data, the engine itself needs to understand geospatial data. That is why we need SedonaDB.
