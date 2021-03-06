ETL
================
2021/9/25

This is a mini ETL project from IBM Data Engineering course.

## Packages

``` python
import glob
import pandas as pd
import xml.etree.ElementTree as ET
from datetime import datetime
```

## Download & Unzip Raw Files

Raw files used in this project is in:
<https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0221EN-SkillsNetwork/labs/module%206/Lab%20-%20Extract%20Transform%20Load/data/datasource.zip>

``` r
download.file("https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0221EN-SkillsNetwork/labs/module%206/Lab%20-%20Extract%20Transform%20Load/data/datasource.zip", "./datasource.zip")
unzip("datasource.zip", exdir = "./dealship_data")
```

## Set Paths

``` python
tmpfile    = "dealership_temp.tmp"               # file used to store all extracted data
logfile    = "dealership_logfile.txt"            # all event logs will be stored in this file
targetfile = "dealership_transformed_data.csv"   # file where transformed data is stored
```

## Extract Functions for Different Types of Data

``` python
def extract_csv(file):
    df = pd.read_csv(file)
    return df


def extract_json(file):
    df = pd.read_json(file, lines = True)
    return df


def extract_xml(file):
    df = pd.DataFrame(columns=['car_model','year_of_manufacture','price', 'fuel'])
    tree = ET.parse(file)
    root = tree.getroot()
    for node in root:
        car_model = node.find('car_model').text
        year_of_manufacture = int(node.find('year_of_manufacture').text)
        price = float(node.find('price').text)
        fuel = node.find('fuel').text
        df = df.append({'car_model':car_model, 'year_of_manufacture':year_of_manufacture, 'price': price, 'fuel': fuel}, ignore_index = True)
    return df
```

## Extract

``` python
def extract():
    extracted_data = pd.DataFrame(columns=['car_model','year_of_manufacture','price', 'fuel']) 
    
    #process all csv files
    for csvfile in glob.glob("dealship_data/*.csv"):
        extracted_data = extracted_data.append(extract_csv(csvfile), ignore_index=True)
        
    #process all json files
    for jsonfile in glob.glob("dealship_data/*.json"):
        extracted_data = extracted_data.append(extract_json(jsonfile), ignore_index=True)
    
    #process all xml files
    for xmlfile in glob.glob("dealship_data/*.xml"):
        extracted_data = extracted_data.append(extract_xml(xmlfile), ignore_index=True)
        
    return extracted_data
```

## Transform

Round the `price` columns to 2 decimal places

``` python
def transform(data):
    data['price'] = round(data.price, 2)
    return data
```

## Loading

``` python
def load(target, data):
    data.to_csv(target)
```

## Logging

``` python
def log(message):
    timestamp_format = "%Y-%h-%d-%H:%M:%S"
    now = datetime.now()
    timestamp = now.strftime(timestamp_format)
    with open("dealership_logfile.txt", "a") as f:
        f.write(timestamp + "," + message + '\n')
```

## Running ETL Process

``` python
log("ETL Start")

log("Extract Start")
extracted_data = extract()
```

    ## C:\Users\Jon\ANACON~1\lib\site-packages\pandas\core\frame.py:6201: FutureWarning: Sorting because non-concatenation axis is not aligned. A future version
    ## of pandas will change to not sort by default.
    ## 
    ## To accept the future behavior, pass 'sort=True'.
    ## 
    ## To retain the current behavior and silence the warning, pass sort=False
    ## 
    ##   sort=sort)
    ## C:\Users\Jon\ANACON~1\lib\site-packages\pandas\core\frame.py:6201: FutureWarning: Sorting because non-concatenation axis is not aligned. A future version
    ## of pandas will change to not sort by default.
    ## 
    ## To accept the future behavior, pass 'sort=True'.
    ## 
    ## To retain the current behavior and silence the warning, pass sort=False
    ## 
    ##   sort=sort)

``` python
log("Extract End")


log("Transform Start")
transformed_data = transform(extracted_data)
log("Transform End")


log("Load Start")
load(targetfile, transformed_data)
log("Load End")


log("ETL End")
```
