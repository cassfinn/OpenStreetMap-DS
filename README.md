# OpenStreetMap-DS
Udacity Data Analysis Nanodegree, Project 2:  OpenStreetMap Data Wrangling

Assignment:
Choose any area of the world in https://www.openstreetmap.org and use data munging techniques, such as assessing the quality of the data for validity, accuracy, completeness, consistency and uniformity, to clean the OpenStreetMap data for a part of the world that you care about. 



OpenStreetMap Project - Claudia Cassidy

Files included:
- openstreetmap_orlando_ccassidy.md:  A document containing answers to the rubric questions and the data wrangling process.

- cleanImport2.py:  the Python script I used to successfully clean up data and convert the OSM file into the csv files.

- sampledata.osm:  a small subset of the "maporlando.osm" file which was exported from OpenStreetMap.  

- getSampleSubset.py:  a script for generating the sampledata.osm.

- schema.py:  the schema used to create the sqlite3 tables.

The csv files which were created after running "python cleanImport2.py"
- ways_nodes.csv
- ways.csv
- ways_tags.csv
- nodes.csv
- nodes_tags.csv

--------------------

Link to Map Position:

https://www.openstreetmap.org/export#map=10/28.5405/-81.3799

The data I exported from OpenStreetMap is from Central Florida.  
I chose this area because I live here and could maybe use my knowledge of the
area to judge how accurate the data reported actually is.  For example, if the
cities listed are correct, if names are misspelled, or areas are missing.



--------------------

Websites Used:

- In addition to reviewing the Udacity lesson on SQL and other lessons in this section in Udacity, 
I mostly used Stack Overflow when I ran into issues getting my python scripts to work and for 
processing the SQLite data. 

