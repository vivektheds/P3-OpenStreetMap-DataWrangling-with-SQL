
# Project 3: OpenStreetMap Project - Data Wrangling with SQL

## By Vivek Pandey

### Introduction

OpenStreetMap (OSM) is an open-source project attempting to created a free map of the entire world from volunteer-entered data. Moreocer,openStreetMap License allows free (or almost free) access to our map images and all of our underlying map data. The project aims to promote new and interesting uses of this data.

This dataset is user generated hence prone to have some dirty data.

In this project, I have used Los Angeles map from OSM and use data wrangling techniques to audit the map data for validity, accuracy, completeness, consistency and uniformity, which helps us to clean the data.In addition to this I converted xml data to CSV format and import these CSV files into sqlite database.

I used SQL queries to analyze Los Angeles map data to findout some interesting thing about this city where I live.


### Map Area: I have choosen Los Angeles.

I have selected Los Angeles Map data because I live here and very much familier with most of the places in this city hence I am more curious to explore this city map data and see what kind of data is being contributed in openstreetmap.

Los Angeles, CA, USA

OpenStreetMap URL: https://www.openstreetmap.org/relation/207359


### 1- Data Audit 

While looking at Los Angeles city XML data I found that it has different kind of elements hence using cElementTree I parse the xml data and cound the no of unique tags.


```python
import os
import collections
import pprint
import xml.etree.cElementTree as ET
import re
import codecs
import csv
import cerberus
import copy
import schema
import pprint
```


```python
"""
Parse the OSM file and count the numbers of unique tag
"""
import os
import collections
import pprint
import xml.etree.cElementTree as ET
import re
import codecs
import csv
import cerberus
import copy
import schema
import pprint


OSMFILE = "los-angeles.osm"

def count_tags(filename):
    tags = {}
    for event, elem in ET.iterparse(filename):
        if elem.tag in tags: 
            tags[elem.tag] += 1
        else:
            tags[elem.tag] = 1
    return tags
    
pprint.pprint(count_tags(OSMFILE))
```

    {'bounds': 1,
     'member': 49554,
     'nd': 5014756,
     'node': 4108185,
     'osm': 1,
     'relation': 5382,
     'tag': 3427715,
     'way': 430328}
    

{'bounds': 1,
 'member': 49554,
 'nd': 5014756,
 'node': 4108185,
 'osm': 1,
 'relation': 5382,
 'tag': 3427715,
 'way': 430328}

The below is the summary of "k" values in the "tag" nodes and in order to find out the count of each of three tag categories I have used below three regular expressions.

1. "lower" : 90228, for tags that contain only lowercase letters and are valid.
2. "lower_colon" : 1104, for otherwise valid tags with a colon in their names.
3. problemchars" : 16, for tags with problematic characters.
4. other" : 0, for other tags that do not fall into the other three categories.


```python
"""
Count multiple patterns in the tags
"""

import xml.etree.cElementTree as ET
import pprint
import re

from collections import defaultdict

lower = re.compile(r'^([a-z]|_)*$')
lower_colon = re.compile(r'^([a-z]|_)*:([a-z]|_)*$')
problemchars = re.compile(r'[=\+/&<>;\'"\?%#$@\,\. \t\r\n]')

OSMFILE = "los-angeles.osm"

def key_type(element, keys):
    if element.tag == "tag":
        for tag in element.iter('tag'):
            k = tag.get('k')
            if lower.search(element.attrib['k']):
                keys['lower'] = keys['lower'] + 1
            elif lower_colon.search(element.attrib['k']):
                keys['lower_colon'] = keys['lower_colon'] + 1
            elif problemchars.search(element.attrib['k']):
                keys['problemchars'] = keys['problemchars'] + 1
            else:
                keys['other'] = keys['other'] + 1
    
    return keys

def process_map(filename):
    keys = {"lower": 0, "lower_colon": 0, "problemchars": 0, "other": 0}
    for _, element in ET.iterparse(filename):
        keys = key_type(element, keys)

    return keys

pprint.pprint(process_map(OSMFILE))
```

    {'lower': 1372875, 'lower_colon': 1990585, 'other': 64228, 'problemchars': 27}
    

## 2. Problems identified in Map

In order to identify problems in Los Angeles(LA) map data and I have audited on sample data of LA and I have identified below problems.

- Inconsistencies in abbreviation of street names
- Inconsistencies in postal code

** Inconsistencies in abbreviation of street names **

In street addresses I observed that there are inconsistencies in street types like Avenue has represented differently (Ave, Ave. ,Av) which can lead to inaccurate results.



```python
# For identifying problem in street name using LA sample data
import xml.etree.cElementTree as ET
from collections import defaultdict
import re

osm_file = "LA_sample.osm"

street_type_re = re.compile(r'\S+\.?$', re.IGNORECASE)
street_types = defaultdict(int)

def audit_street_type(street_types, street_name):
    m = street_type_re.search(street_name)
    if m:
        street_type = m.group()

        street_types[street_type] += 1

def print_sorted_dict(d):
    keys = d.keys()
    keys = sorted(keys, key=lambda s: s.lower())
    for k in keys:
        v = d[k]
        print "%s: %d" % (k, v) 

def is_street_name(elem):
    return (elem.tag == "tag") and (elem.attrib['k'] == "addr:street")

def audit():
    for event, elem in ET.iterparse(osm_file):
        if is_street_name(elem):
            audit_street_type(street_types, elem.attrib['v'])    
    print_sorted_dict(street_types)    

if __name__ == '__main__':
    audit()
```

    3rd: 1
    79: 2
    Aguacate: 1
    Alisos: 1
    Altamira: 2
    Amigos: 2
    Amo: 1
    Av: 18
    Ave: 51
    Ave.: 1
    Avenue: 43
    Bl: 3
    Blvd: 15
    Blvd.: 3
    Boulevard: 26
    Broadway: 6
    Camino: 1
    Caralene: 1
    Castle: 2
    Castlebay: 2
    Cielo: 3
    Cir: 3
    Coltrane: 1
    Commerce: 1
    Court: 8
    Cr: 5
    Ct: 39
    Dr: 119
    Drive: 7
    Dulcea: 1
    Flores: 1
    Floresta: 1
    Gavilan: 2
    Highway: 5
    Hill: 3
    Hwy: 1
    Ic: 2
    Inca: 1
    Ladera: 1
    Limar: 1
    Ln: 83
    Lomas: 1
    Maranatha: 1
    Monserate: 1
    Nord: 1
    Oaks: 1
    Olivos: 1
    Oro: 2
    Paloma: 1
    Park: 1
    Parkway: 1
    Pky: 1
    Pl: 19
    Place: 3
    Portofino: 1
    Rainbow: 1
    Rancheros: 1
    Ranchitos: 1
    Rd: 169
    Ridge: 1
    Road: 16
    Robles: 1
    Rte: 2
    Serena: 1
    Serra: 1
    Sol: 1
    Sonia: 1
    Spur: 2
    Sr-76: 5
    Sr-79: 5
    St: 61
    St.: 6
    Street: 43
    Suerte: 1
    Tiempos: 1
    Tl: 3
    Tr: 6
    Trl: 1
    Tt: 1
    Vecinos: 1
    Venado: 1
    Verde: 3
    Vw: 1
    Wa: 3
    Way: 2
    West: 1
    Wy: 13
    

I have updated all the above identified street types (which has inconsistencies) with the given mapping list, so that query result should give correct results.


** Inconsistencies in postal code **

In postal code I observed that postal codes are mostly of 5 digits and as per audit we found that there are some postal codes where four digit extra portion or state abbrebiation like 'CA' is coming ,which needs to be taken care. 


```python
# Inconsistent Postal Codes

def audit_zipcode(invalid_zipcodes, zipcode):
    twoDigits = zipcode[0:2]
    
    if twoDigits != 90 or twoDigits != 91  or not twoDigits.isdigit():
        invalid_zipcodes[twoDigits].add(zipcode)
        
def is_zipcode(elem):
    return (elem.attrib['k'] == "addr:postcode")

def audit_zip(osmfile):
    osm_file = open(osmfile, "r")
    invalid_zipcodes = collections.defaultdict(set)
    for event, elem in ET.iterparse(osm_file, events=("start",)):

        if elem.tag == "node" or elem.tag == "way":
            for tag in elem.iter("tag"):
                if is_zipcode(tag):
                    audit_zipcode(invalid_zipcodes,tag.attrib['v'])

    return invalid_zipcodes

osm_file = "LA_sample.osm"
LA_zipcode = audit_zip(osm_file)
pprint.pprint(dict(LA_zipcode))
```

    {'90': set(['90002-3024',
                '90006-4005',
                '90012',
                '90013',
                '90016',
                '90017',
                '90022',
                '90023',
                '90024',
                '90028',
                '90040',
                '90045',
                '90049',
                '90065',
                '90069',
                '90077',
                '90095',
                '90240',
                '90265',
                '90277',
                '90503',
                '90620',
                '90631',
                '90680',
                '90710',
                '90712',
                '90731',
                '90731-7415',
                '90802',
                '90806',
                '90807',
                '90813',
                '90815']),
     '91': set(['91007',
                '91030',
                '91102',
                '91103',
                '91104',
                '91105',
                '91106',
                '91107',
                '91214',
                '91303-2211',
                '91304',
                '91321',
                '91365',
                '91367',
                '91502',
                '91506',
                '91606',
                '91701',
                '91710',
                '91711',
                '91730',
                '91733',
                '91737',
                '91739',
                '91740',
                '91741',
                '91752',
                '91761',
                '91764',
                '91765',
                '91767',
                '91770',
                '91776',
                '91784',
                '91786',
                '91801']),
     '92': set(['92028',
                '92059',
                '92060',
                '92061',
                '92082',
                '92086',
                '92223',
                '92315',
                '92336',
                '92342',
                '92374',
                '92377',
                '92399',
                '92401-1400',
                '92506',
                '92518',
                '92530',
                '92536',
                '92543',
                '92551',
                '92570',
                '92591',
                '92592',
                '92602',
                '92610',
                '92618',
                '92627',
                '92663',
                '92704',
                '92802',
                '92821',
                '92843',
                '92866',
                '92869',
                '92880',
                '92881',
                '92883']),
     '93': set(['93001', '93010', '93065']),
     'CA': set(['CA 90012'])}
    

#### We have handled this inconsistency in Audit.py

## 3. Database schema preparation
After auditing I have converted OSM xml data to CSV format then imported these CSV files into sqlite database.

I have used **'LA OSM database.py'** file to convert xml document data into CSV then created tables for each CSV files and loaded that data into database table which helps to explore data more efficiently.

### Los Angeles Map Data overview

Now I have used database query to find out some interesting information about LA.

### File sizes:

los-angeles.osm .............. 932 MB <br>
LA_sample.osm ............... 31.5 MB <br>
LosAngeles.db ............... 22.8 MB <br>
nodes.csv .................... 10.9 MB <br>
nodes_tags.csv .............. 699 KB <br>
ways.csv ...................... 831 KB <br>
ways_nodes.csv .............. 3.85 MB <br>
ways_tags.csv ............... 3.41 MB <br>


###### Connect to "LosAngeles.db" to do analysis :
sqlite> .open LosAngeles.db

##### Number of nodes:

sqlite> Select count(*) from nodes;
#####  Output:
136940

##### Number of ways:
sqlite> Select count(*) from ways;
#####  Output:
14344

##### Number of unique users:
sqlite> Select count(distinct uid) from (select uid from nodes union select uid from ways);
#####  Output:
901

##### Top ammenities in LA:
sqlite> select value, count(value) count from nodes_tags where key='amenity' group by value order by count  desc limit 10;
#####  Output:

    place_of_worship 125
    school           117
    fast_food        54
    restaurant       42
    fire_station     19
    fuel             16
    cafe             14
    bank             13
    parking          12
    toilets          11
    
##### Top tourism in LA:
sqlite> select value, count(value) count from nodes_tags where key='tourism' group by value order by count desc;
#####  Output:    

    attraction     9
    viewpoint      7
    picnic_site    6
    hotel          5
    information    4
    camp_site      3
    museum         3
    motel          2
    artwork        2
    flagpole       1
    hostel         1
    
##### Biggest religion in LA:
sqlite> select value, count(value) count from nodes_tags where key='religion' group by value order by count desc;
#####  Output:      
    
    christian      118
    jewish         1
    muslim         1 
    
##### Top Cuisine in LA:
sqlite> select value, count(value) count from nodes_tags where key='cuisine' group by value order by count desc;
#####  Output:      
    burger      25
    mexican     9
    american    8
    pizza       6
    chicken     4
    italian     3
    chinese     2
    coffee_shop 2
    deli        2
    steak_house 2
    

### 4. Conclusion:


From this entire data wrangling process I realized how important is to clean and parse data ,and most importantly to make data in tablular format which makes data exploration easier.
Moreover openstreetmap is open source project and some of the users can make some mistakes while contributing the map data, hence data cleansing gets very important in order to get rid of human errors.



```python

```
