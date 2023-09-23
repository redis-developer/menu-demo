# Redis JSON + Search Restaurant Menu Demo  

## Contents
1.  [Summary](#summary)
2.  [Features](#features)
3.  [Prerequisites](#prerequisites)
4.  [Installation](#installation)
8.  [Scenario 1](#scenario1)
9.  [Scenario 2](#scenario2)
10.  [Scenario 3](#scenario3)

## Summary <a name="summary"></a>
This is a Jupyter notebook that loads a Kaggle dataset into Redis in two manners:  string and JSON. Comparisons of string and JSON operations are made for three scenarios. 

## Features <a name="features"></a>
- Loads Kaggle restaurant data
- Executes Redis operations on that data to solve various hypothetical business problems 

## Prerequisites <a name="prerequisites"></a>
- Redis Enterprise database with Search and JSON enabled
- Python
- Jupyter

## Installation <a name="installation"></a>
1. Clone this repo.
2. Edit the .env_template file with your credentials.
3. Copy or rename that file to .env
4. Follow notebook steps

## Scenario 1 <a name="scenario1"></a>
### Business Problem
Given a known restaurant ID and a menu item name, find its price.
### String
```python
restaurant_str = client.get(f'restaurant_str:{restaurant_id}')
restaurant_json = json.loads(restaurant_str)
menu = restaurant_json['menu']
for item in menu:
    if item['name'] == menu_item:
        price = item['price'] 
        break
```
### JSON w/Search
```python
query = Query(f'@id:[{restaurant_id}, {restaurant_id}]')\
    .return_field(f'$.menu[?(@.name=="{menu_item}")].price', as_field='price')
result = client.ft('restaurant_idx').search(query)
```

## Scenario 2 <a name="scenario2"></a>
### Business Problem
Find the number of Papa Johns restaurants within a 100 mi radius of Madison WI
### String - Exercise left for the Reader
- Loop thru the Redis key space
- Deserialize each string to JSON
- Perform a string comparison on the restaurant name element, covering letter case variations 
- For restaurant name string matches, perform a Haversine calculation
- Update local counter for Haversine matches

### JSON w/Search
```python
query = Query(f'@name:"Papa Johns" @coords:[{madison} 100 mi]').paging(0, 0)
count = client.ft('restaurant_idx').search(query).total 
```

## Scenario 3 <a name="scenario3"></a>
### Business Problem
Find the Top 3 menu items by count in the State of Texas
### String - Exercise left for the Reader
- Loop thru the Redis key space
- Deserialize each string to JSON
- Perform a string comparison on the restaurant address element
- For restaurant address string matches, initiate a inner loop on the menu elements - cover letter case + stem variations
- Maintain counters for every possible menu item.  Increment applicable counter per menu element
- Sort counters, display Top 3 items

### JSON w/Search
```python
request = AggregateRequest('@address:TX')\
    .group_by('@menu_item_name', reducers.count().alias('item_count'))\
    .sort_by(Desc('@item_count'))\
    .limit(0,3)
result = client.ft('restaurant_idx').aggregate(request)
```  