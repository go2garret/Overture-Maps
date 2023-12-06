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
  Selecting the Places dataset by country for the entire United States.
</p>
<p>
      SELECT *<br>
      FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=places/type=*/*', filename=true, hive_partitioning=1)<br>
      WHERE json_extract_string(json_extract(addresses::json, '$[0]'), '$.country') = 'US'
</p>
<h3>
  Download into file 
</h3>
<p>
  Selecting by country and state.
</p>
<p>
  COPY (
<br>     SELECT *
<br>     FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=places/type=*/*', filename=true, hive_partitioning=1)
<br>        WHERE json_extract_string(json_extract(addresses::json, '$[0]'), '$.country') = 'US' AND
<br>        json_extract_string(json_extract(addresses::json, '$[0]'), '$.region') = 'CO'
<br> ) TO 'c:/temp/places_fl.csv' WITH (FORMAT CSV);
</p>
<h3>
  Inspect the file
</h3>
<p>
  The file contains many fields that are in JSON format, so we need to parse the fields.
</p>
<code>
  COPY (
    SELECT 
        json_extract_string(json_extract(names::json, '$.common[0]'), '$.value') AS name,
        json_extract_string(categories::json, '$.main') AS category,
        json_extract(categories::json, '$.alternate') AS category_alternates,
        confidence,
        websites,
        socials,
        emails,
        phones,
        json_extract_string(json_extract(addresses::json, '$[0]'), '$.locality') AS city,
        json_extract_string(json_extract(addresses::json, '$[0]'), '$.postcode') AS zipcode,
        json_extract_string(json_extract(addresses::json, '$[0]'), '$.freeform') AS freeform,
        json_extract_string(json_extract(addresses::json, '$[0]'), '$.country') AS country,
        json_extract_string(json_extract(brand::json, '$.names.brand_names_common[0]'), '$.value') AS brand_name,
        ST_GeomFromWkb(geometry) AS geometry
    FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=places/type=*/*', filename=true, hive_partitioning=1)
       WHERE json_extract_string(json_extract(addresses::json, '$[0]'), '$.country') = 'US' AND
       json_extract_string(json_extract(addresses::json, '$[0]'), '$.region') = 'FL'
) TO 'places_fl.csv' WITH (FORMAT CSV);
</code>


