# Python_API_Challenge
VacationPy
Starter Code to Import Libraries and Load the Weather and Coordinates Data
# Dependencies and Setup
import hvplot.pandas
import pandas as pd
import requests

# Import API key
from api_keys import geoapify_key
# Load the CSV file created in Part 1 into a Pandas DataFrame
city_data_df = pd.read_csv("output_data/cities.csv")

# Display sample data
city_data_df.head()
City_ID	City	Lat	Lng	Max Temp	Humidity	Cloudiness	Wind Speed	Country	Date
0	0	faya	18.3851	42.4509	22.06	35	21	2.60	SA	1666108228
1	1	farsund	58.0948	6.8047	13.30	100	0	7.65	NO	1666108228
2	2	new norfolk	-42.7826	147.0587	11.72	58	12	1.34	AU	1666108230
3	3	jamestown	42.0970	-79.2353	5.77	77	100	9.77	US	1666107934
4	4	lanzhou	36.0564	103.7922	14.53	48	59	1.20	CN	1666108230
Step 1: Create a map that displays a point for every city in the city_data_df DataFrame. The size of the point should be the humidity in each city.
%%capture --no-display

# Configure the map plot
map_plot = city_data_df.hvplot.points(
    "Lng",
    "Lat",
    geo = True,
    tiles = "EsriImagery",
    size = "Humidity",
    frame_width = 800,
    frame_height = 600, 
    scale = 0.5,
    color = "City"
)

# Display the map
map_plot
Step 2: Narrow down the city_data_df DataFrame to find your ideal weather condition
# Narrow down cities that fit criteria and drop any results with null values
filtered_city_data_df = city_data_df[
    (city_data_df["Max Temp"] < 27) & 
    (city_data_df["Max Temp"] > 21) & 
    (city_data_df["Wind Speed"] < 5) & 
    (city_data_df["Cloudiness"] == 0)
]

# Drop any rows with null values
filtered_city_data_df.dropna()

# Display sample data
filtered_city_data_df
City_ID	City	Lat	Lng	Max Temp	Humidity	Cloudiness	Wind Speed	Country	Date
45	45	kapaa	22.0752	-159.3190	22.99	84	0	3.60	US	1666108257
51	51	hilo	19.7297	-155.0900	26.27	83	0	2.57	US	1666108260
63	63	banda	25.4833	80.3333	24.62	52	0	2.68	IN	1666108268
81	81	makakilo city	21.3469	-158.0858	21.66	81	0	2.57	US	1666108282
146	146	puerto del rosario	28.5004	-13.8627	24.86	73	0	4.63	ES	1666108035
152	152	kahului	20.8947	-156.4700	23.80	60	0	3.09	US	1666108246
197	197	gat	31.6100	34.7642	24.38	100	0	3.69	IL	1666108356
211	211	laguna	38.4210	-121.4238	21.67	79	0	2.06	US	1666108364
240	240	tikaitnagar	26.9500	81.5833	23.56	59	0	0.35	IN	1666108378
265	265	san quintin	30.4833	-115.9500	21.20	74	0	1.37	MX	1666108394
340	340	santa rosalia	27.3167	-112.2833	24.62	56	0	0.74	MX	1666108436
363	363	narwar	25.6500	77.9000	22.35	55	0	1.29	IN	1666108449
375	375	port hedland	-20.3167	118.5667	21.03	73	0	3.09	AU	1666108455
381	381	roebourne	-20.7833	117.1333	23.48	65	0	2.95	AU	1666108458
391	391	saint-francois	46.4154	3.9054	23.69	57	0	4.12	FR	1666108465
409	409	capoterra	39.1763	8.9718	24.84	71	0	3.60	IT	1666108477
421	421	stolac	43.0844	17.9575	24.88	68	0	0.80	BA	1666108483
516	516	guerrero negro	27.9769	-114.0611	23.17	68	0	0.89	MX	1666108537
Step 3: Create a new DataFrame called hotel_df.
# Use the Pandas copy function to create DataFrame called hotel_df to store the city, country, coordinates, and humidity
hotel_df = filtered_city_data_df[["City", "Country", "Lat", "Lng", "Humidity"]].copy()

# Add an empty column, "Hotel Name," to the DataFrame so you can store the hotel found using the Geoapify API
hotel_df["Hotel Name"] = ""

# Display sample data
hotel_df
City	Country	Lat	Lng	Humidity	Hotel Name
45	kapaa	US	22.0752	-159.3190	84	
51	hilo	US	19.7297	-155.0900	83	
63	banda	IN	25.4833	80.3333	52	
81	makakilo city	US	21.3469	-158.0858	81	
146	puerto del rosario	ES	28.5004	-13.8627	73	
152	kahului	US	20.8947	-156.4700	60	
197	gat	IL	31.6100	34.7642	100	
211	laguna	US	38.4210	-121.4238	79	
240	tikaitnagar	IN	26.9500	81.5833	59	
265	san quintin	MX	30.4833	-115.9500	74	
340	santa rosalia	MX	27.3167	-112.2833	56	
363	narwar	IN	25.6500	77.9000	55	
375	port hedland	AU	-20.3167	118.5667	73	
381	roebourne	AU	-20.7833	117.1333	65	
391	saint-francois	FR	46.4154	3.9054	57	
409	capoterra	IT	39.1763	8.9718	71	
421	stolac	BA	43.0844	17.9575	68	
516	guerrero negro	MX	27.9769	-114.0611	68	
Step 4: For each city, use the Geoapify API to find the first hotel located within 10,000 metres of your coordinates.
# Set parameters to search for a hotel
radius = 10000 
params = {"categories": "accommodation.hotel",
          "apiKey":geoapify_key
         }

# Print a message to follow up the hotel search
print("Starting hotel search")

# Iterate through the hotel_df DataFrame
for index, row in hotel_df.iterrows():
    # get latitude, longitude from the DataFrame
    longitude = row["Lng"]
    latitude = row["Lat"]

    # Add filter and bias parameters with the current city's latitude and longitude to the params dictionary
    params["filter"] =  f"circle:{longitude},{latitude},{radius}"
    params["bias"] = f"proximity:{longitude},{latitude}"

    # Set base URL
    base_url = "https://api.geoapify.com/v2/places"

    # Make and API request using the params dictionaty
    name_address = requests.get(base_url, params = params)
    
    # Convert the API response to JSON format
    name_address = name_address.json()
    
    # Grab the first hotel from the results and store the name in the hotel_df DataFrame
    try:
        hotel_df.loc[index, "Hotel Name"] = name_address["features"][0]["properties"]["name"]
    except (KeyError, IndexError):
        # If no hotel is found, set the hotel name as "No hotel found".
        hotel_df.loc[index, "Hotel Name"] = "No hotel found"
        
    # Log the search results
    print(f"{hotel_df.loc[index, 'City']} - nearest hotel: {hotel_df.loc[index, 'Hotel Name']}")
    
# Display sample data
hotel_df
Starting hotel search
kapaa - nearest hotel: Pono Kai Resort
hilo - nearest hotel: Dolphin Bay Hotel
banda - nearest hotel: #acnindiafy21
makakilo city - nearest hotel: Embassy Suites by Hilton Oahu Kapolei
puerto del rosario - nearest hotel: Hotel Tamasite
kahului - nearest hotel: Maui Seaside Hotel
gat - nearest hotel: No hotel found
laguna - nearest hotel: Holiday Inn Express & Suites
tikaitnagar - nearest hotel: No hotel found
san quintin - nearest hotel: Jardines Hotel
santa rosalia - nearest hotel: Sol y Mar
narwar - nearest hotel: No hotel found
port hedland - nearest hotel: The Esplanade Hotel
roebourne - nearest hotel: No hotel found
saint-francois - nearest hotel: Chez Lily
capoterra - nearest hotel: Rosa Hotel
stolac - nearest hotel: Bregava
guerrero negro - nearest hotel: Plaza sal paraiso
City	Country	Lat	Lng	Humidity	Hotel Name
45	kapaa	US	22.0752	-159.3190	84	Pono Kai Resort
51	hilo	US	19.7297	-155.0900	83	Dolphin Bay Hotel
63	banda	IN	25.4833	80.3333	52	#acnindiafy21
81	makakilo city	US	21.3469	-158.0858	81	Embassy Suites by Hilton Oahu Kapolei
146	puerto del rosario	ES	28.5004	-13.8627	73	Hotel Tamasite
152	kahului	US	20.8947	-156.4700	60	Maui Seaside Hotel
197	gat	IL	31.6100	34.7642	100	No hotel found
211	laguna	US	38.4210	-121.4238	79	Holiday Inn Express & Suites
240	tikaitnagar	IN	26.9500	81.5833	59	No hotel found
265	san quintin	MX	30.4833	-115.9500	74	Jardines Hotel
340	santa rosalia	MX	27.3167	-112.2833	56	Sol y Mar
363	narwar	IN	25.6500	77.9000	55	No hotel found
375	port hedland	AU	-20.3167	118.5667	73	The Esplanade Hotel
381	roebourne	AU	-20.7833	117.1333	65	No hotel found
391	saint-francois	FR	46.4154	3.9054	57	Chez Lily
409	capoterra	IT	39.1763	8.9718	71	Rosa Hotel
421	stolac	BA	43.0844	17.9575	68	Bregava
516	guerrero negro	MX	27.9769	-114.0611	68	Plaza sal paraiso
Step 5: Add the hotel name and the country as additional information in the hover message for each city in the map.
%%capture --no-display

# Configure the map plot
hotel_map_plot = hotel_df.hvplot.points(
    "Lng",
    "Lat",
    geo = True,
    tiles = "EsriImagery",
    frame_width = 800,
    frame_height = 600, 
    scale = 0.5,
    color = "City"
)

# Display the map
hotel_map_plot



