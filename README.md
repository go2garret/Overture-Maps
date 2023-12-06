# Overture-Maps
Download the Overture Maps geospatial data repository (See https://overturemaps.org/)

<h1>
  Introduction
</h1>
<p>
  In this document, we will explore how to access and download datasets provided by Overture Maps (see https://overturemaps.org/). The Overture Maps datasets contain millions of geospatial features of businesses and other places, with detailed information about the place and spatial coordinates for the place. <br>We will be looking at the Places dataset provided by Overture, however there are other datasets available (buildings, etc). Happy data hunting!
</p>

# DuckDB
<p>
  We will be using DuckDB to access the data from Overturemaps, which is provided in Parquet files. DuckDB is a free SQL editor that handles parquet files. Get the latest version here: https://duckdb.org/. Download the CLI (command line tool). Within the CLI, we can easily run our SQL statements by copying an pasting from this document into the CLI tool. These statements are intended to be enough to get you started and could be modified.
</p>
<p>
  Reminder, always check for the latest parquet file releases from the Overture Maps website and reference the newest files in the statements below.
</p>

# Querying the Data
<h3>
  Simple
</h3>
<p>
      SELECT *<br>
      FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=places/type=*/*', filename=true, hive_partitioning=1)<br>
       WHERE json_extract_string(json_extract(addresses::json, '$[0]'), '$.country') = 'US' AND<br>
       json_extract_string(json_extract(addresses::json, '$[0]'), '$.region') = 'FL'<br>
       limit 10
</p>
<h3>
  Download into file 
</h3>
<p>
  COPY (
<br>     SELECT *
<br>     FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=places/type=*/*', filename=true, hive_partitioning=1)
<br>        WHERE json_extract_string(json_extract(addresses::json, '$[0]'), '$.country') = 'US' AND
<br>        json_extract_string(json_extract(addresses::json, '$[0]'), '$.region') = 'FL'
<br>        limit 10
<br> ) TO 'c:/temp/places_fl.csv' WITH (FORMAT CSV);
</p>


