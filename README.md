# MongoDB Establishments Analysis

This repository provides a collection of Python scripts to analyze establishments stored in a MongoDB database. The scripts demonstrate how to query and manipulate data using MongoDB's aggregation framework and PyMongo, and how to display the results using pandas.

## Setup

### Prerequisites

- Python 3.x
- MongoDB
- PyMongo
- Pandas

### Installation

1. **Clone the repository**:
    ```bash
    git clone <https://github.com/Birdy173/nosql-challenge.git>
    cd <Starter_Code>
    ```

2. **Install the required Python packages**:
    ```bash
    pip install pymongo pandas
    ```

3. **Set up MongoDB**:
   Ensure your MongoDB server is running and accessible. Update the connection string in the scripts to match your MongoDB instance.

## Scripts Overview

### 1. Count Establishments with a Hygiene Score of 0

This script counts the number of establishments with a hygiene score of 0 and displays the first matching document.

```python
from pymongo import MongoClient
from pprint import pprint

client = MongoClient('your_connection_string')
db = client['restaurant_db']
establishments = db['establishments']

query = {'scores.Hygiene': 0}
count = establishments.count_documents(query)
print(f'Number of establishments with a hygiene score of 0: {count}')

first_result = establishments.find_one(query)
print('First document with a hygiene score of 0:')
pprint(first_result)
```
### 2. Find Establishments in London with a Rating Value >= 4

This script finds establishments in London with a rating value of 4 or greater.
```python
from pymongo import MongoClient
from pprint import pprint

client = MongoClient('your_connection_string')
db = client['restaurant_db']
establishments = db['establishments']

query = {
    'AddressLine3': 'London',
    'RatingValue': {'$gte': 4}
}
count = establishments.count_documents(query)
print(f'Number of establishments in London with a Rating Value >= 4: {count}')

first_result = establishments.find_one(query)
print('First document:')
pprint(first_result)
```
### 3. Find Top Establishments Near "Penang Flavours"

This script finds the top establishments with a rating of 5, within 0.01 degrees of latitude and longitude around "Penang Flavours", sorted by hygiene score.

```python
from pymongo import MongoClient
from pprint import pprint

client = MongoClient('your_connection_string')
db = client['restaurant_db']
establishments = db['establishments']

degree_search = 0.01
latitude = location['geocode']['latitude']
longitude = location['geocode']['longitude']

query = {
    'RatingValue': 5,
    'geocode.latitude': {'$gte': latitude - degree_search, '$lte': latitude + degree_search},
    'geocode.longitude': {'$gte': longitude - degree_search, '$lte': longitude + degree_search}
}
sort = [('scores.Hygiene', 1)]
limit = 5

results = list(establishments.find(query).sort(sort).limit(limit))
print('Top establishments with a Rating Value of 5, within 0.01 degree of Penang Flavours latitude and longitude, sorted by hygiene score: ')
for establishment in results:
    pprint(establishment)
```
### 4. Aggregation Pipeline to Match, Group, and Sort Establishments

This script uses an aggregation pipeline to match establishments with a hygiene score of 0, group them by local authority, and sort the results.

```python
from pymongo import MongoClient
from pprint import pprint

client = MongoClient('your_connection_string')
db = client['restaurant_db']
establishments = db['establishments']

match_query = {'$match': {'scores.Hygiene': 0}}

group_query = {'$group': {'_id': '$LocalAuthorityName',
                          'count': {'$sum': 1}
                         }
              }

sort_values = {'$sort': {'count': -1}}

pipeline = [match_query, group_query, sort_values]
results = list(establishments.aggregate(pipeline))
print(f'Number of Local Authorities with establishments having a hygiene score of 0: {len(results)}')

for establishment in results[:10]:
    pprint(establishment)


