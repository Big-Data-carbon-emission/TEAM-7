# Install Earth Engine and geemap
!pip install earthengine-api geemap
import ee  # Google Earth Engine API
import geemap  # For interactive maps
import matplotlib.pyplot as plt

# Define Region of Interest (ROI) - Example: San Francisco Bay Area
roi = ee.Geometry.Rectangle([-123.1, 37.2, -121.5, 38.5])

# Define Date Range for Analysis
start_date = '2023-01-01'
end_date = '2023-01-31'

# Load Sentinel-5P TROPOMI Nitrogen Dioxide (NO2) Data (Alternative to CO2)
collection = ee.ImageCollection("COPERNICUS/S5P/OFFL/L3_NO2") \
    .filterDate(start_date, end_date) \
    .filterBounds(roi)

# Compute Mean NO2 Concentration Over the Date Range
mean_no2 = collection.mean().select('NO2_column_number_density')

# Visualization Parameters
viz_params = {
    'min': 0,  # Minimum NO2 value (mol/m²)
    'max': 0.0003,  # Maximum NO2 value
    'palette': ['blue', 'green', 'yellow', 'orange', 'red']  # Color range
}

# Create an Interactive Map
Map = geemap.Map(center=[37.8, -122.5], zoom=6)
Map.addLayer(mean_no2, viz_params, "Average NO2 (Jan 2023)")
Map.addLayer(roi, {'color': 'black'}, 'Region of Interest', False)
Map.add_colorbar(viz_params, label="NO2 Levels (mol/m²)", orientation="horizontal")
Map
# Install Google Earth Engine and geemap
!pip install earthengine-api geemap --quiet

# Import Libraries
import ee  # Google Earth Engine
import geemap  # Interactive Map Visualization
# Authenticate and initialize Earth Engine
ee.Authenticate()
ee.Initialize()
print("Google Earth Engine initialized successfully!")
# Define the Region of Interest (ROI)
roi = ee.Geometry.Rectangle([77.4, 12.8, 77.8, 13.2])  # Bangalore,India

# Define the Date Range for One Year
start_date = '2022-01-01'
end_date = '2022-12-31'

# Function to compute monthly averages
def compute_monthly_mean(month):
    start = ee.Date(f'2022-{month:02d}-01')  # Start of the month
    end = start.advance(1, 'month')  # End of the month
    no2_month = ee.ImageCollection("COPERNICUS/S5P/OFFL/L3_NO2") \
        .filterDate(start, end) \
        .filterBounds(roi) \
        .mean() \
        .select('NO2_column_number_density')
    ch4_month = ee.ImageCollection("COPERNICUS/S5P/OFFL/L3_CH4") \
        .filterDate(start, end) \
        .filterBounds(roi) \
        .mean() \
        .select('CH4_column_volume_mixing_ratio_dry_air')
    co_month = ee.ImageCollection("COPERNICUS/S5P/OFFL/L3_CO") \
        .filterDate(start, end) \
        .filterBounds(roi) \
        .mean() \
        .select('CO_column_number_density')
    so2_month = ee.ImageCollection("COPERNICUS/S5P/OFFL/L3_SO2") \
        .filterDate(start, end) \
        .filterBounds(roi) \
        .mean() \
        .select('SO2_column_number_density')
    return {'no2': no2_month, 'ch4': ch4_month, 'co': co_month, 'so2': so2_month}

# Loop through each month and calculate the mean
monthly_data = [compute_monthly_mean(month) for month in range(1, 13)]

# Combine monthly averages into yearly averages
yearly_no2 = ee.ImageCollection([data['no2'] for data in monthly_data]).mean()
yearly_ch4 = ee.ImageCollection([data['ch4'] for data in monthly_data]).mean()
yearly_co = ee.ImageCollection([data['co'] for data in monthly_data]).mean()
yearly_so2 = ee.ImageCollection([data['so2'] for data in monthly_data]).mean()

# Visualization Parameters
viz_params_no2 = {'min': 0, 'max': 0.0003, 'palette': ['blue', 'green', 'yellow', 'orange', 'red']}
viz_params_ch4 = {'min': 1800, 'max': 2000, 'palette': ['blue', 'green', 'yellow', 'orange', 'red']}
viz_params_co = {'min': 0.03, 'max': 0.05, 'palette': ['blue', 'green', 'yellow', 'orange', 'red']}
viz_params_so2 = {'min': 0, 'max': 0.0005, 'palette': ['blue', 'green', 'yellow', 'orange', 'red']}

# Create Interactive Map
Map = geemap.Map(center=[37.8, -122.5], zoom=6)

# Add Yearly Pollutant Layers to the Map
Map.addLayer(yearly_no2, viz_params_no2, "Yearly NO2 (Nitrogen Dioxide)")
Map.addLayer(yearly_ch4, viz_params_ch4, "Yearly CH4 (Methane)")
Map.addLayer(yearly_co, viz_params_co, "Yearly CO (Carbon Monoxide)")
Map.addLayer(yearly_so2, viz_params_so2, "Yearly SO2 (Sulfur Dioxide)")

# Add ROI Outline
Map.addLayer(roi, {'color': 'black'}, 'Region of Interest', False)

# Add Colorbars for Each Pollutant
Map.add_colorbar(viz_params_no2, label="NO2 Levels (mol/m²)")
Map.add_colorbar(viz_params_ch4, label="CH4 Levels (ppm)")
Map.add_colorbar(viz_params_co, label="CO Levels (mol/m²)")
Map.add_colorbar(viz_params_so2, label="SO2 Levels (mol/m²)")

# Display the Map
Map
import ee
import matplotlib.pyplot as plt

# Authenticate and initialize Earth Engine
ee.Authenticate()
ee.Initialize()

# Define the Region of Interest (ROI) - Bangalore, India
roi = ee.Geometry.Rectangle([77.4, 12.8, 77.8, 13.2])  # Approximate bounding box for Bangalore

# Define the Year for Analysis
year = 2023

# General function to compute monthly averages for any pollutant
def compute_monthly_pollutant(month, collection_name, band_name):
    start = ee.Date(f'{year}-{month:02d}-01')  # Start of the month
    end = start.advance(1, 'month')  # End of the month
    monthly_data = ee.ImageCollection(collection_name) \
        .filterDate(start, end) \
        .filterBounds(roi) \
        .mean() \
        .select(band_name) \
        .reduceRegion(
            reducer=ee.Reducer.mean(),
            geometry=roi,
            scale=1000
        )
    return ee.Number(monthly_data.get(band_name)).getInfo()

# Function to fetch monthly averages for a pollutant
def fetch_monthly_trends(collection_name, band_name):
    pollutant_values = []
    for month in range(1, 13):  # Loop over 12 months
        try:
            value = compute_monthly_pollutant(month, collection_name, band_name)
            pollutant_values.append(value if value is not None else 0)  # Append 0 if data is missing
        except Exception as e:
            print(f"Error retrieving data for month {month}: {e}")
            pollutant_values.append(0)  # Append 0 for failed data retrieval
    return pollutant_values

# Define collections and bands for each pollutant
pollutants = {
    "NO2": {"collection": "COPERNICUS/S5P/OFFL/L3_NO2", "band": "NO2_column_number_density"},
    "CH4": {"collection": "COPERNICUS/S5P/OFFL/L3_CH4", "band": "CH4_column_volume_mixing_ratio_dry_air"},
    "CO": {"collection": "COPERNICUS/S5P/OFFL/L3_CO", "band": "CO_column_number_density"},
    "SO2": {"collection": "COPERNICUS/S5P/OFFL/L3_SO2", "band": "SO2_column_number_density"}
}

# Fetch data for each pollutant
results = {}
for pollutant, details in pollutants.items():
    print(f"Fetching data for {pollutant}...")
    results[pollutant] = fetch_monthly_trends(details["collection"], details["band"])

# Plot trends for each pollutant
months = list(range(1, 13))  # Months from January to December
colors = {"NO2": "blue", "CH4": "green", "CO": "red", "SO2": "purple"}

plt.figure(figsize=(12, 8))
for pollutant, values in results.items():
    plt.plot(months, values, marker='o', label=f'{pollutant} Levels', color=colors[pollutant])

# Customize the plot
plt.title(f'Monthly Average Levels of Pollutants ({year}) - Bangalore, India')
plt.xlabel('Month')
plt.ylabel('Pollutant Levels')
plt.grid(True)
plt.xticks(months)
plt.legend()
plt.show()
# List all files in the extracted folder
extracted_files = os.listdir(extracted_folder_path)
extracted_files

# Path to the 'Big_Data_Project' subfolder
subfolder_path = os.path.join(extracted_folder_path, 'Big_Data_Project')

# List all files in the 'Big_Data_Project' subfolder
subfolder_files = os.listdir(subfolder_path)
subfolder_files
# List to store paths of CSV files
csv_files = []

# Loop through each subfolder to find CSV files
for sector_folder in subfolder_files:
    sector_folder_path = os.path.join(subfolder_path, sector_folder)

    # List all files in the subfolder
    sector_files = os.listdir(sector_folder_path)

    # Find CSV files in the subfolder
    for file in sector_files:
        if file.endswith('.csv'):  # Check if the file is a CSV
            csv_files.append(os.path.join(sector_folder_path, file))

# Check if CSV files were found
csv_files
import pandas as pd

# List to hold dataframes
data_frames = []

# Loop through each CSV file and load it into a dataframe
for file_path in csv_files:
    df = pd.read_csv(file_path)
    data_frames.append(df)

# Combine all dataframes into one
if data_frames:
    combined_data = pd.concat(data_frames, ignore_index=True)
    print(combined_data.head())  # Display the first few rows to verify the data
else:
    print("No CSV files found in the subfolders.")

