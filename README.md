# ndvi-wildfires
Downloads MODIS NDVI data using Google Earth Engine

Collect NDVI data for every US wildfire in 2022 using Python and Google Earth Engine
What is NDVI?
NDVI stands for Normalized Difference Vegetation Index and is a metric for analyzing the amount of green-ness that exists in an area. It can be calculated from bands of light available in satellite images. The color green is visible from Near Infra-Red (NIR) bands and this is used to create a ratio of vegetation versus Red (R) light that is described in the below calculation:
NDVI = (NIR - R) / (NIR + R)
If NIR (green) is higher than R (red) then NDVI is positive and indicates the presence of foliage. If NIR is negative, it indicates water. NDVI values close zero are barren rock and urban areas. 
Photo by Vlad Hilitanu on UnsplashFire Data: WFIGS 2022 Wildfire data and Geometry
WFIGS stands for Wildland Fire Interagency Geospatial Services and provides geospatial data products under the interagency Wildland Fire Data Program. Their 2022 data was originally available at https://www.nwcg.gov/publications/pms936/nifs/public-distribution but the link appears to be broken now. This data includes rows for each fire, a geometry column for each record, the dates of the fire, and how the data was collected. We are mainly interested in the geometry column so that we can download NDVI data for each geometry polygon.
Satellite image provider: MODIS
Satellite: MODIS Terra Daily NDVI (link). MODIS stands for Moderate Resolution Imaging Spectroradiometer and is a satellite operated by NASA with multiple instruments to collect data beyond visible light.
Terra MODIS and Aqua MODIS are viewing the entire Earth's surface every 1 to 2 days, acquiring data in 36 spectral bands, or groups of wavelengths (see MODIS Technical Specifications). https://modis.gsfc.nasa.gov/about/
Cloud provider: Google Earth Engine 
Google Earth Engine can receive requests using the python package earthengine-api . With a free hobby account you can submit up to 6,000 requests/minute. The data collection for this project will only have 5,081 total observations. We will send a request to Google Earth Engine for NDVI data using the earth engine API and read the results into a GeoPandas dataframe.
Desired goal: 2 years of NDVI data from 2021 to 2023 for each fire record in the WFIGS data file, stored as separate files 
This will allow us to ensure that each file is a complete data set for a single fire. Google Earth Engine has request limits that require our requests be broken down into batches. Each file can be easily read back in to a Pandas Dataframe and combined into a master data set.
Python Libraries used
GeoPandas and Pandas
earthengine-api
timeit, sys, json, glob

Necessary cleanup: NaN, None, and Duplicates
We will use GeoPandas to clean the data set and shape the MULTIPOLYGON geometry column into an acceptable json format for Google Earth Engine to parse. Ultimately, I decided to keep the NaN and None records as their geometries may prove useful for calculating data. There were 20 observations with NULL names and NaNs exist in many columns. For records with duplicate names (poly_IncidentName), I dropped all the duplicates except for the record with the greatest geometry area. These duplicates may have been taken as the fire grew in size and the last record contains the greatest fire area.

Request NDVI data from Google Earth Engine
Now that your data set has been cleaned of duplicates, need to submit batch requests to Google Earth Engine for NDVI data for geometry polygons, 250–500 at a time. There are over 5,000 wildfire polygons in this data file so we have to create download batches to give us some control over which of the records are being submitted for processing, in the case of failure for timeouts or system issues. 
The following code has three functions:
getNDVI(fire_polygon_json)sends the request to Google Earth Engine to ask for NDVI data over the past 2 years for the given polygon.
collectAllNDVI(gdf) takes a json version of the GeoPandas geometry column and passes this to getNDVI() to retrieve the satellite data for this column. 
Write the NDVI data to a file named for its record's OBJECTID.

Read the observation files back into a single Pandas dataframe
In this step we will combine the 5,018 CSV files into a single Pandas dataframe. Since we downloaded the files in batches, we will also check to make sure that every OBJECTID in the gdf dataframe is also in the new combined dataframe. Any missing OBJECTIDs will be re-submitted to Google Earth Engine to download NDVI. In my first run, I was missing 3 OBJECTIDs. After downloading the missing files, I recombine all the CSV files into a Pandas dataframe.

Analyze the NDVI data to identify fires with greatest loss of vegetation
This analysis will very simply just find the largest decrease in vegetation between 2021 and 2022 for the months of June, July, and August.

The top 5 wildfires resulting in greatest vegetative loss from 2021 to 2022 are:
Thanks for reading this far! There will be more geospatial content to come.
