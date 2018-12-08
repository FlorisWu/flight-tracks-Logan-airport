# The burden of aircraft noise over West Boston

In the summer of 2018, as part of [#ARforChange](http://www.storybench.org/arforchange-project-using-immersive-technologies-data-social-change/), a project exploring new platforms for civic engagement and social change using immersive media at Northeastern University's School of Journalism, I collected flight data and presented them through heatmaps to visualize the unfair burden of aircraft noise over West Boston with Python.

## Data
Data are from [Kent Johnson](http://kentsj.com/BWFS/Logan_33L_Flight_Tracks_2013_vs_2015.html), a member of [Boston West Fair Skies](https://www.bostonwestfairskies.org/news.html), an advocacy group aiming to address this issue.

## Loading packages
```Python
import geopandas as gpd
import numpy as np
```

## Loading and processing data
```Python
Jan_2013 = '...../Jan2013.gpkg' #data from Jan 2013
data_2013 = gpd.read_file(Jan_2013)
```

## Creating heatmaps
```Python
# I created two heatmaps of airplane counts and heights. The first one shows the mean heights of flights over a 100x100 grid over parts Boston area near Logan Airport, while the second one shows the number of counts of flights in each 100x100 cell.
Lon = np.arange(-71.33, -70.7, 0.0063) #this divides the longitude range we want into 100 equal intervals; we do this because we want to take mean values of heights in those 100x100 small cells in the 100x100 grid to plot the heatmap
Lat = np.arange(42.1, 42.6, 0.005) #this divides the latitude range we want into 100 equal intervals

Heights = np.zeros((100,100))
Counts = np.zeros((100,100))

for a in range(0,24747): #there are 24747 data points
    for b in range(0,len(data['geometry'].values[a].coords[:][:])): #this is the length of
        for c1 in range (0,100):
            if Lat[c1] - 0.00315 <= data['geometry'].values[a].coords[b][0] < Lat[c1] + 0.00315: #we are making the 100 latitude and longnitude points in Lat and Lon the center of the small cells here, that's why we have plus and minus half of the change of lat/lon per small cell (the small cells make up the grid that we will need for the heat map)
                for c2 in range (0,100):
                    if Lon[c2] - 0.0025 <= data['geometry'].values[a].coords[b][1] < Lon[c2] + 0.0025:
                        Heights[c1,c2] += data['geometry'].values[a].coords[b][2] # we are putting heights of flight path points (like the points recorded in a linestring) that correspond to the small cell of lat/lon area in that small cell
                        Counts[c1,c2] += 1 #recording counts
                        break
```
