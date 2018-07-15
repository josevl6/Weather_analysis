

```python
# dependecies
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import requests
import json
import random
import time
from citipy import citipy
from config import api_key
from datetime import datetime
from os import path, makedirs, remove, rmdir
import datetime
now = datetime.datetime.now()
now
```




    datetime.datetime(2018, 7, 11, 21, 26, 45, 388802)



## Observable Trends

##### Based on the data recollected from 674 random cities in the world (source: openweathermap): 
##### 1) Latitude vs Temperature: we can deduce that at this time of the year, the temperature tends to get colder the farther the distance from the equator, or the higher the latitude (either positive or negative). In addition, the cities with latitudes bewteen +20 and + 40 reported a higher temperature in July of this year. 

##### 2) Latitude vs Humidity: The majority of cities during this time of the year, regardless of the latitude, have a humidity % between 60 and 100.

##### 3) Latitude vs Wind speed (mph): The majority of cities, regardless of the latitude, have a wind speed in miles/hour of 10 or lower.  

#####  4) It also seems that when the temperature is high, the wind speed tends to be low.


```python
# url = "http://api.openweathermap.org/data/2.5/weather?"

#Define Lat and long
lat = np.random.uniform(low=-90.0,high=90.0,size=2000)

#why is it lng and not long? does it matter?
lng = np.random.uniform(low=-180.0,high=180.0,size=2000) 

coordinates = []
for x in range(0,len(lat)):
    coordinates.append((lat[x],lng[x]))
# print(coordinates)
```


```python
# Generate Cities List
# coordinate_pair in coordinate
cities = []
for coordinate_pair in coordinates:
    lat, lon = coordinate_pair
    cities.append(citipy.nearest_city(lat, lon))
    
# create df
city_df = pd.DataFrame(cities)
city_df["City"] = ""
city_df["Country"] = ""  

for index, row in city_df.iterrows():
    row["City"] = city_df.iloc[index,0].city_name
    row["Country"] = city_df.iloc[index,0].country_code
    
city_df.drop_duplicates(["City","Country"], inplace=True)
city_df.reset_index(inplace = True)

    
del city_df[0]
del city_df["index"]
                
city_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>new norfolk</td>
      <td>au</td>
    </tr>
    <tr>
      <th>1</th>
      <td>punta arenas</td>
      <td>cl</td>
    </tr>
    <tr>
      <th>2</th>
      <td>montclair</td>
      <td>us</td>
    </tr>
    <tr>
      <th>3</th>
      <td>cidreira</td>
      <td>br</td>
    </tr>
    <tr>
      <th>4</th>
      <td>hithadhoo</td>
      <td>mv</td>
    </tr>
  </tbody>
</table>
</div>




```python
city_df["Latitude"] = ""
city_df["Longitude"] = ""
city_df["Cloudiness %"] = ""
city_df["Humidity %"] = ""
city_df["Temperature (F)"] = ""
city_df["Wind Speed (mph)"] = ""

city_df.columns=["City","Country","Latitude", "Longitude","Temperature (F)","Wind Speed (mph)","Humidity %","Cloudiness %"]
city_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Country</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Temperature (F)</th>
      <th>Wind Speed (mph)</th>
      <th>Humidity %</th>
      <th>Cloudiness %</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>new norfolk</td>
      <td>au</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>punta arenas</td>
      <td>cl</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>montclair</td>
      <td>us</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>cidreira</td>
      <td>br</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>hithadhoo</td>
      <td>mv</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>




```python
# Accessing Data
print("Beginning Data Retrieval")
print("--------------------------------")

# Limiting pull request
start_time = time.time()

# for i, row in city in enumerate(cities):

for index, row in city_df.iterrows():
    
    # Building the URL
    url = "http://api.openweathermap.org/data/2.5/weather?q=%s,%s&units=imperial&appid=%s" % (row['City'],
                                                                                        row['Country'], api_key)
    
    print("Retriving city# " + str(index) + ":" + row["City"] + ", " + row["Country"])
    print(url)
    
    weather = requests.get(url).json()
    
    
    try:
        row["Latitude"] = weather["coord"]["lat"]
        row["Longitude"] = weather["coord"]["lon"]
        row["Cloudiness %"] = weather["clouds"]["all"]
        row["Humidity %"] = weather["main"]["humidity"]
        row["Temperature (F)"] = weather["main"]["temp"]
        row["Wind Speed (mph)"] = weather["wind"]["speed"]
        
    except:
        print("Error with city data. Skipping")
        continue
         
    # pausing to limit requests
    if (index + 1) % 60 == 0:
        run_time = start_time - start_time
        time.sleep(60 - run_time)
        
print("------------------------------------")
print("Data Retrieval Complete")
print("------------------------------------")

city_df.head()
```

    Beginning Data Retrieval
    --------------------------------
    Retriving city# 0:new norfolk, au
    http://api.openweathermap.org/data/2.5/weather?q=new norfolk,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 1:punta arenas, cl
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas,cl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 2:montclair, us
    http://api.openweathermap.org/data/2.5/weather?q=montclair,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 3:cidreira, br
    http://api.openweathermap.org/data/2.5/weather?q=cidreira,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 4:hithadhoo, mv
    http://api.openweathermap.org/data/2.5/weather?q=hithadhoo,mv&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 5:susanville, us
    http://api.openweathermap.org/data/2.5/weather?q=susanville,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 6:saint-pierre, pm
    http://api.openweathermap.org/data/2.5/weather?q=saint-pierre,pm&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 7:holme, no
    http://api.openweathermap.org/data/2.5/weather?q=holme,no&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 8:dudinka, ru
    http://api.openweathermap.org/data/2.5/weather?q=dudinka,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 9:taolanaro, mg
    http://api.openweathermap.org/data/2.5/weather?q=taolanaro,mg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 10:te anau, nz
    http://api.openweathermap.org/data/2.5/weather?q=te anau,nz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 11:carnarvon, au
    http://api.openweathermap.org/data/2.5/weather?q=carnarvon,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 12:seoul, kr
    http://api.openweathermap.org/data/2.5/weather?q=seoul,kr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 13:nsoko, sz
    http://api.openweathermap.org/data/2.5/weather?q=nsoko,sz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 14:thinadhoo, mv
    http://api.openweathermap.org/data/2.5/weather?q=thinadhoo,mv&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 15:chapais, ca
    http://api.openweathermap.org/data/2.5/weather?q=chapais,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 16:ilulissat, gl
    http://api.openweathermap.org/data/2.5/weather?q=ilulissat,gl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 17:east london, za
    http://api.openweathermap.org/data/2.5/weather?q=east london,za&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 18:rikitea, pf
    http://api.openweathermap.org/data/2.5/weather?q=rikitea,pf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 19:byron bay, au
    http://api.openweathermap.org/data/2.5/weather?q=byron bay,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 20:nikolskoye, ru
    http://api.openweathermap.org/data/2.5/weather?q=nikolskoye,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 21:talaya, ru
    http://api.openweathermap.org/data/2.5/weather?q=talaya,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 22:ahipara, nz
    http://api.openweathermap.org/data/2.5/weather?q=ahipara,nz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 23:albany, au
    http://api.openweathermap.org/data/2.5/weather?q=albany,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 24:tiksi, ru
    http://api.openweathermap.org/data/2.5/weather?q=tiksi,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 25:daman, in
    http://api.openweathermap.org/data/2.5/weather?q=daman,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 26:ribeira grande, pt
    http://api.openweathermap.org/data/2.5/weather?q=ribeira grande,pt&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 27:mount gambier, au
    http://api.openweathermap.org/data/2.5/weather?q=mount gambier,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 28:dikson, ru
    http://api.openweathermap.org/data/2.5/weather?q=dikson,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 29:cherskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=cherskiy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 30:santiago de cao, pe
    http://api.openweathermap.org/data/2.5/weather?q=santiago de cao,pe&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 31:cap malheureux, mu
    http://api.openweathermap.org/data/2.5/weather?q=cap malheureux,mu&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 32:banyo, cm
    http://api.openweathermap.org/data/2.5/weather?q=banyo,cm&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 33:chokurdakh, ru
    http://api.openweathermap.org/data/2.5/weather?q=chokurdakh,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 34:sitka, us
    http://api.openweathermap.org/data/2.5/weather?q=sitka,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 35:waingapu, id
    http://api.openweathermap.org/data/2.5/weather?q=waingapu,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 36:ponta do sol, pt
    http://api.openweathermap.org/data/2.5/weather?q=ponta do sol,pt&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 37:nizhneyansk, ru
    http://api.openweathermap.org/data/2.5/weather?q=nizhneyansk,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 38:alice springs, au
    http://api.openweathermap.org/data/2.5/weather?q=alice springs,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 39:manaure, co
    http://api.openweathermap.org/data/2.5/weather?q=manaure,co&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 40:tura, ru
    http://api.openweathermap.org/data/2.5/weather?q=tura,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 41:bredasdorp, za
    http://api.openweathermap.org/data/2.5/weather?q=bredasdorp,za&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 42:faanui, pf
    http://api.openweathermap.org/data/2.5/weather?q=faanui,pf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 43:hobart, au
    http://api.openweathermap.org/data/2.5/weather?q=hobart,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 44:kapaa, us
    http://api.openweathermap.org/data/2.5/weather?q=kapaa,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 45:aykhal, ru
    http://api.openweathermap.org/data/2.5/weather?q=aykhal,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 46:fukue, jp
    http://api.openweathermap.org/data/2.5/weather?q=fukue,jp&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 47:port alfred, za
    http://api.openweathermap.org/data/2.5/weather?q=port alfred,za&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 48:kahului, us
    http://api.openweathermap.org/data/2.5/weather?q=kahului,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 49:la ronge, ca
    http://api.openweathermap.org/data/2.5/weather?q=la ronge,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 50:nemuro, jp
    http://api.openweathermap.org/data/2.5/weather?q=nemuro,jp&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 51:khatanga, ru
    http://api.openweathermap.org/data/2.5/weather?q=khatanga,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 52:butaritari, ki
    http://api.openweathermap.org/data/2.5/weather?q=butaritari,ki&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 53:mpanda, tz
    http://api.openweathermap.org/data/2.5/weather?q=mpanda,tz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 54:bambous virieux, mu
    http://api.openweathermap.org/data/2.5/weather?q=bambous virieux,mu&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 55:cumnock, gb
    http://api.openweathermap.org/data/2.5/weather?q=cumnock,gb&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 56:airai, pw
    http://api.openweathermap.org/data/2.5/weather?q=airai,pw&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 57:georgetown, sh
    http://api.openweathermap.org/data/2.5/weather?q=georgetown,sh&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 58:kayerkan, ru
    http://api.openweathermap.org/data/2.5/weather?q=kayerkan,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 59:soe, id
    http://api.openweathermap.org/data/2.5/weather?q=soe,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 60:qasigiannguit, gl
    http://api.openweathermap.org/data/2.5/weather?q=qasigiannguit,gl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 61:bani, do
    http://api.openweathermap.org/data/2.5/weather?q=bani,do&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 62:hilo, us
    http://api.openweathermap.org/data/2.5/weather?q=hilo,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 63:vaini, to
    http://api.openweathermap.org/data/2.5/weather?q=vaini,to&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 64:itarema, br
    http://api.openweathermap.org/data/2.5/weather?q=itarema,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 65:yellowknife, ca
    http://api.openweathermap.org/data/2.5/weather?q=yellowknife,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 66:souillac, mu
    http://api.openweathermap.org/data/2.5/weather?q=souillac,mu&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 67:labuan, my
    http://api.openweathermap.org/data/2.5/weather?q=labuan,my&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 68:bengkulu, id
    http://api.openweathermap.org/data/2.5/weather?q=bengkulu,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 69:hermanus, za
    http://api.openweathermap.org/data/2.5/weather?q=hermanus,za&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 70:labuhan, id
    http://api.openweathermap.org/data/2.5/weather?q=labuhan,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 71:manaus, br
    http://api.openweathermap.org/data/2.5/weather?q=manaus,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 72:san patricio, mx
    http://api.openweathermap.org/data/2.5/weather?q=san patricio,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 73:jodhpur, in
    http://api.openweathermap.org/data/2.5/weather?q=jodhpur,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 74:lebu, cl
    http://api.openweathermap.org/data/2.5/weather?q=lebu,cl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 75:pacific grove, us
    http://api.openweathermap.org/data/2.5/weather?q=pacific grove,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 76:namibe, ao
    http://api.openweathermap.org/data/2.5/weather?q=namibe,ao&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 77:wanaka, nz
    http://api.openweathermap.org/data/2.5/weather?q=wanaka,nz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 78:sembakung, id
    http://api.openweathermap.org/data/2.5/weather?q=sembakung,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 79:veraval, in
    http://api.openweathermap.org/data/2.5/weather?q=veraval,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 80:prince rupert, ca
    http://api.openweathermap.org/data/2.5/weather?q=prince rupert,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 81:saskylakh, ru
    http://api.openweathermap.org/data/2.5/weather?q=saskylakh,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 82:atuona, pf
    http://api.openweathermap.org/data/2.5/weather?q=atuona,pf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 83:busselton, au
    http://api.openweathermap.org/data/2.5/weather?q=busselton,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 84:castro, cl
    http://api.openweathermap.org/data/2.5/weather?q=castro,cl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 85:puerto colombia, co
    http://api.openweathermap.org/data/2.5/weather?q=puerto colombia,co&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 86:artyom, az
    http://api.openweathermap.org/data/2.5/weather?q=artyom,az&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 87:palmer, us
    http://api.openweathermap.org/data/2.5/weather?q=palmer,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 88:mataura, pf
    http://api.openweathermap.org/data/2.5/weather?q=mataura,pf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 89:umzimvubu, za
    http://api.openweathermap.org/data/2.5/weather?q=umzimvubu,za&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 90:torbay, ca
    http://api.openweathermap.org/data/2.5/weather?q=torbay,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 91:yerbogachen, ru
    http://api.openweathermap.org/data/2.5/weather?q=yerbogachen,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 92:avarua, ck
    http://api.openweathermap.org/data/2.5/weather?q=avarua,ck&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 93:ocean springs, us
    http://api.openweathermap.org/data/2.5/weather?q=ocean springs,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 94:mar del plata, ar
    http://api.openweathermap.org/data/2.5/weather?q=mar del plata,ar&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 95:provideniya, ru
    http://api.openweathermap.org/data/2.5/weather?q=provideniya,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 96:illoqqortoormiut, gl
    http://api.openweathermap.org/data/2.5/weather?q=illoqqortoormiut,gl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 97:villa carlos paz, ar
    http://api.openweathermap.org/data/2.5/weather?q=villa carlos paz,ar&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 98:hofn, is
    http://api.openweathermap.org/data/2.5/weather?q=hofn,is&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 99:ushuaia, ar
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia,ar&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 100:barcelona, ve
    http://api.openweathermap.org/data/2.5/weather?q=barcelona,ve&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 101:guatire, ve
    http://api.openweathermap.org/data/2.5/weather?q=guatire,ve&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 102:hutchinson, us
    http://api.openweathermap.org/data/2.5/weather?q=hutchinson,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 103:vila, vu
    http://api.openweathermap.org/data/2.5/weather?q=vila,vu&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 104:sur, om
    http://api.openweathermap.org/data/2.5/weather?q=sur,om&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 105:upernavik, gl
    http://api.openweathermap.org/data/2.5/weather?q=upernavik,gl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 106:leiva, co
    http://api.openweathermap.org/data/2.5/weather?q=leiva,co&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 107:kaitangata, nz
    http://api.openweathermap.org/data/2.5/weather?q=kaitangata,nz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 108:saint-philippe, re
    http://api.openweathermap.org/data/2.5/weather?q=saint-philippe,re&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 109:tucuman, ar
    http://api.openweathermap.org/data/2.5/weather?q=tucuman,ar&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 110:longyearbyen, sj
    http://api.openweathermap.org/data/2.5/weather?q=longyearbyen,sj&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 111:victoria, sc
    http://api.openweathermap.org/data/2.5/weather?q=victoria,sc&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 112:north myrtle beach, us
    http://api.openweathermap.org/data/2.5/weather?q=north myrtle beach,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 113:zyryanka, ru
    http://api.openweathermap.org/data/2.5/weather?q=zyryanka,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 114:talnakh, ru
    http://api.openweathermap.org/data/2.5/weather?q=talnakh,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 115:tazovskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=tazovskiy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 116:berdigestyakh, ru
    http://api.openweathermap.org/data/2.5/weather?q=berdigestyakh,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 117:leningradskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=leningradskiy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 118:narsaq, gl
    http://api.openweathermap.org/data/2.5/weather?q=narsaq,gl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 119:moranbah, au
    http://api.openweathermap.org/data/2.5/weather?q=moranbah,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 120:bluff, nz
    http://api.openweathermap.org/data/2.5/weather?q=bluff,nz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 121:belmonte, br
    http://api.openweathermap.org/data/2.5/weather?q=belmonte,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 122:ponta delgada, pt
    http://api.openweathermap.org/data/2.5/weather?q=ponta delgada,pt&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 123:beringovskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=beringovskiy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 124:puerto ayora, ec
    http://api.openweathermap.org/data/2.5/weather?q=puerto ayora,ec&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 125:lipari, it
    http://api.openweathermap.org/data/2.5/weather?q=lipari,it&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 126:cabo san lucas, mx
    http://api.openweathermap.org/data/2.5/weather?q=cabo san lucas,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 127:urambo, tz
    http://api.openweathermap.org/data/2.5/weather?q=urambo,tz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 128:yuzhno-yeniseyskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=yuzhno-yeniseyskiy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 129:vaitape, pf
    http://api.openweathermap.org/data/2.5/weather?q=vaitape,pf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 130:barrow, us
    http://api.openweathermap.org/data/2.5/weather?q=barrow,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 131:constitucion, mx
    http://api.openweathermap.org/data/2.5/weather?q=constitucion,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 132:half moon bay, us
    http://api.openweathermap.org/data/2.5/weather?q=half moon bay,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 133:micheweni, tz
    http://api.openweathermap.org/data/2.5/weather?q=micheweni,tz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 134:mersin, tr
    http://api.openweathermap.org/data/2.5/weather?q=mersin,tr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 135:jalu, ly
    http://api.openweathermap.org/data/2.5/weather?q=jalu,ly&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 136:constanza, do
    http://api.openweathermap.org/data/2.5/weather?q=constanza,do&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 137:cape town, za
    http://api.openweathermap.org/data/2.5/weather?q=cape town,za&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 138:humberto de campos, br
    http://api.openweathermap.org/data/2.5/weather?q=humberto de campos,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 139:okha, ru
    http://api.openweathermap.org/data/2.5/weather?q=okha,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 140:poronaysk, ru
    http://api.openweathermap.org/data/2.5/weather?q=poronaysk,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 141:sept-iles, ca
    http://api.openweathermap.org/data/2.5/weather?q=sept-iles,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 142:ancud, cl
    http://api.openweathermap.org/data/2.5/weather?q=ancud,cl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 143:adrar, dz
    http://api.openweathermap.org/data/2.5/weather?q=adrar,dz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 144:sentyabrskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=sentyabrskiy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 145:atikokan, ca
    http://api.openweathermap.org/data/2.5/weather?q=atikokan,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 146:batsfjord, no
    http://api.openweathermap.org/data/2.5/weather?q=batsfjord,no&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 147:tuktoyaktuk, ca
    http://api.openweathermap.org/data/2.5/weather?q=tuktoyaktuk,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 148:rawson, ar
    http://api.openweathermap.org/data/2.5/weather?q=rawson,ar&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 149:mnogovershinnyy, ru
    http://api.openweathermap.org/data/2.5/weather?q=mnogovershinnyy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 150:maghama, mr
    http://api.openweathermap.org/data/2.5/weather?q=maghama,mr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 151:arraial do cabo, br
    http://api.openweathermap.org/data/2.5/weather?q=arraial do cabo,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 152:devils lake, us
    http://api.openweathermap.org/data/2.5/weather?q=devils lake,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 153:sorland, no
    http://api.openweathermap.org/data/2.5/weather?q=sorland,no&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 154:avera, pf
    http://api.openweathermap.org/data/2.5/weather?q=avera,pf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 155:nizhniy tsasuchey, ru
    http://api.openweathermap.org/data/2.5/weather?q=nizhniy tsasuchey,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 156:alyangula, au
    http://api.openweathermap.org/data/2.5/weather?q=alyangula,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 157:ambilobe, mg
    http://api.openweathermap.org/data/2.5/weather?q=ambilobe,mg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 158:thompson, ca
    http://api.openweathermap.org/data/2.5/weather?q=thompson,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 159:port elizabeth, za
    http://api.openweathermap.org/data/2.5/weather?q=port elizabeth,za&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 160:santa cruz de la palma, es
    http://api.openweathermap.org/data/2.5/weather?q=santa cruz de la palma,es&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 161:bonham, us
    http://api.openweathermap.org/data/2.5/weather?q=bonham,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 162:severo-kurilsk, ru
    http://api.openweathermap.org/data/2.5/weather?q=severo-kurilsk,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 163:ayagoz, kz
    http://api.openweathermap.org/data/2.5/weather?q=ayagoz,kz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 164:uvat, ru
    http://api.openweathermap.org/data/2.5/weather?q=uvat,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 165:meyungs, pw
    http://api.openweathermap.org/data/2.5/weather?q=meyungs,pw&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 166:buin, pg
    http://api.openweathermap.org/data/2.5/weather?q=buin,pg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 167:yulara, au
    http://api.openweathermap.org/data/2.5/weather?q=yulara,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 168:amparafaravola, mg
    http://api.openweathermap.org/data/2.5/weather?q=amparafaravola,mg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 169:rochegda, ru
    http://api.openweathermap.org/data/2.5/weather?q=rochegda,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 170:port hedland, au
    http://api.openweathermap.org/data/2.5/weather?q=port hedland,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 171:wajima, jp
    http://api.openweathermap.org/data/2.5/weather?q=wajima,jp&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 172:vostok, ru
    http://api.openweathermap.org/data/2.5/weather?q=vostok,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 173:ishigaki, jp
    http://api.openweathermap.org/data/2.5/weather?q=ishigaki,jp&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 174:cabedelo, br
    http://api.openweathermap.org/data/2.5/weather?q=cabedelo,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 175:athens, us
    http://api.openweathermap.org/data/2.5/weather?q=athens,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 176:flagstaff, us
    http://api.openweathermap.org/data/2.5/weather?q=flagstaff,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 177:mandalgovi, mn
    http://api.openweathermap.org/data/2.5/weather?q=mandalgovi,mn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 178:povenets, ru
    http://api.openweathermap.org/data/2.5/weather?q=povenets,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 179:grand river south east, mu
    http://api.openweathermap.org/data/2.5/weather?q=grand river south east,mu&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 180:green valley, us
    http://api.openweathermap.org/data/2.5/weather?q=green valley,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 181:lompoc, us
    http://api.openweathermap.org/data/2.5/weather?q=lompoc,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 182:hami, cn
    http://api.openweathermap.org/data/2.5/weather?q=hami,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 183:karaman, tr
    http://api.openweathermap.org/data/2.5/weather?q=karaman,tr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 184:mahebourg, mu
    http://api.openweathermap.org/data/2.5/weather?q=mahebourg,mu&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 185:balkanabat, tm
    http://api.openweathermap.org/data/2.5/weather?q=balkanabat,tm&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 186:kindia, gn
    http://api.openweathermap.org/data/2.5/weather?q=kindia,gn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 187:tabory, ru
    http://api.openweathermap.org/data/2.5/weather?q=tabory,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 188:jibuti, dj
    http://api.openweathermap.org/data/2.5/weather?q=jibuti,dj&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 189:saint-francois, gp
    http://api.openweathermap.org/data/2.5/weather?q=saint-francois,gp&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 190:tual, id
    http://api.openweathermap.org/data/2.5/weather?q=tual,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 191:las vegas, us
    http://api.openweathermap.org/data/2.5/weather?q=las vegas,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 192:dandong, cn
    http://api.openweathermap.org/data/2.5/weather?q=dandong,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 193:kloulklubed, pw
    http://api.openweathermap.org/data/2.5/weather?q=kloulklubed,pw&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 194:aranos, na
    http://api.openweathermap.org/data/2.5/weather?q=aranos,na&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 195:hervey bay, au
    http://api.openweathermap.org/data/2.5/weather?q=hervey bay,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 196:salalah, om
    http://api.openweathermap.org/data/2.5/weather?q=salalah,om&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 197:porosozero, ru
    http://api.openweathermap.org/data/2.5/weather?q=porosozero,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 198:jamestown, sh
    http://api.openweathermap.org/data/2.5/weather?q=jamestown,sh&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 199:uchaly, ru
    http://api.openweathermap.org/data/2.5/weather?q=uchaly,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 200:bentiu, sd
    http://api.openweathermap.org/data/2.5/weather?q=bentiu,sd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 201:kampene, cd
    http://api.openweathermap.org/data/2.5/weather?q=kampene,cd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 202:esperance, au
    http://api.openweathermap.org/data/2.5/weather?q=esperance,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 203:barentsburg, sj
    http://api.openweathermap.org/data/2.5/weather?q=barentsburg,sj&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 204:fenoarivo, mg
    http://api.openweathermap.org/data/2.5/weather?q=fenoarivo,mg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 205:champerico, gt
    http://api.openweathermap.org/data/2.5/weather?q=champerico,gt&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 206:mangai, cd
    http://api.openweathermap.org/data/2.5/weather?q=mangai,cd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 207:mukhen, ru
    http://api.openweathermap.org/data/2.5/weather?q=mukhen,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 208:suez, eg
    http://api.openweathermap.org/data/2.5/weather?q=suez,eg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 209:fort saint john, ca
    http://api.openweathermap.org/data/2.5/weather?q=fort saint john,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 210:sorong, id
    http://api.openweathermap.org/data/2.5/weather?q=sorong,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 211:plettenberg bay, za
    http://api.openweathermap.org/data/2.5/weather?q=plettenberg bay,za&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 212:sergeyevka, kz
    http://api.openweathermap.org/data/2.5/weather?q=sergeyevka,kz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 213:belushya guba, ru
    http://api.openweathermap.org/data/2.5/weather?q=belushya guba,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 214:coquimbo, cl
    http://api.openweathermap.org/data/2.5/weather?q=coquimbo,cl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 215:maniitsoq, gl
    http://api.openweathermap.org/data/2.5/weather?q=maniitsoq,gl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 216:ankazoabo, mg
    http://api.openweathermap.org/data/2.5/weather?q=ankazoabo,mg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 217:tahta, eg
    http://api.openweathermap.org/data/2.5/weather?q=tahta,eg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 218:atambua, id
    http://api.openweathermap.org/data/2.5/weather?q=atambua,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 219:iskateley, ru
    http://api.openweathermap.org/data/2.5/weather?q=iskateley,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 220:tres lagoas, br
    http://api.openweathermap.org/data/2.5/weather?q=tres lagoas,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 221:beatrice, us
    http://api.openweathermap.org/data/2.5/weather?q=beatrice,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 222:lagoa, pt
    http://api.openweathermap.org/data/2.5/weather?q=lagoa,pt&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 223:outjo, na
    http://api.openweathermap.org/data/2.5/weather?q=outjo,na&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 224:twentynine palms, us
    http://api.openweathermap.org/data/2.5/weather?q=twentynine palms,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 225:tsihombe, mg
    http://api.openweathermap.org/data/2.5/weather?q=tsihombe,mg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 226:port lincoln, au
    http://api.openweathermap.org/data/2.5/weather?q=port lincoln,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 227:tiarei, pf
    http://api.openweathermap.org/data/2.5/weather?q=tiarei,pf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 228:port hardy, ca
    http://api.openweathermap.org/data/2.5/weather?q=port hardy,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 229:mingshui, cn
    http://api.openweathermap.org/data/2.5/weather?q=mingshui,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 230:mount isa, au
    http://api.openweathermap.org/data/2.5/weather?q=mount isa,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 231:sao filipe, cv
    http://api.openweathermap.org/data/2.5/weather?q=sao filipe,cv&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 232:lufilufi, ws
    http://api.openweathermap.org/data/2.5/weather?q=lufilufi,ws&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 233:menongue, ao
    http://api.openweathermap.org/data/2.5/weather?q=menongue,ao&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 234:atar, mr
    http://api.openweathermap.org/data/2.5/weather?q=atar,mr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 235:lata, sb
    http://api.openweathermap.org/data/2.5/weather?q=lata,sb&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 236:attawapiskat, ca
    http://api.openweathermap.org/data/2.5/weather?q=attawapiskat,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 237:lerwick, gb
    http://api.openweathermap.org/data/2.5/weather?q=lerwick,gb&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 238:geraldton, au
    http://api.openweathermap.org/data/2.5/weather?q=geraldton,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 239:norman wells, ca
    http://api.openweathermap.org/data/2.5/weather?q=norman wells,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 240:catuday, ph
    http://api.openweathermap.org/data/2.5/weather?q=catuday,ph&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 241:tasiilaq, gl
    http://api.openweathermap.org/data/2.5/weather?q=tasiilaq,gl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 242:kawalu, id
    http://api.openweathermap.org/data/2.5/weather?q=kawalu,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 243:fortuna, us
    http://api.openweathermap.org/data/2.5/weather?q=fortuna,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 244:nandu, cn
    http://api.openweathermap.org/data/2.5/weather?q=nandu,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 245:hernani, ph
    http://api.openweathermap.org/data/2.5/weather?q=hernani,ph&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 246:los llanos de aridane, es
    http://api.openweathermap.org/data/2.5/weather?q=los llanos de aridane,es&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 247:quatre cocos, mu
    http://api.openweathermap.org/data/2.5/weather?q=quatre cocos,mu&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 248:ocos, gt
    http://api.openweathermap.org/data/2.5/weather?q=ocos,gt&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 249:faya, td
    http://api.openweathermap.org/data/2.5/weather?q=faya,td&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 250:haverfordwest, gb
    http://api.openweathermap.org/data/2.5/weather?q=haverfordwest,gb&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 251:canico, pt
    http://api.openweathermap.org/data/2.5/weather?q=canico,pt&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 252:hambantota, lk
    http://api.openweathermap.org/data/2.5/weather?q=hambantota,lk&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 253:isangel, vu
    http://api.openweathermap.org/data/2.5/weather?q=isangel,vu&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 254:husavik, is
    http://api.openweathermap.org/data/2.5/weather?q=husavik,is&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 255:austin, us
    http://api.openweathermap.org/data/2.5/weather?q=austin,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 256:soto, an
    http://api.openweathermap.org/data/2.5/weather?q=soto,an&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 257:dunedin, nz
    http://api.openweathermap.org/data/2.5/weather?q=dunedin,nz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 258:baoro, cf
    http://api.openweathermap.org/data/2.5/weather?q=baoro,cf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 259:mackenzie, ca
    http://api.openweathermap.org/data/2.5/weather?q=mackenzie,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 260:namatanai, pg
    http://api.openweathermap.org/data/2.5/weather?q=namatanai,pg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 261:harlingen, nl
    http://api.openweathermap.org/data/2.5/weather?q=harlingen,nl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 262:samusu, ws
    http://api.openweathermap.org/data/2.5/weather?q=samusu,ws&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 263:guerrero negro, mx
    http://api.openweathermap.org/data/2.5/weather?q=guerrero negro,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 264:garowe, so
    http://api.openweathermap.org/data/2.5/weather?q=garowe,so&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 265:san quintin, mx
    http://api.openweathermap.org/data/2.5/weather?q=san quintin,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 266:angoche, mz
    http://api.openweathermap.org/data/2.5/weather?q=angoche,mz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 267:deoria, in
    http://api.openweathermap.org/data/2.5/weather?q=deoria,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 268:nome, us
    http://api.openweathermap.org/data/2.5/weather?q=nome,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 269:abu dhabi, ae
    http://api.openweathermap.org/data/2.5/weather?q=abu dhabi,ae&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 270:morondava, mg
    http://api.openweathermap.org/data/2.5/weather?q=morondava,mg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 271:muriwai beach, nz
    http://api.openweathermap.org/data/2.5/weather?q=muriwai beach,nz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 272:padang, id
    http://api.openweathermap.org/data/2.5/weather?q=padang,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 273:eagle pass, us
    http://api.openweathermap.org/data/2.5/weather?q=eagle pass,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 274:acapulco, mx
    http://api.openweathermap.org/data/2.5/weather?q=acapulco,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 275:inhambane, mz
    http://api.openweathermap.org/data/2.5/weather?q=inhambane,mz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 276:castrovillari, it
    http://api.openweathermap.org/data/2.5/weather?q=castrovillari,it&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 277:sidney, us
    http://api.openweathermap.org/data/2.5/weather?q=sidney,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 278:khushab, pk
    http://api.openweathermap.org/data/2.5/weather?q=khushab,pk&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 279:qaanaaq, gl
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq,gl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 280:hovd, mn
    http://api.openweathermap.org/data/2.5/weather?q=hovd,mn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 281:bambari, cf
    http://api.openweathermap.org/data/2.5/weather?q=bambari,cf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 282:anchorage, us
    http://api.openweathermap.org/data/2.5/weather?q=anchorage,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 283:lodja, cd
    http://api.openweathermap.org/data/2.5/weather?q=lodja,cd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 284:vilaka, lv
    http://api.openweathermap.org/data/2.5/weather?q=vilaka,lv&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 285:belaya gora, ru
    http://api.openweathermap.org/data/2.5/weather?q=belaya gora,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 286:lavrentiya, ru
    http://api.openweathermap.org/data/2.5/weather?q=lavrentiya,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 287:griffith, au
    http://api.openweathermap.org/data/2.5/weather?q=griffith,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 288:roswell, us
    http://api.openweathermap.org/data/2.5/weather?q=roswell,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 289:lolua, tv
    http://api.openweathermap.org/data/2.5/weather?q=lolua,tv&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 290:batagay, ru
    http://api.openweathermap.org/data/2.5/weather?q=batagay,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 291:sungaipenuh, id
    http://api.openweathermap.org/data/2.5/weather?q=sungaipenuh,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 292:kidal, ml
    http://api.openweathermap.org/data/2.5/weather?q=kidal,ml&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 293:piney green, us
    http://api.openweathermap.org/data/2.5/weather?q=piney green,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 294:maine-soroa, ne
    http://api.openweathermap.org/data/2.5/weather?q=maine-soroa,ne&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 295:port macquarie, au
    http://api.openweathermap.org/data/2.5/weather?q=port macquarie,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 296:saleaula, ws
    http://api.openweathermap.org/data/2.5/weather?q=saleaula,ws&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 297:ijaki, ki
    http://api.openweathermap.org/data/2.5/weather?q=ijaki,ki&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 298:marzuq, ly
    http://api.openweathermap.org/data/2.5/weather?q=marzuq,ly&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 299:loikaw, mm
    http://api.openweathermap.org/data/2.5/weather?q=loikaw,mm&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 300:codrington, ag
    http://api.openweathermap.org/data/2.5/weather?q=codrington,ag&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 301:dawlatabad, af
    http://api.openweathermap.org/data/2.5/weather?q=dawlatabad,af&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 302:eureka, us
    http://api.openweathermap.org/data/2.5/weather?q=eureka,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 303:ambodifototra, mg
    http://api.openweathermap.org/data/2.5/weather?q=ambodifototra,mg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 304:fonte boa, br
    http://api.openweathermap.org/data/2.5/weather?q=fonte boa,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 305:fairbanks, us
    http://api.openweathermap.org/data/2.5/weather?q=fairbanks,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 306:kyabe, td
    http://api.openweathermap.org/data/2.5/weather?q=kyabe,td&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 307:hot springs, us
    http://api.openweathermap.org/data/2.5/weather?q=hot springs,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 308:trinidad, uy
    http://api.openweathermap.org/data/2.5/weather?q=trinidad,uy&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 309:banepa, np
    http://api.openweathermap.org/data/2.5/weather?q=banepa,np&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 310:plastun, ru
    http://api.openweathermap.org/data/2.5/weather?q=plastun,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 311:coihaique, cl
    http://api.openweathermap.org/data/2.5/weather?q=coihaique,cl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 312:orange cove, us
    http://api.openweathermap.org/data/2.5/weather?q=orange cove,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 313:katsuura, jp
    http://api.openweathermap.org/data/2.5/weather?q=katsuura,jp&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 314:teya, ru
    http://api.openweathermap.org/data/2.5/weather?q=teya,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 315:bargal, so
    http://api.openweathermap.org/data/2.5/weather?q=bargal,so&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 316:sorvag, fo
    http://api.openweathermap.org/data/2.5/weather?q=sorvag,fo&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 317:kuala terengganu, my
    http://api.openweathermap.org/data/2.5/weather?q=kuala terengganu,my&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 318:vaitupu, wf
    http://api.openweathermap.org/data/2.5/weather?q=vaitupu,wf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 319:honningsvag, no
    http://api.openweathermap.org/data/2.5/weather?q=honningsvag,no&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 320:kruisfontein, za
    http://api.openweathermap.org/data/2.5/weather?q=kruisfontein,za&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 321:alofi, nu
    http://api.openweathermap.org/data/2.5/weather?q=alofi,nu&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 322:clayton, us
    http://api.openweathermap.org/data/2.5/weather?q=clayton,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 323:chernyshkovskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=chernyshkovskiy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 324:korla, cn
    http://api.openweathermap.org/data/2.5/weather?q=korla,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 325:anadyr, ru
    http://api.openweathermap.org/data/2.5/weather?q=anadyr,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 326:meadow lake, ca
    http://api.openweathermap.org/data/2.5/weather?q=meadow lake,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 327:yangambi, cd
    http://api.openweathermap.org/data/2.5/weather?q=yangambi,cd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 328:raudeberg, no
    http://api.openweathermap.org/data/2.5/weather?q=raudeberg,no&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 329:kholodnyy, ru
    http://api.openweathermap.org/data/2.5/weather?q=kholodnyy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 330:brazzaville, cg
    http://api.openweathermap.org/data/2.5/weather?q=brazzaville,cg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 331:vila velha, br
    http://api.openweathermap.org/data/2.5/weather?q=vila velha,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 332:scottsburgh, za
    http://api.openweathermap.org/data/2.5/weather?q=scottsburgh,za&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 333:clarence town, bs
    http://api.openweathermap.org/data/2.5/weather?q=clarence town,bs&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 334:necochea, ar
    http://api.openweathermap.org/data/2.5/weather?q=necochea,ar&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 335:sakaiminato, jp
    http://api.openweathermap.org/data/2.5/weather?q=sakaiminato,jp&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 336:ranong, th
    http://api.openweathermap.org/data/2.5/weather?q=ranong,th&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 337:viedma, ar
    http://api.openweathermap.org/data/2.5/weather?q=viedma,ar&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 338:santa lucia, es
    http://api.openweathermap.org/data/2.5/weather?q=santa lucia,es&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 339:mackay, au
    http://api.openweathermap.org/data/2.5/weather?q=mackay,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 340:bogotol, ru
    http://api.openweathermap.org/data/2.5/weather?q=bogotol,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 341:ariyalur, in
    http://api.openweathermap.org/data/2.5/weather?q=ariyalur,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 342:neiafu, to
    http://api.openweathermap.org/data/2.5/weather?q=neiafu,to&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 343:beidao, cn
    http://api.openweathermap.org/data/2.5/weather?q=beidao,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 344:peleduy, ru
    http://api.openweathermap.org/data/2.5/weather?q=peleduy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 345:trento, it
    http://api.openweathermap.org/data/2.5/weather?q=trento,it&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 346:zhangjiakou, cn
    http://api.openweathermap.org/data/2.5/weather?q=zhangjiakou,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 347:hualmay, pe
    http://api.openweathermap.org/data/2.5/weather?q=hualmay,pe&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 348:klaksvik, fo
    http://api.openweathermap.org/data/2.5/weather?q=klaksvik,fo&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 349:kilindoni, tz
    http://api.openweathermap.org/data/2.5/weather?q=kilindoni,tz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 350:nizhniy baskunchak, ru
    http://api.openweathermap.org/data/2.5/weather?q=nizhniy baskunchak,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 351:san jose, gt
    http://api.openweathermap.org/data/2.5/weather?q=san jose,gt&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 352:shingu, jp
    http://api.openweathermap.org/data/2.5/weather?q=shingu,jp&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 353:kango, ga
    http://api.openweathermap.org/data/2.5/weather?q=kango,ga&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 354:formoso do araguaia, br
    http://api.openweathermap.org/data/2.5/weather?q=formoso do araguaia,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 355:henties bay, na
    http://api.openweathermap.org/data/2.5/weather?q=henties bay,na&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 356:richards bay, za
    http://api.openweathermap.org/data/2.5/weather?q=richards bay,za&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 357:sumbe, ao
    http://api.openweathermap.org/data/2.5/weather?q=sumbe,ao&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 358:zhigansk, ru
    http://api.openweathermap.org/data/2.5/weather?q=zhigansk,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 359:port-gentil, ga
    http://api.openweathermap.org/data/2.5/weather?q=port-gentil,ga&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 360:kavieng, pg
    http://api.openweathermap.org/data/2.5/weather?q=kavieng,pg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 361:touros, br
    http://api.openweathermap.org/data/2.5/weather?q=touros,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 362:hirara, jp
    http://api.openweathermap.org/data/2.5/weather?q=hirara,jp&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 363:mujiayingzi, cn
    http://api.openweathermap.org/data/2.5/weather?q=mujiayingzi,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 364:tumannyy, ru
    http://api.openweathermap.org/data/2.5/weather?q=tumannyy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 365:iqaluit, ca
    http://api.openweathermap.org/data/2.5/weather?q=iqaluit,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 366:les cayes, ht
    http://api.openweathermap.org/data/2.5/weather?q=les cayes,ht&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 367:soyo, ao
    http://api.openweathermap.org/data/2.5/weather?q=soyo,ao&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 368:nong phai, th
    http://api.openweathermap.org/data/2.5/weather?q=nong phai,th&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 369:yanam, in
    http://api.openweathermap.org/data/2.5/weather?q=yanam,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 370:manavgat, tr
    http://api.openweathermap.org/data/2.5/weather?q=manavgat,tr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 371:sombrio, br
    http://api.openweathermap.org/data/2.5/weather?q=sombrio,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 372:brezno, sk
    http://api.openweathermap.org/data/2.5/weather?q=brezno,sk&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 373:ahuimanu, us
    http://api.openweathermap.org/data/2.5/weather?q=ahuimanu,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 374:oktyabrskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=oktyabrskiy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 375:arlit, ne
    http://api.openweathermap.org/data/2.5/weather?q=arlit,ne&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 376:warkworth, nz
    http://api.openweathermap.org/data/2.5/weather?q=warkworth,nz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 377:maceio, br
    http://api.openweathermap.org/data/2.5/weather?q=maceio,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 378:san cristobal, ec
    http://api.openweathermap.org/data/2.5/weather?q=san cristobal,ec&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 379:muli, mv
    http://api.openweathermap.org/data/2.5/weather?q=muli,mv&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 380:awbari, ly
    http://api.openweathermap.org/data/2.5/weather?q=awbari,ly&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 381:portales, us
    http://api.openweathermap.org/data/2.5/weather?q=portales,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 382:santa fe, us
    http://api.openweathermap.org/data/2.5/weather?q=santa fe,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 383:tankhoy, ru
    http://api.openweathermap.org/data/2.5/weather?q=tankhoy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 384:mys shmidta, ru
    http://api.openweathermap.org/data/2.5/weather?q=mys shmidta,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 385:gushikawa, jp
    http://api.openweathermap.org/data/2.5/weather?q=gushikawa,jp&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 386:serta, pt
    http://api.openweathermap.org/data/2.5/weather?q=serta,pt&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 387:kargasok, ru
    http://api.openweathermap.org/data/2.5/weather?q=kargasok,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 388:beloha, mg
    http://api.openweathermap.org/data/2.5/weather?q=beloha,mg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 389:kodiak, us
    http://api.openweathermap.org/data/2.5/weather?q=kodiak,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 390:cesky krumlov, cz
    http://api.openweathermap.org/data/2.5/weather?q=cesky krumlov,cz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 391:kysyl-syr, ru
    http://api.openweathermap.org/data/2.5/weather?q=kysyl-syr,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 392:road town, vg
    http://api.openweathermap.org/data/2.5/weather?q=road town,vg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 393:terme, tr
    http://api.openweathermap.org/data/2.5/weather?q=terme,tr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 394:shebunino, ru
    http://api.openweathermap.org/data/2.5/weather?q=shebunino,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 395:boa vista, br
    http://api.openweathermap.org/data/2.5/weather?q=boa vista,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 396:karasburg, na
    http://api.openweathermap.org/data/2.5/weather?q=karasburg,na&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 397:sabang, id
    http://api.openweathermap.org/data/2.5/weather?q=sabang,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 398:halalo, wf
    http://api.openweathermap.org/data/2.5/weather?q=halalo,wf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 399:clyde river, ca
    http://api.openweathermap.org/data/2.5/weather?q=clyde river,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 400:pasni, pk
    http://api.openweathermap.org/data/2.5/weather?q=pasni,pk&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 401:berbera, so
    http://api.openweathermap.org/data/2.5/weather?q=berbera,so&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 402:hyeres, fr
    http://api.openweathermap.org/data/2.5/weather?q=hyeres,fr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 403:bodden town, ky
    http://api.openweathermap.org/data/2.5/weather?q=bodden town,ky&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 404:zagreb, hr
    http://api.openweathermap.org/data/2.5/weather?q=zagreb,hr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 405:aguimes, es
    http://api.openweathermap.org/data/2.5/weather?q=aguimes,es&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 406:kade, gh
    http://api.openweathermap.org/data/2.5/weather?q=kade,gh&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 407:antropovo, ru
    http://api.openweathermap.org/data/2.5/weather?q=antropovo,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 408:saint anthony, ca
    http://api.openweathermap.org/data/2.5/weather?q=saint anthony,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 409:micoud, lc
    http://api.openweathermap.org/data/2.5/weather?q=micoud,lc&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 410:narsingi, in
    http://api.openweathermap.org/data/2.5/weather?q=narsingi,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 411:humaita, br
    http://api.openweathermap.org/data/2.5/weather?q=humaita,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 412:sao borja, br
    http://api.openweathermap.org/data/2.5/weather?q=sao borja,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 413:chikoy, ru
    http://api.openweathermap.org/data/2.5/weather?q=chikoy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 414:esso, ru
    http://api.openweathermap.org/data/2.5/weather?q=esso,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 415:bubaque, gw
    http://api.openweathermap.org/data/2.5/weather?q=bubaque,gw&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 416:sovetskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=sovetskiy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 417:chuy, uy
    http://api.openweathermap.org/data/2.5/weather?q=chuy,uy&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 418:mazarron, es
    http://api.openweathermap.org/data/2.5/weather?q=mazarron,es&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 419:madarounfa, ne
    http://api.openweathermap.org/data/2.5/weather?q=madarounfa,ne&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 420:nioaque, br
    http://api.openweathermap.org/data/2.5/weather?q=nioaque,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 421:oskemen, kz
    http://api.openweathermap.org/data/2.5/weather?q=oskemen,kz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 422:sylva, ru
    http://api.openweathermap.org/data/2.5/weather?q=sylva,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 423:san felipe, mx
    http://api.openweathermap.org/data/2.5/weather?q=san felipe,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 424:tautira, pf
    http://api.openweathermap.org/data/2.5/weather?q=tautira,pf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 425:iquique, cl
    http://api.openweathermap.org/data/2.5/weather?q=iquique,cl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 426:norden, de
    http://api.openweathermap.org/data/2.5/weather?q=norden,de&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 427:luderitz, na
    http://api.openweathermap.org/data/2.5/weather?q=luderitz,na&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 428:biltine, td
    http://api.openweathermap.org/data/2.5/weather?q=biltine,td&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 429:mangan, in
    http://api.openweathermap.org/data/2.5/weather?q=mangan,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 430:srivardhan, in
    http://api.openweathermap.org/data/2.5/weather?q=srivardhan,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 431:chumikan, ru
    http://api.openweathermap.org/data/2.5/weather?q=chumikan,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 432:pionki, pl
    http://api.openweathermap.org/data/2.5/weather?q=pionki,pl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 433:tanout, ne
    http://api.openweathermap.org/data/2.5/weather?q=tanout,ne&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 434:saint-pierre, re
    http://api.openweathermap.org/data/2.5/weather?q=saint-pierre,re&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 435:sawtell, au
    http://api.openweathermap.org/data/2.5/weather?q=sawtell,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 436:havoysund, no
    http://api.openweathermap.org/data/2.5/weather?q=havoysund,no&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 437:chingirlau, kz
    http://api.openweathermap.org/data/2.5/weather?q=chingirlau,kz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 438:tezu, in
    http://api.openweathermap.org/data/2.5/weather?q=tezu,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 439:robat karim, ir
    http://api.openweathermap.org/data/2.5/weather?q=robat karim,ir&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 440:wattegama, lk
    http://api.openweathermap.org/data/2.5/weather?q=wattegama,lk&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 441:teguldet, ru
    http://api.openweathermap.org/data/2.5/weather?q=teguldet,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 442:saedinenie, bg
    http://api.openweathermap.org/data/2.5/weather?q=saedinenie,bg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 443:laguna, br
    http://api.openweathermap.org/data/2.5/weather?q=laguna,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 444:kahama, tz
    http://api.openweathermap.org/data/2.5/weather?q=kahama,tz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 445:nechi, co
    http://api.openweathermap.org/data/2.5/weather?q=nechi,co&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 446:taltal, cl
    http://api.openweathermap.org/data/2.5/weather?q=taltal,cl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 447:ormara, pk
    http://api.openweathermap.org/data/2.5/weather?q=ormara,pk&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 448:plouzane, fr
    http://api.openweathermap.org/data/2.5/weather?q=plouzane,fr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 449:setermoen, no
    http://api.openweathermap.org/data/2.5/weather?q=setermoen,no&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 450:olavarria, ar
    http://api.openweathermap.org/data/2.5/weather?q=olavarria,ar&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 451:cayenne, gf
    http://api.openweathermap.org/data/2.5/weather?q=cayenne,gf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 452:gat, ly
    http://api.openweathermap.org/data/2.5/weather?q=gat,ly&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 453:robe, et
    http://api.openweathermap.org/data/2.5/weather?q=robe,et&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 454:santa rosa, bo
    http://api.openweathermap.org/data/2.5/weather?q=santa rosa,bo&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 455:beyneu, kz
    http://api.openweathermap.org/data/2.5/weather?q=beyneu,kz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 456:sao joao da barra, br
    http://api.openweathermap.org/data/2.5/weather?q=sao joao da barra,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 457:igarka, ru
    http://api.openweathermap.org/data/2.5/weather?q=igarka,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 458:healesville, au
    http://api.openweathermap.org/data/2.5/weather?q=healesville,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 459:westport, ie
    http://api.openweathermap.org/data/2.5/weather?q=westport,ie&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 460:coahuayana, mx
    http://api.openweathermap.org/data/2.5/weather?q=coahuayana,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 461:qandahar, af
    http://api.openweathermap.org/data/2.5/weather?q=qandahar,af&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 462:talcahuano, cl
    http://api.openweathermap.org/data/2.5/weather?q=talcahuano,cl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 463:pochutla, mx
    http://api.openweathermap.org/data/2.5/weather?q=pochutla,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 464:ponta do sol, cv
    http://api.openweathermap.org/data/2.5/weather?q=ponta do sol,cv&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 465:karratha, au
    http://api.openweathermap.org/data/2.5/weather?q=karratha,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 466:hasaki, jp
    http://api.openweathermap.org/data/2.5/weather?q=hasaki,jp&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 467:honiara, sb
    http://api.openweathermap.org/data/2.5/weather?q=honiara,sb&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 468:georgetown, gy
    http://api.openweathermap.org/data/2.5/weather?q=georgetown,gy&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 469:kutum, sd
    http://api.openweathermap.org/data/2.5/weather?q=kutum,sd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 470:buchach, ua
    http://api.openweathermap.org/data/2.5/weather?q=buchach,ua&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 471:pevek, ru
    http://api.openweathermap.org/data/2.5/weather?q=pevek,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 472:tabiauea, ki
    http://api.openweathermap.org/data/2.5/weather?q=tabiauea,ki&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 473:selcuk, tr
    http://api.openweathermap.org/data/2.5/weather?q=selcuk,tr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 474:puquio, pe
    http://api.openweathermap.org/data/2.5/weather?q=puquio,pe&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 475:gazojak, tm
    http://api.openweathermap.org/data/2.5/weather?q=gazojak,tm&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 476:noumea, nc
    http://api.openweathermap.org/data/2.5/weather?q=noumea,nc&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 477:nuuk, gl
    http://api.openweathermap.org/data/2.5/weather?q=nuuk,gl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 478:ketchikan, us
    http://api.openweathermap.org/data/2.5/weather?q=ketchikan,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 479:anahuac, mx
    http://api.openweathermap.org/data/2.5/weather?q=anahuac,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 480:bouna, ci
    http://api.openweathermap.org/data/2.5/weather?q=bouna,ci&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 481:wanning, cn
    http://api.openweathermap.org/data/2.5/weather?q=wanning,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 482:auki, sb
    http://api.openweathermap.org/data/2.5/weather?q=auki,sb&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 483:arroyo, us
    http://api.openweathermap.org/data/2.5/weather?q=arroyo,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 484:puerto escondido, mx
    http://api.openweathermap.org/data/2.5/weather?q=puerto escondido,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 485:northam, au
    http://api.openweathermap.org/data/2.5/weather?q=northam,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 486:brewster, us
    http://api.openweathermap.org/data/2.5/weather?q=brewster,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 487:kuito, ao
    http://api.openweathermap.org/data/2.5/weather?q=kuito,ao&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 488:indi, in
    http://api.openweathermap.org/data/2.5/weather?q=indi,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 489:panuco, mx
    http://api.openweathermap.org/data/2.5/weather?q=panuco,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 490:pangkalanbuun, id
    http://api.openweathermap.org/data/2.5/weather?q=pangkalanbuun,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 491:santa cruz, cr
    http://api.openweathermap.org/data/2.5/weather?q=santa cruz,cr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 492:sabha, ly
    http://api.openweathermap.org/data/2.5/weather?q=sabha,ly&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 493:yaan, cn
    http://api.openweathermap.org/data/2.5/weather?q=yaan,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 494:cam ranh, vn
    http://api.openweathermap.org/data/2.5/weather?q=cam ranh,vn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 495:antoniny, ua
    http://api.openweathermap.org/data/2.5/weather?q=antoniny,ua&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 496:kartaly, ru
    http://api.openweathermap.org/data/2.5/weather?q=kartaly,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 497:orlik, ru
    http://api.openweathermap.org/data/2.5/weather?q=orlik,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 498:panguna, pg
    http://api.openweathermap.org/data/2.5/weather?q=panguna,pg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 499:maracacume, br
    http://api.openweathermap.org/data/2.5/weather?q=maracacume,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 500:khani, ru
    http://api.openweathermap.org/data/2.5/weather?q=khani,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 501:grand gaube, mu
    http://api.openweathermap.org/data/2.5/weather?q=grand gaube,mu&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 502:caravelas, br
    http://api.openweathermap.org/data/2.5/weather?q=caravelas,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 503:hearst, ca
    http://api.openweathermap.org/data/2.5/weather?q=hearst,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 504:paita, pe
    http://api.openweathermap.org/data/2.5/weather?q=paita,pe&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 505:galesong, id
    http://api.openweathermap.org/data/2.5/weather?q=galesong,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 506:mangaluru, in
    http://api.openweathermap.org/data/2.5/weather?q=mangaluru,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 507:hailar, cn
    http://api.openweathermap.org/data/2.5/weather?q=hailar,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 508:santa marta, co
    http://api.openweathermap.org/data/2.5/weather?q=santa marta,co&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 509:sonoita, mx
    http://api.openweathermap.org/data/2.5/weather?q=sonoita,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 510:umm kaddadah, sd
    http://api.openweathermap.org/data/2.5/weather?q=umm kaddadah,sd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 511:mitu, co
    http://api.openweathermap.org/data/2.5/weather?q=mitu,co&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 512:manggar, id
    http://api.openweathermap.org/data/2.5/weather?q=manggar,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 513:saraland, us
    http://api.openweathermap.org/data/2.5/weather?q=saraland,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 514:peniche, pt
    http://api.openweathermap.org/data/2.5/weather?q=peniche,pt&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 515:srednekolymsk, ru
    http://api.openweathermap.org/data/2.5/weather?q=srednekolymsk,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 516:ostrovnoy, ru
    http://api.openweathermap.org/data/2.5/weather?q=ostrovnoy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 517:linqing, cn
    http://api.openweathermap.org/data/2.5/weather?q=linqing,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 518:ituni, gy
    http://api.openweathermap.org/data/2.5/weather?q=ituni,gy&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 519:novoorsk, ru
    http://api.openweathermap.org/data/2.5/weather?q=novoorsk,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 520:svetlogorsk, ru
    http://api.openweathermap.org/data/2.5/weather?q=svetlogorsk,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 521:ambon, id
    http://api.openweathermap.org/data/2.5/weather?q=ambon,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 522:ballangen, no
    http://api.openweathermap.org/data/2.5/weather?q=ballangen,no&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 523:loiza, us
    http://api.openweathermap.org/data/2.5/weather?q=loiza,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 524:kokopo, pg
    http://api.openweathermap.org/data/2.5/weather?q=kokopo,pg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 525:marcona, pe
    http://api.openweathermap.org/data/2.5/weather?q=marcona,pe&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 526:willowmore, za
    http://api.openweathermap.org/data/2.5/weather?q=willowmore,za&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 527:mercedes, ar
    http://api.openweathermap.org/data/2.5/weather?q=mercedes,ar&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 528:atasu, kz
    http://api.openweathermap.org/data/2.5/weather?q=atasu,kz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 529:djambala, cg
    http://api.openweathermap.org/data/2.5/weather?q=djambala,cg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 530:nanortalik, gl
    http://api.openweathermap.org/data/2.5/weather?q=nanortalik,gl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 531:petauke, zm
    http://api.openweathermap.org/data/2.5/weather?q=petauke,zm&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 532:buchanan, lr
    http://api.openweathermap.org/data/2.5/weather?q=buchanan,lr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 533:chikmagalur, in
    http://api.openweathermap.org/data/2.5/weather?q=chikmagalur,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 534:baykit, ru
    http://api.openweathermap.org/data/2.5/weather?q=baykit,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 535:sisimiut, gl
    http://api.openweathermap.org/data/2.5/weather?q=sisimiut,gl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 536:antofagasta, cl
    http://api.openweathermap.org/data/2.5/weather?q=antofagasta,cl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 537:logansport, us
    http://api.openweathermap.org/data/2.5/weather?q=logansport,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 538:obo, cf
    http://api.openweathermap.org/data/2.5/weather?q=obo,cf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 539:bandar-e lengeh, ir
    http://api.openweathermap.org/data/2.5/weather?q=bandar-e lengeh,ir&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 540:wuwei, cn
    http://api.openweathermap.org/data/2.5/weather?q=wuwei,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 541:amderma, ru
    http://api.openweathermap.org/data/2.5/weather?q=amderma,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 542:pinega, ru
    http://api.openweathermap.org/data/2.5/weather?q=pinega,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 543:trairi, br
    http://api.openweathermap.org/data/2.5/weather?q=trairi,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 544:loudeac, fr
    http://api.openweathermap.org/data/2.5/weather?q=loudeac,fr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 545:lasa, cn
    http://api.openweathermap.org/data/2.5/weather?q=lasa,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 546:bukama, cd
    http://api.openweathermap.org/data/2.5/weather?q=bukama,cd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 547:vestmanna, fo
    http://api.openweathermap.org/data/2.5/weather?q=vestmanna,fo&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 548:canakkale, tr
    http://api.openweathermap.org/data/2.5/weather?q=canakkale,tr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 549:saint-augustin, ca
    http://api.openweathermap.org/data/2.5/weather?q=saint-augustin,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 550:bud, no
    http://api.openweathermap.org/data/2.5/weather?q=bud,no&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 551:bangassou, cf
    http://api.openweathermap.org/data/2.5/weather?q=bangassou,cf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 552:burnie, au
    http://api.openweathermap.org/data/2.5/weather?q=burnie,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 553:ostersund, se
    http://api.openweathermap.org/data/2.5/weather?q=ostersund,se&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 554:santa helena, br
    http://api.openweathermap.org/data/2.5/weather?q=santa helena,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 555:dolores, ar
    http://api.openweathermap.org/data/2.5/weather?q=dolores,ar&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 556:maningrida, au
    http://api.openweathermap.org/data/2.5/weather?q=maningrida,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 557:hunfeld, de
    http://api.openweathermap.org/data/2.5/weather?q=hunfeld,de&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 558:lithakia, gr
    http://api.openweathermap.org/data/2.5/weather?q=lithakia,gr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 559:ola, ru
    http://api.openweathermap.org/data/2.5/weather?q=ola,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 560:guilin, cn
    http://api.openweathermap.org/data/2.5/weather?q=guilin,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 561:tecoanapa, mx
    http://api.openweathermap.org/data/2.5/weather?q=tecoanapa,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 562:dubrovka, ru
    http://api.openweathermap.org/data/2.5/weather?q=dubrovka,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 563:haines junction, ca
    http://api.openweathermap.org/data/2.5/weather?q=haines junction,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 564:aloleng, ph
    http://api.openweathermap.org/data/2.5/weather?q=aloleng,ph&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 565:marawi, sd
    http://api.openweathermap.org/data/2.5/weather?q=marawi,sd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 566:taloda, in
    http://api.openweathermap.org/data/2.5/weather?q=taloda,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 567:kyle of lochalsh, gb
    http://api.openweathermap.org/data/2.5/weather?q=kyle of lochalsh,gb&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 568:aranda de duero, es
    http://api.openweathermap.org/data/2.5/weather?q=aranda de duero,es&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 569:flinders, au
    http://api.openweathermap.org/data/2.5/weather?q=flinders,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 570:petropavlovka, ru
    http://api.openweathermap.org/data/2.5/weather?q=petropavlovka,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 571:matagami, ca
    http://api.openweathermap.org/data/2.5/weather?q=matagami,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 572:volkermarkt, at
    http://api.openweathermap.org/data/2.5/weather?q=volkermarkt,at&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 573:manta, ec
    http://api.openweathermap.org/data/2.5/weather?q=manta,ec&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 574:aklavik, ca
    http://api.openweathermap.org/data/2.5/weather?q=aklavik,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 575:mbanza-ngungu, cd
    http://api.openweathermap.org/data/2.5/weather?q=mbanza-ngungu,cd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 576:rawannawi, ki
    http://api.openweathermap.org/data/2.5/weather?q=rawannawi,ki&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 577:kiunga, pg
    http://api.openweathermap.org/data/2.5/weather?q=kiunga,pg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 578:caarapo, br
    http://api.openweathermap.org/data/2.5/weather?q=caarapo,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 579:pauini, br
    http://api.openweathermap.org/data/2.5/weather?q=pauini,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 580:ankang, cn
    http://api.openweathermap.org/data/2.5/weather?q=ankang,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 581:dingle, ie
    http://api.openweathermap.org/data/2.5/weather?q=dingle,ie&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 582:geraldton, ca
    http://api.openweathermap.org/data/2.5/weather?q=geraldton,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 583:vakhtan, ru
    http://api.openweathermap.org/data/2.5/weather?q=vakhtan,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 584:bereda, so
    http://api.openweathermap.org/data/2.5/weather?q=bereda,so&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 585:awjilah, ly
    http://api.openweathermap.org/data/2.5/weather?q=awjilah,ly&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 586:ondorhaan, mn
    http://api.openweathermap.org/data/2.5/weather?q=ondorhaan,mn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 587:goundam, ml
    http://api.openweathermap.org/data/2.5/weather?q=goundam,ml&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 588:rolim de moura, br
    http://api.openweathermap.org/data/2.5/weather?q=rolim de moura,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 589:rio gallegos, ar
    http://api.openweathermap.org/data/2.5/weather?q=rio gallegos,ar&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 590:asau, tv
    http://api.openweathermap.org/data/2.5/weather?q=asau,tv&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 591:bouca, cf
    http://api.openweathermap.org/data/2.5/weather?q=bouca,cf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 592:saldanha, za
    http://api.openweathermap.org/data/2.5/weather?q=saldanha,za&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 593:rio grande, br
    http://api.openweathermap.org/data/2.5/weather?q=rio grande,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 594:westford, us
    http://api.openweathermap.org/data/2.5/weather?q=westford,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 595:lubeck, de
    http://api.openweathermap.org/data/2.5/weather?q=lubeck,de&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 596:rungata, ki
    http://api.openweathermap.org/data/2.5/weather?q=rungata,ki&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 597:mehamn, no
    http://api.openweathermap.org/data/2.5/weather?q=mehamn,no&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 598:vila franca do campo, pt
    http://api.openweathermap.org/data/2.5/weather?q=vila franca do campo,pt&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 599:ayan, ru
    http://api.openweathermap.org/data/2.5/weather?q=ayan,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 600:nanakuli, us
    http://api.openweathermap.org/data/2.5/weather?q=nanakuli,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 601:salinas, ec
    http://api.openweathermap.org/data/2.5/weather?q=salinas,ec&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 602:kisangani, cd
    http://api.openweathermap.org/data/2.5/weather?q=kisangani,cd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 603:tamasopo, mx
    http://api.openweathermap.org/data/2.5/weather?q=tamasopo,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 604:we, nc
    http://api.openweathermap.org/data/2.5/weather?q=we,nc&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 605:palabuhanratu, id
    http://api.openweathermap.org/data/2.5/weather?q=palabuhanratu,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 606:alexandria, eg
    http://api.openweathermap.org/data/2.5/weather?q=alexandria,eg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 607:palana, ru
    http://api.openweathermap.org/data/2.5/weather?q=palana,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 608:dingzhou, cn
    http://api.openweathermap.org/data/2.5/weather?q=dingzhou,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 609:grindavik, is
    http://api.openweathermap.org/data/2.5/weather?q=grindavik,is&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 610:sola, vu
    http://api.openweathermap.org/data/2.5/weather?q=sola,vu&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 611:tilichiki, ru
    http://api.openweathermap.org/data/2.5/weather?q=tilichiki,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 612:karaul, ru
    http://api.openweathermap.org/data/2.5/weather?q=karaul,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 613:ozinki, ru
    http://api.openweathermap.org/data/2.5/weather?q=ozinki,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 614:roma, au
    http://api.openweathermap.org/data/2.5/weather?q=roma,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 615:jiddah, sa
    http://api.openweathermap.org/data/2.5/weather?q=jiddah,sa&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 616:blagoveshchensk, ru
    http://api.openweathermap.org/data/2.5/weather?q=blagoveshchensk,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 617:fort nelson, ca
    http://api.openweathermap.org/data/2.5/weather?q=fort nelson,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 618:kawambwa, zm
    http://api.openweathermap.org/data/2.5/weather?q=kawambwa,zm&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 619:linxia, cn
    http://api.openweathermap.org/data/2.5/weather?q=linxia,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 620:zhaoyuan, cn
    http://api.openweathermap.org/data/2.5/weather?q=zhaoyuan,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 621:lagunas, pe
    http://api.openweathermap.org/data/2.5/weather?q=lagunas,pe&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 622:makokou, ga
    http://api.openweathermap.org/data/2.5/weather?q=makokou,ga&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 623:constitucion, cl
    http://api.openweathermap.org/data/2.5/weather?q=constitucion,cl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 624:zeya, ru
    http://api.openweathermap.org/data/2.5/weather?q=zeya,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 625:moussoro, td
    http://api.openweathermap.org/data/2.5/weather?q=moussoro,td&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 626:puerto baquerizo moreno, ec
    http://api.openweathermap.org/data/2.5/weather?q=puerto baquerizo moreno,ec&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 627:cilegon, id
    http://api.openweathermap.org/data/2.5/weather?q=cilegon,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 628:kieta, pg
    http://api.openweathermap.org/data/2.5/weather?q=kieta,pg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 629:chelles, fr
    http://api.openweathermap.org/data/2.5/weather?q=chelles,fr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 630:svetlyy, ru
    http://api.openweathermap.org/data/2.5/weather?q=svetlyy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 631:abnub, eg
    http://api.openweathermap.org/data/2.5/weather?q=abnub,eg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 632:muroto, jp
    http://api.openweathermap.org/data/2.5/weather?q=muroto,jp&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 633:dibaya, cd
    http://api.openweathermap.org/data/2.5/weather?q=dibaya,cd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 634:nador, ma
    http://api.openweathermap.org/data/2.5/weather?q=nador,ma&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 635:vardo, no
    http://api.openweathermap.org/data/2.5/weather?q=vardo,no&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 636:saint george, bm
    http://api.openweathermap.org/data/2.5/weather?q=saint george,bm&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 637:kpandae, gh
    http://api.openweathermap.org/data/2.5/weather?q=kpandae,gh&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 638:dekar, bw
    http://api.openweathermap.org/data/2.5/weather?q=dekar,bw&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 639:komsomolets, kz
    http://api.openweathermap.org/data/2.5/weather?q=komsomolets,kz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 640:vao, nc
    http://api.openweathermap.org/data/2.5/weather?q=vao,nc&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 641:khorramshahr, ir
    http://api.openweathermap.org/data/2.5/weather?q=khorramshahr,ir&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 642:macenta, gn
    http://api.openweathermap.org/data/2.5/weather?q=macenta,gn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 643:magadan, ru
    http://api.openweathermap.org/data/2.5/weather?q=magadan,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 644:udachnyy, ru
    http://api.openweathermap.org/data/2.5/weather?q=udachnyy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 645:mayo, ca
    http://api.openweathermap.org/data/2.5/weather?q=mayo,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 646:nerja, es
    http://api.openweathermap.org/data/2.5/weather?q=nerja,es&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 647:kirakira, sb
    http://api.openweathermap.org/data/2.5/weather?q=kirakira,sb&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 648:zanjan, ir
    http://api.openweathermap.org/data/2.5/weather?q=zanjan,ir&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 649:ternate, id
    http://api.openweathermap.org/data/2.5/weather?q=ternate,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 650:axixa do tocantins, br
    http://api.openweathermap.org/data/2.5/weather?q=axixa do tocantins,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 651:miquelon, pm
    http://api.openweathermap.org/data/2.5/weather?q=miquelon,pm&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 652:otavi, na
    http://api.openweathermap.org/data/2.5/weather?q=otavi,na&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 653:comodoro rivadavia, ar
    http://api.openweathermap.org/data/2.5/weather?q=comodoro rivadavia,ar&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 654:gopalpur, in
    http://api.openweathermap.org/data/2.5/weather?q=gopalpur,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 655:mezen, ru
    http://api.openweathermap.org/data/2.5/weather?q=mezen,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 656:patiya, bd
    http://api.openweathermap.org/data/2.5/weather?q=patiya,bd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 657:sao gabriel da cachoeira, br
    http://api.openweathermap.org/data/2.5/weather?q=sao gabriel da cachoeira,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 658:umm lajj, sa
    http://api.openweathermap.org/data/2.5/weather?q=umm lajj,sa&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 659:slyudyanka, ru
    http://api.openweathermap.org/data/2.5/weather?q=slyudyanka,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 660:bethel, us
    http://api.openweathermap.org/data/2.5/weather?q=bethel,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 661:cockburn town, bs
    http://api.openweathermap.org/data/2.5/weather?q=cockburn town,bs&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 662:biberach, de
    http://api.openweathermap.org/data/2.5/weather?q=biberach,de&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 663:krasnoarmeysk, kz
    http://api.openweathermap.org/data/2.5/weather?q=krasnoarmeysk,kz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 664:urengoy, ru
    http://api.openweathermap.org/data/2.5/weather?q=urengoy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 665:celestun, mx
    http://api.openweathermap.org/data/2.5/weather?q=celestun,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 666:paka, my
    http://api.openweathermap.org/data/2.5/weather?q=paka,my&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 667:xadani, mx
    http://api.openweathermap.org/data/2.5/weather?q=xadani,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 668:borogontsy, ru
    http://api.openweathermap.org/data/2.5/weather?q=borogontsy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 669:xihe, cn
    http://api.openweathermap.org/data/2.5/weather?q=xihe,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 670:gardiner, us
    http://api.openweathermap.org/data/2.5/weather?q=gardiner,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 671:maykain, kz
    http://api.openweathermap.org/data/2.5/weather?q=maykain,kz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 672:taksimo, ru
    http://api.openweathermap.org/data/2.5/weather?q=taksimo,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 673:bayan, kw
    http://api.openweathermap.org/data/2.5/weather?q=bayan,kw&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 674:ust-kuyga, ru
    http://api.openweathermap.org/data/2.5/weather?q=ust-kuyga,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 675:kimbe, pg
    http://api.openweathermap.org/data/2.5/weather?q=kimbe,pg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 676:baindur, in
    http://api.openweathermap.org/data/2.5/weather?q=baindur,in&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 677:armacao dos buzios, br
    http://api.openweathermap.org/data/2.5/weather?q=armacao dos buzios,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 678:lichinga, mz
    http://api.openweathermap.org/data/2.5/weather?q=lichinga,mz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 679:mao, td
    http://api.openweathermap.org/data/2.5/weather?q=mao,td&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 680:adeje, es
    http://api.openweathermap.org/data/2.5/weather?q=adeje,es&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 681:macusani, pe
    http://api.openweathermap.org/data/2.5/weather?q=macusani,pe&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 682:pisco, pe
    http://api.openweathermap.org/data/2.5/weather?q=pisco,pe&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 683:sungairaya, id
    http://api.openweathermap.org/data/2.5/weather?q=sungairaya,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 684:nantucket, us
    http://api.openweathermap.org/data/2.5/weather?q=nantucket,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 685:andenes, no
    http://api.openweathermap.org/data/2.5/weather?q=andenes,no&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 686:scarborough, tt
    http://api.openweathermap.org/data/2.5/weather?q=scarborough,tt&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 687:doctor pedro p. pena, py
    http://api.openweathermap.org/data/2.5/weather?q=doctor pedro p. pena,py&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 688:launceston, au
    http://api.openweathermap.org/data/2.5/weather?q=launceston,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 689:copiapo, cl
    http://api.openweathermap.org/data/2.5/weather?q=copiapo,cl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 690:saint-gaudens, fr
    http://api.openweathermap.org/data/2.5/weather?q=saint-gaudens,fr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 691:hammerfest, no
    http://api.openweathermap.org/data/2.5/weather?q=hammerfest,no&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 692:khandyga, ru
    http://api.openweathermap.org/data/2.5/weather?q=khandyga,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 693:gbely, sk
    http://api.openweathermap.org/data/2.5/weather?q=gbely,sk&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 694:mastic beach, us
    http://api.openweathermap.org/data/2.5/weather?q=mastic beach,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 695:sao jose da coroa grande, br
    http://api.openweathermap.org/data/2.5/weather?q=sao jose da coroa grande,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 696:waddan, ly
    http://api.openweathermap.org/data/2.5/weather?q=waddan,ly&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 697:redlands, us
    http://api.openweathermap.org/data/2.5/weather?q=redlands,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 698:angoram, pg
    http://api.openweathermap.org/data/2.5/weather?q=angoram,pg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 699:bonavista, ca
    http://api.openweathermap.org/data/2.5/weather?q=bonavista,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 700:barbar, sd
    http://api.openweathermap.org/data/2.5/weather?q=barbar,sd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 701:kiruna, se
    http://api.openweathermap.org/data/2.5/weather?q=kiruna,se&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 702:pangnirtung, ca
    http://api.openweathermap.org/data/2.5/weather?q=pangnirtung,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 703:chagda, ru
    http://api.openweathermap.org/data/2.5/weather?q=chagda,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 704:campbell river, ca
    http://api.openweathermap.org/data/2.5/weather?q=campbell river,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 705:paamiut, gl
    http://api.openweathermap.org/data/2.5/weather?q=paamiut,gl&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 706:idaho falls, us
    http://api.openweathermap.org/data/2.5/weather?q=idaho falls,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 707:liku, wf
    http://api.openweathermap.org/data/2.5/weather?q=liku,wf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 708:cumpas, mx
    http://api.openweathermap.org/data/2.5/weather?q=cumpas,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 709:tarudant, ma
    http://api.openweathermap.org/data/2.5/weather?q=tarudant,ma&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 710:banmo, mm
    http://api.openweathermap.org/data/2.5/weather?q=banmo,mm&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 711:wurzen, de
    http://api.openweathermap.org/data/2.5/weather?q=wurzen,de&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 712:longhua, cn
    http://api.openweathermap.org/data/2.5/weather?q=longhua,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 713:shawville, ca
    http://api.openweathermap.org/data/2.5/weather?q=shawville,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 714:sinnamary, gf
    http://api.openweathermap.org/data/2.5/weather?q=sinnamary,gf&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 715:togur, ru
    http://api.openweathermap.org/data/2.5/weather?q=togur,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 716:mahon, es
    http://api.openweathermap.org/data/2.5/weather?q=mahon,es&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 717:cockburn town, tc
    http://api.openweathermap.org/data/2.5/weather?q=cockburn town,tc&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 718:egvekinot, ru
    http://api.openweathermap.org/data/2.5/weather?q=egvekinot,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 719:oros, br
    http://api.openweathermap.org/data/2.5/weather?q=oros,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 720:angouleme, fr
    http://api.openweathermap.org/data/2.5/weather?q=angouleme,fr&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 721:kenai, us
    http://api.openweathermap.org/data/2.5/weather?q=kenai,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 722:qesarya, il
    http://api.openweathermap.org/data/2.5/weather?q=qesarya,il&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 723:tupik, ru
    http://api.openweathermap.org/data/2.5/weather?q=tupik,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 724:hornepayne, ca
    http://api.openweathermap.org/data/2.5/weather?q=hornepayne,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 725:sumbawa, id
    http://api.openweathermap.org/data/2.5/weather?q=sumbawa,id&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 726:weligama, lk
    http://api.openweathermap.org/data/2.5/weather?q=weligama,lk&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 727:komsomolskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=komsomolskiy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 728:doka, sd
    http://api.openweathermap.org/data/2.5/weather?q=doka,sd&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 729:luocheng, cn
    http://api.openweathermap.org/data/2.5/weather?q=luocheng,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 730:anar darreh, af
    http://api.openweathermap.org/data/2.5/weather?q=anar darreh,af&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 731:akyab, mm
    http://api.openweathermap.org/data/2.5/weather?q=akyab,mm&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 732:artigas, uy
    http://api.openweathermap.org/data/2.5/weather?q=artigas,uy&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 733:opuwo, na
    http://api.openweathermap.org/data/2.5/weather?q=opuwo,na&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 734:rincon, us
    http://api.openweathermap.org/data/2.5/weather?q=rincon,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 735:sayanskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=sayanskiy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 736:riyadh, sa
    http://api.openweathermap.org/data/2.5/weather?q=riyadh,sa&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 737:oriximina, br
    http://api.openweathermap.org/data/2.5/weather?q=oriximina,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 738:manzhouli, cn
    http://api.openweathermap.org/data/2.5/weather?q=manzhouli,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 739:hanover, us
    http://api.openweathermap.org/data/2.5/weather?q=hanover,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 740:takoradi, gh
    http://api.openweathermap.org/data/2.5/weather?q=takoradi,gh&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 741:copala, mx
    http://api.openweathermap.org/data/2.5/weather?q=copala,mx&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 742:westport, nz
    http://api.openweathermap.org/data/2.5/weather?q=westport,nz&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 743:xiongzhou, cn
    http://api.openweathermap.org/data/2.5/weather?q=xiongzhou,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 744:strezhevoy, ru
    http://api.openweathermap.org/data/2.5/weather?q=strezhevoy,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 745:imbituba, br
    http://api.openweathermap.org/data/2.5/weather?q=imbituba,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 746:bandarbeyla, so
    http://api.openweathermap.org/data/2.5/weather?q=bandarbeyla,so&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 747:ambanja, mg
    http://api.openweathermap.org/data/2.5/weather?q=ambanja,mg&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 748:bathsheba, bb
    http://api.openweathermap.org/data/2.5/weather?q=bathsheba,bb&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 749:mandera, ke
    http://api.openweathermap.org/data/2.5/weather?q=mandera,ke&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 750:chimbote, pe
    http://api.openweathermap.org/data/2.5/weather?q=chimbote,pe&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 751:synya, ru
    http://api.openweathermap.org/data/2.5/weather?q=synya,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 752:surt, ly
    http://api.openweathermap.org/data/2.5/weather?q=surt,ly&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 753:galiwinku, au
    http://api.openweathermap.org/data/2.5/weather?q=galiwinku,au&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 754:shasta lake, us
    http://api.openweathermap.org/data/2.5/weather?q=shasta lake,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 755:mubende, ug
    http://api.openweathermap.org/data/2.5/weather?q=mubende,ug&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 756:cabo rojo, us
    http://api.openweathermap.org/data/2.5/weather?q=cabo rojo,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 757:itaituba, br
    http://api.openweathermap.org/data/2.5/weather?q=itaituba,br&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 758:saint-michel-des-saints, ca
    http://api.openweathermap.org/data/2.5/weather?q=saint-michel-des-saints,ca&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 759:luanda, ao
    http://api.openweathermap.org/data/2.5/weather?q=luanda,ao&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 760:mamonovo, ru
    http://api.openweathermap.org/data/2.5/weather?q=mamonovo,ru&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 761:qui nhon, vn
    http://api.openweathermap.org/data/2.5/weather?q=qui nhon,vn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Error with city data. Skipping
    Retriving city# 762:ugoofaaru, mv
    http://api.openweathermap.org/data/2.5/weather?q=ugoofaaru,mv&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 763:saint-paul, re
    http://api.openweathermap.org/data/2.5/weather?q=saint-paul,re&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 764:tromso, no
    http://api.openweathermap.org/data/2.5/weather?q=tromso,no&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 765:yumen, cn
    http://api.openweathermap.org/data/2.5/weather?q=yumen,cn&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 766:tripoli, ly
    http://api.openweathermap.org/data/2.5/weather?q=tripoli,ly&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    Retriving city# 767:newport, us
    http://api.openweathermap.org/data/2.5/weather?q=newport,us&units=imperial&appid=9f07c20b520c6f77a821a921c2e8ce64
    ------------------------------------
    Data Retrieval Complete
    ------------------------------------
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Country</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Temperature (F)</th>
      <th>Wind Speed (mph)</th>
      <th>Humidity %</th>
      <th>Cloudiness %</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>new norfolk</td>
      <td>au</td>
      <td>-42.78</td>
      <td>147.06</td>
      <td>44.6</td>
      <td>10.29</td>
      <td>87</td>
      <td>75</td>
    </tr>
    <tr>
      <th>1</th>
      <td>punta arenas</td>
      <td>cl</td>
      <td>-53.16</td>
      <td>-70.91</td>
      <td>33.8</td>
      <td>10.29</td>
      <td>100</td>
      <td>75</td>
    </tr>
    <tr>
      <th>2</th>
      <td>montclair</td>
      <td>us</td>
      <td>34.08</td>
      <td>-117.69</td>
      <td>85.41</td>
      <td>18.34</td>
      <td>33</td>
      <td>75</td>
    </tr>
    <tr>
      <th>3</th>
      <td>cidreira</td>
      <td>br</td>
      <td>-30.17</td>
      <td>-50.22</td>
      <td>47.06</td>
      <td>3.83</td>
      <td>91</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>hithadhoo</td>
      <td>mv</td>
      <td>-0.6</td>
      <td>73.08</td>
      <td>81.71</td>
      <td>6.4</td>
      <td>100</td>
      <td>68</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Check if output folder exist, create if it doesn't exists!

filePath = 'Weather_analysis'
if not path.exists(filePath):
    makedirs(filePath)

# Save the result to a csv file
city_df.to_csv("Weather_analysis/Weather_analysis.csv", index=False)
```


```python
# Count number of values in the dataframe

# city_df.count()
# city_df.dtypes

# Converting object types to float types
city_df["Latitude"] = pd.to_numeric(city_df["Latitude"])
city_df["Longitude"] = pd.to_numeric(city_df["Longitude"])
city_df["Temperature (F)"] = pd.to_numeric(city_df["Temperature (F)"])
city_df["Wind Speed (mph)"] = pd.to_numeric(city_df["Wind Speed (mph)"])
city_df["Humidity %"] = pd.to_numeric(city_df["Humidity %"])
city_df["Cloudiness %"] = pd.to_numeric(city_df["Cloudiness %"])


city_df.dtypes
city_df.count()
```




    City                768
    Country             768
    Latitude            674
    Longitude           674
    Temperature (F)     674
    Wind Speed (mph)    674
    Humidity %          674
    Cloudiness %        674
    dtype: int64




```python
# Drop NA values and create a new dataframe
new_city_df = city_df.dropna(how='any') 
new_city_df.count()
```




    City                674
    Country             674
    Latitude            674
    Longitude           674
    Temperature (F)     674
    Wind Speed (mph)    674
    Humidity %          674
    Cloudiness %        674
    dtype: int64




```python
# city_df_1 = pd.read_csv()
```


```python
# Creating a function to set the properties of the graph
def plot_graph(x_label,y_label,x_limits,save_file_name):
    plt.xlabel(x_label)
    plt.ylabel(y_label)
    plt.xlim(x_limits)
    plt.title("%s vs %s (%s/%s/%s)"%(x_label,y_label,now.month,now.day,now.year),fontsize=14)
    plt.grid(True)
    plt.savefig(save_file_name)
    plt.show()
```

# Latitude vs Temperature(F)


```python
# Latitude vs Temperature(F)
plt.scatter(x=new_city_df["Latitude"],y=new_city_df["Temperature (F)"],facecolors="blue",edgecolors="black")  
plot_graph("Latitude","Temperature (F)",[-90,90],"Weather_analysis/City Latitude vs Temperature.png")
```


![png](output_12_0.png)


# Latitude vs Humidity% 


```python
# # Latitude vs Humidity in % 
plt.scatter(x=new_city_df["Latitude"],y=new_city_df["Humidity %"],facecolors="blue",edgecolors="black")
plot_graph("Latitude","Humidity(%)",[-90,90],"Weather_analysis/City Latitude vs Humidity.png")
```


![png](output_14_0.png)


# Latitude vs. Cloudiness %


```python
# Latitude vs Cloudliness in %
plt.scatter(x=new_city_df["Latitude"],y=new_city_df["Cloudiness %"],facecolors="blue",edgecolors="black")
plot_graph("Latitude","Cloudliness(%)",[-90,90],"Weather_analysis/City Latitude vs Cloudliness.png")
```


![png](output_16_0.png)


# Latitude vs Wind Speed in (mph)


```python
# Latitude vs Wind Speed in (mph)
plt.scatter(x=city_df["Latitude"],y=city_df["Wind Speed (mph)"],facecolors="blue",edgecolors="black")
plot_graph("Latitude","Wind Speed (mph)",[-90,90],"Weather_analysis/City Latitude vs Wind speed.png")
```


![png](output_18_0.png)

