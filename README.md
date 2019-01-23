# The burden of aircraft noise over West Boston

In the summer of 2018, as part of [#ARforChange](http://www.storybench.org/arforchange-project-using-immersive-technologies-data-social-change/), a project exploring new platforms for civic engagement and social change using immersive media at Northeastern University's School of Journalism, I collected flight data and presented them through heatmaps to visualize the unfair burden of aircraft noise over West Boston with Python.

## Data
Data are from [Kent Johnson](http://kentsj.com/BWFS/Logan_33L_Flight_Tracks_2013_vs_2015.html), a member of [Boston West Fair Skies](https://www.bostonwestfairskies.org/news.html), an advocacy group aiming to address this issue.

## Loading packages
```python
import geopandas as gpd
import numpy as np
import pandas as pd
```

## Loading and processing data
```python
Jan_2013 = '...../Jan2015.gpkg' #data from Jan 2013
data_2013 = gpd.read_file(Jan_2013)
```

## Creating heatmaps
I created two heatmaps of airplane counts and heights. The first one shows the mean heights of flights over a 500x500 grid over parts Boston area near Logan Airport, while the second one shows the number of counts of flights in each 500x500 cell.
```python
Lon = np.arange(-71.33, -70.7, 0.00126) #this divides the longitude range we want into 500 equal intervals; we do this because we want to take mean values of heights in those 500x500 small cells in the 500x500 grid to plot the heatmap
Lat = np.arange(42.1, 42.6, 0.001) #this divides the latitude range we want into 500 equal intervals
```

Here, I am creating an empty list to store heights.
```python
Heights = []

for i in range(500):
    column=[]
    for j in range(500):
        column.append([])
    Heights.append(column)
```

Here we process and store the data.
```python
for a in range(0,24500): #there are 24500 data points
    for b in range(0,len(data['geometry'].values[a].coords[:][:])): #this is the length of the linestring for each data point
        for c1 in range (0,500):
            if Lat[c1] - 0.00063 <= data['geometry'].values[a].coords[b][0] < Lat[c1] + 0.00063: #we are making the 500 latitude and longnitude points in Lat and Lon the center of the small cells here, that's why we have plus and minus half of the change of lat/lon per small cell (the small cells make up the grid that we will need for the heat map)
                for c2 in range (0,500):
                    if Lon[c2] - 0.0005 <= data['geometry'].values[a].coords[b][1] < Lon[c2] + 0.0005:
                        Heights[c1][c2].append(data['geometry'].values[a].coords[b][2]) # storing data in "Heights"
                        break
```

Here, we are creating arrays of 0s that are 500 wide and 500 tall. Then we store the mean values of heights for each small cell and counts in those arrays.
```python
Mean = np.zeros((500,500))
Counts = np.zeros((500,500))

for i in range(500):
    for j in range(500):
        if len(Heights[i][j]) > 0:
            Mean[i,j] = np.mean(Heights[i][j])
            

for i in range(500):
    for j in range(500):
        if len(Heights[i][j]) > 0:
            Counts[i,j] = len(Heights[i][j])
```

To plot the heatmaps, I am using [gmaps](https://github.com/pbugnion/gmaps), a plugin for including Google maps in the IPython Notebook.
```python
import gmaps
import gmaps.datasets
gmaps.configure(api_key="AI...") # Your Google API key
```

Pandas dataframe can't take two-dimensional arrays per column. So we need to put the values of latitudes, longitudes, means and counts in 1d arrays.
```python
longitude_values = [Lon,]*500
latitude_values = np.repeat(Lat,500)
Counts2015=np.array(Counts)
Counts2015.resize((250000,))
Mean2015=np.array(Mean)
Mean2015.resize((250000,))
```

Plotting heatmap for airplane counts
```python
k = {'Counts': Counts2015, 'latitude': latitude_values, 'longitude': np.concatenate(longitude_values)}
df = pd.DataFrame(data=k)
```

```python
locations = df[['latitude', 'longitude']]
weights = df['Counts']
fig = gmaps.figure()
heatmap_layer = gmaps.heatmap_layer(locations, weights=weights)
fig.add_layer(gmaps.heatmap_layer(locations, weights=weights))
fig
```

<img src="https://github.com/FlorisWu/flight-tracks-logan-airport/blob/master/Counts.png?raw=true" width="900"/>

To plot the heatmap for heights, I divided the values of heights into four groups.
```python
low = np.zeros(250000)
low_med = np.zeros(250000)
high_med = np.zeros(250000)
high = np.zeros(250000)

for a in range (0,250000):
    if 0 < Mean2015[a] < 1000:
        low[a] = Mean2015[a]
    elif 1000 <= Mean2015[a] < 2000:
        low_med[a] = Mean2015[a]
    elif 2000 <= Mean2015[a] < 3000:
        high_med[a] = Mean2015[a]
    elif 3000 <= Mean2015[a] < 32500:
        high[a] = Mean2015[a]
        
j = {'x': low, 'latitude': latitude_values, 'longitude': np.concatenate(longitude_values)} 
df = pd.DataFrame(data=j)

k = {'y': low_med, 'latitude': latitude_values, 'longitude': np.concatenate(longitude_values)} 
dg = pd.DataFrame(data=k)

l = {'z': high_med, 'latitude': latitude_values, 'longitude': np.concatenate(longitude_values)} 
dh = pd.DataFrame(data=l)

m = {'n': high, 'latitude': latitude_values, 'longitude': np.concatenate(longitude_values)} 
di = pd.DataFrame(data=m)

locations_low = df[['latitude', 'longitude']]
weights_low = df['x']
fig = gmaps.figure()
heatmap_layer_low = gmaps.heatmap_layer(locations_low, weights_low)
heatmap_layer_low.gradient = [
    (200, 200, 200, 0),
    #(200, 200, 200, 0),
    (255, 0, 0, 1) #color red
]


locations_low_med = dg[['latitude', 'longitude']]
weights_low_med = dg['y']
fig = gmaps.figure()
heatmap_layer_low_med = gmaps.heatmap_layer(locations_low_med, weights_low_med)
heatmap_layer_low_med.gradient = [
    (200, 200, 200, 0),
    #(200, 200, 200, 0), 
    (255, 160, 122, 1) #color orange
]


locations_high_med = dh[['latitude', 'longitude']]
weights_high_med = dh['z']
fig = gmaps.figure()
heatmap_layer_high_med = gmaps.heatmap_layer(locations_high_med, weights_high_med)
fig.add_layer(heatmap_layer_high_med)
heatmap_layer_high_med.gradient = [
    (200, 200, 200, 0),
    #(200, 200, 200, 0),
    (255, 255, 0, 1) #color yellow
]


locations_high = di[['latitude', 'longitude']]
weights_high = di['n']
fig = gmaps.figure()
heatmap_layer_high = gmaps.heatmap_layer(locations_high, weights_high)
heatmap_layer_high.gradient = [
    (200, 200, 200, 0),
    #(200, 200, 200, 0), 
    (0, 255, 127, 1) #color green
]

fig.add_layer(heatmap_layer_high)
fig.add_layer(heatmap_layer_high_med)
fig.add_layer(heatmap_layer_low_med)
fig.add_layer(heatmap_layer_low)


fig
```

<img src="https://github.com/FlorisWu/flight-tracks-logan-airport/blob/master/Heights.png?raw=true" width="900"/>
