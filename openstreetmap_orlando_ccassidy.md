# OpenStreetMap Data Wrangling Project
## Claudia Cassidy


### Map Area
Central Florida, United States


This map is of Central Florida, where I live. I chose this area because I'm familiar with it and can use my knowledge to evaluate whether the OpenStreetMap data is more or less accurate enough about what's here.


## Problems Encountered in the Map
I first ran the script "cleanImport2.py" to generate the csv files.  When I tried to import them into sqlite3 there were errors in the csv files. Since the original data file took almost an hour to run, I decided to generate a small sample of the .osm file, fix the reported issues, add code to clean those issues and rerun the import until no errors are reported. 

I encountered an issue when importing that data into SQLite:
- Sqlite was treating the character "|" as a separator, as if it was a comma.  I updated the python script to replace the pipe character with a dash.


Once the data was imported into sqlite, I ran some queries to see if the data needed further cleaning. There were issues I observed with postal codes and city names.


### Postal Codes

I saw that postal codes were not formatted consistently.  Some began with FL, others had a dash with a 4 digit extension. Most were 5 digits. In order to get the postal codes consistent, I removed the FL prefix and the -#### extensions.

select node.id, node_tags.id, node_tags.key, node_tags.value
from node, node_tags
where node.id = node_tags.id and node_tags.key = "postcode";


Example of postal codes which needed cleaning:
FL 32803
32707-5509


###SQL to clean the postal codes:

### view the postal codes:
select distinct value
from node_tags
where key = "postcode";

### Remove Leading "FL" from postal codes:
update node_tags
set value = replace(value, 'FL ', '')
where key = "postcode";

### Trim -#### from postal codes:
update node_tags
set value = substr(value, 1, 5)
where key = "postcode";

Final Result:
32809
32811
32703
32750
32751
32808
34786
32817
32828
32792
32803
32806
32818
32822
32810
34761
32805
34787
32789
32804
32825
34762
32708
32765
32819
32835
32807
32839
32746
32771
32801
32701
32768
34734
32710
32798
32773
32714
32826
32730
32712
32816
32779
32707




I was curious about what amenities are in the database

### Get an alphabetical list of amenities:

select distinct b.key, b.value
from way a, way_tags b
where a.id = b.id and b.key = "amenity"
order by b.value;

I noticed that similar amenities were categorized into different values for what appears to be the same thing.  For example:
- restaurant, ice_cream, fast_food, cafe
- college, university, school


Also, "arts_centre" and "community_centre" were spelled with a British style of english.  I would have expected to see "centre" spelled as "center".


Since the list of unique amenities showed that the data had two values for higher education: "college" and "university".
a user of the data should probably run a query as follows to get all of the relevant data: 
select *
from way a, way_tags b
where a.id = b.id and b.key = "amenity" and (value="university" or value="college")
order by b.value;



# Cities in the Orlando Area

Since I live in Central Florida, I am familiar with the cities here.  
I ran a query to list the names of cities in the database.

```sql
sqlite> SELECT tags.value 
FROM (SELECT * FROM node_tags UNION ALL 
      SELECT * FROM way_tags) tags
WHERE tags.key LIKE '%city'
GROUP BY tags.value
ORDER BY tags.value;
```

Here are the results:

"Altamonte Springs"
Apopka
"Apopka, FL"
Casselberry
Florida
Gotha
LONGWOOD
"Lake Mary"
Longwood
Maitland
Ocoee
Orlando
Oviedo
"Pine Castle"
Sanford
"Wekiva Springs"
Windermere
"Winter Garden"
"Winter Park"
"Winter Springs"
apopka
ocoee
sanford

While the results are accurate, there were problems with the data in which double quotes were part of the name, names that were entered beginning with a capital letter were showing up separately from the same name with lower case.  In one case, "Apopka" appeared three different ways:  Apopka, "Apopka, FL", and apopka.


I wrote the following SQL to clean up this data:


### Remove the ", FL"
update way_tags
set value = replace(value, ', FL', '')
where key = 'city';

### Remove the double quote character
update way_tags
set value = replace(value, '"', '')
where key = 'city';


### Make the data proper case
update way_tags
set value = UPPER(SUBSTR(value, 1, 1)) || SUBSTR(value, 2)
where key = 'city';


### Result
select distinct value from way_tags
where key = 'city'
order by value;

Altamonte Springs
Apopka
Florida
Lake Mary
Longwood
Maitland
Ocoee
Orlando
Oviedo
Sanford
Wekiva Springs
Windermere
Winter Garden
Winter Park
Winter Springs





# Data Overview and Additional Ideas
This section contains basic statistics about the dataset, the SQL queries used to gather them, and some additional ideas about the data in context.

### File sizes
```
maporlando.osm ........ 169.4 MB
orlando.db ............ 115.9 MB
nodes.csv ............. 57.2 MB
nodes_tags.csv ........ 2.9 MB
ways.csv .............. 5.5 MB
ways_tags.csv ......... 16 MB
ways_nodes.cv ......... 20 MB  
```  

### Number of nodes
```
sqlite> SELECT COUNT(*) FROM nodes;
```
709504

### Number of ways
```
sqlite> SELECT COUNT(*) FROM ways;
```
98688

### Number of unique users
```sql
sqlite> SELECT COUNT(DISTINCT(e.uid))          
FROM (SELECT uid FROM node UNION ALL SELECT uid FROM way) e;
```
784

### Top 10 contributing users
```sql
sqlite> SELECT e.user, COUNT(*) as num
FROM (SELECT user FROM node UNION ALL SELECT user FROM way) e
GROUP BY e.user
ORDER BY num DESC
LIMIT 10;
```

```sql
NE2, 397340
crystalwalrein, 35181
"Adam Martin", 34702
grouper, 21753
epcotfan, 21656
dale_p, 21367
Cdale1986, 18306
cwhite13, 12881
3yoda, 11973
woodpeck_fixbot, 11199
```
 
### Number of users appearing only once (having 1 post)
```sql
sqlite> SELECT COUNT(*) 
FROM
    (SELECT e.user, COUNT(*) as num
     FROM (SELECT user FROM node UNION ALL SELECT user FROM way) e
     GROUP BY e.user
     HAVING num=1)  u;
```
211



# Additional Ideas

### Top 10 Appearing Amenities

```sql
sqlite> SELECT value, COUNT(*) as num
FROM node_tags
WHERE key='amenity'
GROUP BY value
ORDER BY num DESC
LIMIT 10;
```

```sql
place_of_worship, 441
restaurant, 267
fast_food, 241
school, 149
fuel, 101
bicycle_parking, 79
fountain, 76
fire_station, 68
bank, 59
cafe, 49

Number one by far listed is place_of_worship.

```

### Biggest religions

```sql
sqlite> SELECT node_tags.value, COUNT(*) as num
FROM node_tags 
    JOIN (SELECT DISTINCT(id) FROM node_tags WHERE value='place_of_worship') i
    ON node_tags.id=i.id
WHERE node_tags.key='religion'
GROUP BY node_tags.value
ORDER BY num DESC
LIMIT 3;
```
There were only 2 results:

christian, 504
scientologist, 1


### Most popular cuisines, Top 10

```sql
sqlite> SELECT node_tags.value, COUNT(*) as num
FROM node_tags 
    JOIN (SELECT DISTINCT(id) FROM node_tags WHERE value='restaurant') i
    ON node_tags.id=i.id
WHERE node_tags.key='cuisine'
GROUP BY node_tags.value
ORDER BY num DESC
LIMIT 10;
```

```sql
american, 16
mexican, 14
pizza, 10
burger, 6
italian, 6
sandwich, 5
steak_house, 4
asian, 3
chinese, 3
barbecue, 2
```

# Conclusion
 After this review of the data it’s obvious that the Orlando area is inconsistent.  For example, amenities for places to eat are categorized in several different ways:  fast_food, restaurant, cafe, and more.  There are inconsistencies in how data such as city names are entered.  For example, some cities are all lowercase, others are in double quotes.  It is adequate for the purposes of this exercise. The following section lists suggested solutions for improving the data, potential issues and benefits. 
 
### Benefits/Anticipated Issues
### Issues with the Current Data:
The amenities listed do not include many other categories which might make the data more useful to people:
- 1 - For example, there is only one sports center listed in all of the Central Florida area, but I know from living here that there are many more. If I wanted to find a soccer field or a tennis court the OpenStreetMap data would not be helpful. Someone visiting would not know that there are more facilities if their only source of information was OSM.
- 2 - There seem to be many errors reported by people. For example, mislabeled facilities, missing roads which are closed, incorrect information about speed limits.

### Recommendations and Possible Solutions for Improving the Data
Since OpenStreetMap is updated voluntarily, there is not a pressing incentive to maintain and improve the accuracy of the data.

One solution might be to find a way to incentivize business owners and other stakeholders to update their own data.  For example, the Florida Youth Soccer Association could update all of the soccer fields and label them on the map.  This would be useful for coordinating recreational games. Another example would be for the United States Tennis Association to update all of the tennis fields available. 
Anticipated issues: Someone would have to own the incentivization project and communicate it to all. This is a volunteer, unpaid task and it would be difficult to find people willing to promote this initiative. People are on their honor to enter data correctly.

Another solution might be to have local governments take responsibility for updating the data on the OpenStreetMap project. Local governments already maintain property information for use in zoning decisions and planning. While there are other sources of mapping information which may be more accurate, such as Google Maps and Google Places, those sources of data are privately owned and it can be too expensive for many localities to purchase an API key.

Benefits of Government Responsibility:
 <br />- The OSM data could be useful for both public and private interests if the quality of the data was improved. The government could add layers and amenities which Google and other sources may not have access to, but which could lead to growth in profits and quality of life for businesses and residents. 
 
 ### Ideas for OpenStreetMap Applications
 Assuming the data was cleaned up and supplemented with layers and more amenities:
      <br />- Businesses considering relocating to Central Florida would have a reliable source of mapping information where they could easily see which areas are zoned for their purposes. For example, if a major corporation like Amazon was considering opening a second headquarters in Central Florida, they would get a much better idea of transportation, facilities and zoning areas if they happened to use OSM data to research.  As it is now, the OSM data is incomplete.
      <br />
      <br />- Layers could be added to the map which could help city planners and residents design "Smart Cities" in which newer technology is applied to improve the quality of life of residents.  For example, high tech sanitation facilities, traffic lights which can adapt to the flow of traffic, and street lights which notify the government automatically when their bulbs need to be replaced. More information: https://www2.deloitte.com/nl/nl/pages/real-estate/articles/smart-cities-and-the-vital-role-of-the-government.html  
      <br />- Apps could be written which combine the publicly available GPS data with data from Data.gov and other sources to create interesting and meaningful maps. For example, a map showing location of schools combined with census data may show a correlation between zip code and academic success (Florida grades schools on A,B,C,D levels).  This would provide more meaningful information to education planners and decision makers in the school districts, counties and state about funding allocation.
      <br />
      <br />- Political parties would also find useful information if they overlaid voting districts onto maps and planned routes for campaigning.  
      <br />- Delivery services could use the maps to generate smarter, faster routes to their customers.





