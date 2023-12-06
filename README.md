# Overture-Maps
<p>Download the Overture Maps geospatial data repository provided by Microsoft and others. See <a href="https://overturemaps.org/" target="_blank">overturemaps.org/</a> for details.</p>

<h1>
  Introduction
</h1>
<p>
  In this document, we will explore how to access and download datasets provided by Overture Maps (<a href="https://overturemaps.org/" target="_blank">overturemaps.org/</a>). The Overture Maps datasets contain millions of geospatial features of businesses and other places, with detailed information about the place and spatial coordinates for the place. 
</p>
<p>
  We will be looking at the Places dataset provided by Overture, however there are other datasets available (buildings, etc)
</p>
<p>
  Happy data hunting!
</p>

# DuckDB
<p>
  We will be using DuckDB to access the data from Overturemaps, which is provided in Parquet files. DuckDB is a free SQL editor that handles parquet files. Get the latest version here: https://duckdb.org/. Download the CLI (command line tool). Within the CLI, we can easily run our SQL statements by copying an pasting from this document into the CLI tool. These statements are intended to be enough to get you started, so customize them for your needs.
</p>
<p>
  Reminder, always check for the latest parquet file releases from the Overture Maps website! Reference the newest files in the statements below.
</p>

# Querying the Data
<h3>
  Simple
</h3>
<p>
  Selecting the Places dataset by country for the entire United States. Do not run this, as we need to download the data (see next step).
</p>
<code>SELECT *
    FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=places/type=*/*', filename=true, hive_partitioning=1)
       WHERE json_extract_string(json_extract(addresses::json, '$[0]'), '$.country') = 'US'
</code>
<h3>
  Download into file 
</h3>
<p>
  Selecting by country and state and copy into file. 
</p>
<code>COPY (
    SELECT *
    FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=places/type=*/*', filename=true, hive_partitioning=1)
       WHERE json_extract_string(json_extract(addresses::json, '$[0]'), '$.country') = 'US' AND
       json_extract_string(json_extract(addresses::json, '$[0]'), '$.region') = 'FL'
) TO 'c:/temp/places_fl.csv' WITH (FORMAT CSV);
</code>
<h3>
  Parse the JSON and WKB fields
</h3>
<p>
  The file contains many fields that are in JSON format. We can parse the fields in our output table for easy accessibility using the statement below. Use json_extract and json_extract_string to accomplish this.
</p>
<p>
  The <b>geometry</b> field is in WKB format. Use the ST_GeomFromWkb() function to parse the <b>geometry</b> field. This will output latitude and longitude coordinates for mapping and analytics.
</p>
<code>COPY (
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
<p>
  In this example, we parse the address field into city, zipcode, and address (see freeform). Just in the state of Florida, it contains location information for over 800,000 business and other locations!
</p>

# Finally
<p>There are more datasets available that are continually being updated, so always get the latest version of the data.</p>
<p>Hope this helps!</p>
