# Overture-Maps
Download the Overture Maps geospatial data repository (See https://overturemaps.org/)

<h1>
  Introduction
</h1>
<p>
  In this document, we will explore how to access and download datasets provided by Overture Maps (see https://overturemaps.org/). The Overture Maps dataset contains millions of geospatial features of businesses and other establishments, with detailed information about the places and spatial coordinates of the place. Specifically, we will be looking at the Places dataset provided by Overture, however there are other datasets available (buildings, etc). Happy data hunting!
</p>

# DuckDB
We will be using DuckDB to access the data from Overturemaps, which is provided in Parquet files. DuckDB is a free SQL editor that handles parquet files. Get the latest version here: https://duckdb.org/. Download the CLI (command line tool). Within the CLI, we can easily run our SQL statements by copying an pasting from this document into the CLI tool. 

# Querying the data


