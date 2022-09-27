# Weather Feature Analysis

James Fairgrieve

 - Linkedin: [https://www.linkedin.com/in/jfairgrieve/](https://www.linkedin.com/in/jfairgrieve/)
 - Portfolio: [https://j-fairgrieve.github.io/](https://j-fairgrieve.github.io/)

## Contents
- About the Project
- Resources
- OpenWeatherMap API
- Analysing Extracted Data
- Google Maps API

## About the Project

Please note, the data for this project was collated on 25/09/2022. There are three sections in total for this project:

 - Using the OpenWeatherMap API: In this section, a list of city names are curated using a randomly generated set of 1,500 co-ordinates. These cities are subsequently used for the API calls, generating the weather information for analysis throughout the rest of the project. Overall, the API retreived the weather information for 560 cities.
 - Analysing the Extracted Data: A quick analysis was taken using the weather features collated (Max Temperature, Humidity, Wind Speed and Cloudiness). Any outliers are identified and highlighted, but not removed in this case. The features were then compared against the Latitude of each city co-ordinate to identify any trend in the weather as you approach the equator.
 - Using Google Maps API: Finally, the Google Maps API was used to plot the humidity changes across the globe. After defining the ideal weather conditions for a holiday, map markers were then generated to show the nearest hotel in relation to the city co-ordinates.

## Resources
 - [OpenWeatherMap API](https://openweathermap.org/api)
 - [Google Maps API](https://developers.google.com/maps)

## OpenWeatherMap API
###### You can find the full code for this section [here](https://github.com/J-Fairgrieve/Weather-API/blob/main/1%20-%20OpenWeatherMap%20API.ipynb).

#### 1.1 Generating a List of Cities
Firstly, the latitude and longitude must be defined. Latitude ranges between **-90 and 90**, whereas Longitude ranges between **-180 and 180**.

~~~
# Create a range of latitudes and longitudes
lat_range = (-90, 90)
lng_range = (-180, 180)

# Create two empty lists for holding the latitude/longitude and city names
lats_lngs = []
city_names = []
~~~

Next, a set of random co-ordinates must be created. For this example a total of **1,500** co-ordinates were sampled.

~~~
# Create a set of random co-ordinates to fill the lat_lngs list
lats = np.random.uniform(lat_range[0], lat_range[1], size = 1500)
lngs = np.random.uniform(lng_range[0], lng_range[1], size = 1500)
lats_lngs = zip(lats, lngs)
~~~

With the co-ordinates created and put into a list, we can now use the Citipy library to generate the nearest city for each location. The name of each city found will be added to the city_names list. In order to prevent duplicates, a check will be implemented into the function before adding any of these city names.

~~~
# Identify the nearest city for each generated co-ordinate
for lat_lng in lats_lngs:
    city = citipy.nearest_city(lat_lng[0], lat_lng[1]).city_name
    
    # If the located city is not in the current list, add to the city_names list
    if city not in city_names:
        city_names.append(city)
~~~

From the **1,500** co-ordinates, a total of **612** city names were extracted (**40.8%** retrieval rate).

#### 1.2 Performing the API Calls
In order to run the code in this section, you will need to create a variable for the url (using your API key) and an empty list to hold the city data. Once these variables are created, the loop for scraping the weather information is as follows:

~~~
# Create a loop for scraping data from the API
for x, city in enumerate(city_names):
    city_url = url + "&q=" + city
    
    # Run an API request for each city in the cities list
    try:
        city_weather = requests.get(city_url).json()
        
        city_lat = city_weather["coord"]["lat"]
        city_lng = city_weather["coord"]["lon"]
        city_max_temp = city_weather["main"]["temp_max"]
        city_humidity = city_weather["main"]["humidity"]
        city_clouds = city_weather["clouds"]["all"]
        city_wind = city_weather["wind"]["speed"]
        city_country = city_weather["sys"]["country"]
        city_date = city_weather["dt"]
        
        # Append the city information into the city_data list
        city_data.append({"City": city,
                          "Lat": city_lat,
                          "Lng": city_lng,
                          "Max Temp": city_max_temp,
                          "Humidity": city_humidity,
                          "Cloudiness": city_clouds,
                          "Wind Speed": city_wind,
                          "Country": city_country,
                          "Date": city_date})
        record_counter = record_counter + 1
    except:
        record_counter = record_counter + 1
        pass
~~~

In this example, the API was able to retreive the weather information for **560** cities in total, a **91.5%** success rate.

## Analysing Extracted Data
###### You can find the full code for this section [here](https://github.com/J-Fairgrieve/Weather-API/blob/main/2%20-%20Analysing%20Extracted%20Data.ipynb).

In this section, we delve into the four main weather features extracted from the API in section 1:
 - Max Temperature
 - Humidity
 - Cloudiness
 - Wind Speed
 
#### 2.1 Identifying Outliers
Are there any cities with weather features significantly higher or lower than the rest of the sample? The easiest way to identify this visually is by creating box plots.

![Weather Feature Box Plots](https://github.com/J-Fairgrieve/Weather-API/blob/main/Images/Weather%20Feature%20Boxplot.png?raw=true)

The boxplots instantly identify the outliers present in both the Humidity and Wind Speed features. From these graphs we can also see that Cloudiness has the highest variance and Wind Speed appears to be skewed towards slower speeds.

After further coding, the outliers in Humidity are the following cities:

![Humidity Outliers](https://github.com/J-Fairgrieve/Weather-API/blob/main/Images/Humidity%20Outliers.png?raw=true)

The IQR for Humidity is **40.5**, meaning that any Humidity below **17.5** is considered an outlier. **12** cities meet this criteria.

In similar fashion, the outliers for Wind Speed are as follows:

![Wind Speed Outliers](https://github.com/J-Fairgrieve/Weather-API/blob/main/Images/Wind%20Speed%20Outliers.png?raw=true)

The IQR for Wind Speed is **12.1**, meaning that any Wind Speed above **19** is considered an outlier. **11** cities meet this criteria.

Ordinarily, outliers would be removed at this point but in order to keep the criteria for the ideal holiday fully customisable in section 3 I have chosen to keep the outliers.

#### 2.2 Weather Features Across Latitude
Note, visualisations were produced to compare each weather feature against Latitude, but only the Max Temperature appeared to have a correlation. The other charts can be viewed in the code file.
##### 2.2.1 Latitude vs. Temperature
![Latitude vs. Temperature](https://github.com/J-Fairgrieve/Weather-API/blob/main/Images/Latitude%20vs%20Max%20Temp.png?raw=true)

A clear curve can be seen. The highest observed temperatures can be seen where the Latitude is close to zero (the equator). The further north or south you travel, the lower the max temperature will be.

#### 2.3 Linear Regression
Similarly to the previous section, Linear Regression models have been created for every weather feature but only the Max Temperature findings will be discussed. The other charts have very low R Squared values so do not explain a high proportion of the variance. However, the models can be viewed in the code file.

In order to perform the Linear Regression models, the data has been split into Northern/Southern Hemisphere DataFrames. In the sample cities, there are **386** cities in the northern hemisphere and **174** cities in the southern hemisphere.

##### 2.3.1 Northern Hemisphere - Temperature vs. Latitude
![Latitude vs. Temperature - North](https://github.com/J-Fairgrieve/Weather-API/blob/main/Images/Temperature%20Regression%20North.png?raw=true)

The northern hemisphere regression model has an R Squared value of **0.697**, demonstrating a strong correlation between the Latitude and Max Temperature. The variance appears to be higher as the Latitude decreases.

##### 2.3.2 Southern Hemisphere - Temperature vs. Latitude
![Latitude vs. Temperature - North](https://github.com/J-Fairgrieve/Weather-API/blob/main/Images/Temperature%20Regression%20South.png?raw=true)

At first sight, the southern hemisphere model definitely displays a higher variance than the northern model. This is demonstrated further with the lower R squared value of **0.550**. This could be due to the smaller sample size of southern cities in this dataset.

## Google Maps API
###### You can find the full code for this section [here](https://github.com/J-Fairgrieve/Weather-API/blob/main/3%20-%20Google%20Maps%20API.ipynb).

#### 3.1 Humidity Heatmap
Firstly, the API must be called and the Humidity data defined in order to plot the layer onto a map. Once these variables have been defined, the map figure can be created and the Humidity layer can be applied:

~~~
# Configure gmaps and call location and humidity data to plot on the map
gmaps.configure(api_key = google_key)
coords = city_df[["Lat", "Lng"]]
humidity = city_df["Humidity"]
max_humidity = humidity.max()

# Create the map
fig = gmaps.figure()

# Create a new layer for the heatmap
heat_layer = gmaps.heatmap_layer(coords,
                                 weights = humidity,
                                 max_intensity = max_humidity,
                                 point_radius = 6)

# Add the newly created heatmap layer and display the final map
fig.add_layer(heat_layer)
~~~

The map appears as follows:

![Humidity Heatmap](https://github.com/J-Fairgrieve/Weather-API/blob/main/Images/Google%20Maps%20Humidity%20Heatmap.png?raw=true)

#### 3.2 Finding Hotels Using Weather Criteria
For this example, the ideal location is defined as a temperature above 70F, below 80F and observes a Wind Speed below 10mph.

~~~
# Create a new DataFrame for the above criteria
filtered_cities = city_df.loc[(city_df["Max Temp"] > 70)
                              & (city_df["Max Temp"] < 80)
                              & (city_df["Wind Speed"] < 10)]

# Shuffle the new cities so a random sample can be taken later on
filtered_cities = filtered_cities.sample(frac = 1)
filtered_cities.reset_index(inplace = True)
filtered_cities
~~~

There are 105 hotels that match this criteria which would result in too many markers if all placed on a map. For this example, no more than 10 markers will be placed on the map. This will need to be outlined in the loop gathering the hotel data.

~~~
# Create a copy of the filterest cities DataFrame
hotels = filtered_cities

# Track the number of hotels logged so no more than 10 are in the final map
hotelcount = 0

# Set the parameters for a google search
params = {
    "radius": 5000,
    "types": "lodging",
    "key": google_key}

# Create a loop to find the hotel information
for index, row in hotels.iterrows():
    # End loop if hotel quota is met
    if hotelcount == 10:
        break
    
    # Create the location information
    lat = row["Lat"]
    lng = row["Lng"]
    params["location"] = f"{lat},{lng}"
    base_url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json"
    
    # Make a search through the API
    response = requests.get(base_url, params = params).json()
    results = response["results"]
    
    # Add the hotel information
    try:
        hotels.loc[index, "Hotel Name"] = results[0]["name"]
        hotelcount = hotelcount + 1
    
    # Skip if no hotel found
    except (keyError, IndexError):
        hotels.loc[index, "Hotel Name"] = None
~~~

Once placing the new layer onto the map, the final map displays as follows:

![Complete Map with Markers](https://github.com/J-Fairgrieve/Weather-API/blob/main/Images/Google%20Maps%20Marker%20Heatmap.png?raw=true)