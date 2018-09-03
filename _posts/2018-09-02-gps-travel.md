---
layout: post
title: Tracking your travel adventures using GPS data from your phone and Python/folium
comments: True
---

Having a visual GPS log of your trips to foreign places can be very beneficial for telling your friends and family a cool story about your adventures. I recently started tracking my location and other data such as accelerometer with the iOS app ['SensorLog'](https://itunes.apple.com/us/app/sensorlog/id388014573?mt=8) to visualise it using Python/R scripts. In this blog post I will briefly guide you through setting up the GPS tracking on your phone and how you can create beautiful maps with your data.

<a href="https://loopingleo.github.io/GPS-tracking/kyoto/" rel="folium map">![demo.png](https://raw.githubusercontent.com/loopingleo/blog/master/images/Screenshot%202018-09-03%2001.05.12.png)</a>


GPS data logging with your smartphone
---

[SensorLog](https://itunes.apple.com/us/app/sensorlog/id388014573?mt=8) is a great app for iOS devices to record all your phone's sensor data. You can simply let the app run in the background -- whether your hiking in New Zealand, enjoying Renaissance paintings in Rome or sipping Moscow Mules on a cruise ship.

![SensorLog-1](https://raw.githubusercontent.com/loopingleo/blog/master/images/IMG_9662.JPG) ![SensorLog-2](https://raw.githubusercontent.com/loopingleo/blog/master/images/IMG_9663.JPG)


### Transferring GPS logging data to your computer

Once you recorded your sensor data, you can transfer it from your phone to your computer. Personally, I save these files using iCloud.


### Reading in the data in Python

Reading in your sensor data into Python is easy. My choice for this use case is pandas and locally stored files since it's relatively simple and it works great. You can read single files. But in my case I had been tracking my location for many days and had multiple files ready to be visualised in my iCloud. Let's have a look how this works:

``` python
import os, glob
import pandas as pd

# read multiple files with SensorLog data
# adjust to your directory - I use iCloud directory on my Mac
path = os.path.expanduser("~/Library/Mobile Documents/com~apple~CloudDocs/SensorLogData")  
allFiles = glob.glob(path + "/*.csv")
frame = pd.DataFrame()
list_ = []

# read all files in directory
for file_ in allFiles:
  df = pd.read_csv(file_, index_col=None, header=0)
  list_.append(df)

# concat all files into one DataFrame
df_sensorData = pd.concat(list_, sort=True)
```

Once you've read in the file, you can have a brief look at the data:


``` python
df_sensorData.info()
with pd.option_context('display.max_rows', 1, 'display.max_columns', None):
    print(df_sensorData)
```

Or you could do some simple plotting with matplotlib.pyplot.


Once you have had a first look at the data, you can decide which variables are most important to you. In this example, I selected the following columns from my pandas DataFrame and shortened the column names for easier handling:

``` python
# select relevant columns
df_compressed = df_sensorData[["loggingTime(txt)",
                               "locationTimestamp_since1970(s)",
                               "loggingSample(N)",
                               "locationLatitude(WGS84)",
                               "locationLongitude(WGS84)",
                               "locationAltitude(m)",
                               "locationSpeed(m/s)",
                               "accelerometerTimestamp_sinceReboot(s)",
                               "accelerometerAccelerationX(G)",
                               "accelerometerAccelerationY(G)",
                               "accelerometerAccelerationZ(G)",
                               "gyroRotationX(rad/s)",
                               "gyroRotationY(rad/s)",
                               "gyroRotationZ(rad/s)"]]

# shorten names of columns
df_compressed = df_compressed.rename(index=str, columns={"loggingTime(txt)":"time",
                               "locationTimestamp_since1970(s)":"time_s1970",
                               "loggingSample(N)":"sample_ind",
                               "locationLatitude(WGS84)":"lat",
                               "locationLongitude(WGS84)":"lon",
                               "locationAltitude(m)":"altitude",
                               "locationSpeed(m/s)":"speed",
                               "accelerometerTimestamp_sinceReboot(s)":"accel_time",
                               "accelerometerAccelerationX(G)":"x",
                               "accelerometerAccelerationY(G)":"y",
                               "accelerometerAccelerationZ(G)":"z",
                               "gyroRotationX(rad/s)":"Xgyro",
                               "gyroRotationY(rad/s)":"Ygyro",
                               "gyroRotationZ(rad/s)":"Zgyro"})
```

There you go. Now your data is ready to be put on a map.



Plotting GPS data on a map using folium or gmplot
---

It is time to plot your GPS data on a map. If you have a Google Maps API key or you just want to print the Google maps on your local machine, you could use the following code:

``` python
## ---------- PLOT ON Google MAP  ----------------------------
gmap = gmplot.GoogleMapPlotter(df_compressed.lat.mean(), df_compressed.lon.mean(), 14)

#plot map circles in different sizes (and colors)
for i in range(0,len(df_compressed)-1,20):
    if abs(df_compressed.speed[i]) < 3/3.6:
        gmap.circle(df_compressed['lat'][i], df_compressed['lon'][i],
                    color="#f44242",
                    marker=False,
                    radius= 20 * (df_compressed["speed"][i])**(2),
                    #radius = 10,
                    ew = 0.3,
                    face_alpha=0.8,
                    )

#gmap.scatter(df['Latitude'], df['Longitude'], df["markColScale"][i], size=0.5, marker=False)
#gmap.heatmap(df_compressed['lat'], df_compressed['lon'], threshold=5, radius=40)

gmap.draw("your_map_filename.html")  # saves to html file for display - open file in your browser
```

A nice and free alternative to Google Maps is [OpenStreetMap](https://www.openstreetmap.org/). You can use the Python package [folium](https://github.com/python-visualization/folium) which uses [leaflet.js](https://leafletjs.com) and OpenStreetMap. Folium comes in handy if you want to use the final map and host it in your blog like I did [here](https://loopingleo.github.io/GPS-tracking/kyoto/).

In case you want to go with folium, let us have a look at the code for that:


``` python
## ---------- PLOT ON OSM MAP with folium  ----------------------------

locationlist = df_compressed[["lat","lon"]].dropna().values.tolist()
map = folium.Map(location=[np.mean(df_compressed['lat']), np.mean(df_compressed['lon'])], zoom_start=7)

for point in range(0, len(locationlist), 20):
    if abs(df_compressed.speed[point]) < 3 / 3.6:
        folium.Circle(locationlist[point], radius=1,#/(1+ (df_compressed["speed"][point])**(2)),
                            color='#00d4ff', fill=True, fill_color ='#00d4ff', fill_opacity=0.2,
                            stroke = True, weight = 0.1).add_to(map)

folium.TileLayer('cartodbdark_matter').add_to(map)
#folium.TileLayer('cartodbpositron').add_to(map)
#folium.TileLayer('Stamen Toner').add_to(map)

map

filepath = 'your_map_filename.html'
map.save(filepath)
```

<a href="https://loopingleo.github.io/GPS-tracking/kyoto/" rel="folium map">![demo.png](https://raw.githubusercontent.com/loopingleo/blog/master/images/Screenshot%202018-09-03%2001.22.22.png)</a>

Once you've got the feel for it, you can create maps like this one with larger datasets:

<a href="https://raw.githubusercontent.com/loopingleo/blog/master/images/Screenshot%202018-09-03%2001.25.02.png" rel="folium map">![demo.png](https://raw.githubusercontent.com/loopingleo/blog/master/images/Screenshot%202018-09-03%2001.25.02.png)</a>

You can find the complete code in my [GitHub repo 'GPS-tracking'](https://github.com/loopingleo/GPS-tracking)



### Now it's your turn

Finally, please let me know what kind of interesting plots you were able to create.
Leave a comment below or send me an email. Thank you and happy coding.

Leo
