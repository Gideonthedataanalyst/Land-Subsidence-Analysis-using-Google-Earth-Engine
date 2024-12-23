# Land-Subsidence-Analysis-using-Google-Earth-Engine

![Screenshot (44)](https://github.com/user-attachments/assets/71c1b1e8-4b5f-4408-97d2-becc5273b43a)


1. Defining a Point of Interest
javascript
Copy code
var subsidencePoint = [36.8219, -1.2921]; // Coordinates for Nairobi
var pointGeometry = ee.Geometry.Point(subsidencePoint); // Create a point geometry
A single point is defined using latitude and longitude coordinates for Nairobi.
This point represents the area of interest (AOI) and helps filter relevant spatial datasets.
2. Loading the Global Subsidence Probability Dataset
javascript
Copy code
var subsidenceProbability = ee.Image("projects/sat-io/open-datasets/global_subsidence/Final_subsidence_proba_greater_1cm_2013_2019_recoded");
A global dataset containing probabilities of land subsidence (2013–2019) is loaded.
The dataset shows areas where land has subsided by more than 1 cm.
3. Adding the Probability Dataset to the Map
javascript
Copy code
Map.addLayer(subsidenceProbability, 
  {palette: ['white', 'yellow', 'orange', 'red', 'brown', 'black']}, 
  'Global Subsidence Probability', 
  false);
The dataset is added to the map with a color palette indicating probability levels (e.g., white = low, black = high).
It is initially turned off (false) to avoid cluttering the map.
4. Loading and Filtering Basin Boundaries
javascript
Copy code
var Hydrobasin = ee.FeatureCollection("WWF/HydroSHEDS/v1/Basins/hybas_5");
var basinBoundaries = Hydrobasin.filterBounds(pointGeometry)
  .map(function(basin) {
    return basin.simplify(1000); // Simplify geometry to reduce complexity
  });
Basin boundaries from the HydroSHEDS dataset are loaded as a FeatureCollection.
Only the basins that intersect the defined point (Nairobi) are selected and simplified to reduce computational complexity.
5. Adding Basin Boundaries to the Map
javascript
Copy code
Map.centerObject(basinBoundaries);
Map.addLayer(basinBoundaries, {}, 'Basin Boundaries');
The map is centered on the filtered basin boundaries.
The selected basin boundaries are overlaid on the map.
6. Clipping the Subsidence Probability Dataset
javascript
Copy code
var clippedSubsidenceProbability = subsidenceProbability.clip(basinBoundaries);
Map.addLayer(clippedSubsidenceProbability, 
  {palette: ['white', 'yellow', 'orange', 'red', 'brown', 'black']}, 
  'Clipped Subsidence Probability', 
  false);
The global subsidence probability dataset is clipped to the basin boundaries to focus on the area of interest.
The clipped dataset is visualized with the same color palette.
7. Loading and Visualizing Subsidence Classification
javascript
Copy code
var subsidenceClassification = ee.Image("projects/sat-io/open-datasets/global_subsidence/Final_subsidence_prediction_recoded");

Map.addLayer(subsidenceClassification.updateMask(subsidenceClassification.neq(1)), 
  {palette: ['white', 'yellow', 'orange', 'red', 'brown', 'black']}, 
  'Global Subsidence Classification', 
  false);
A classification dataset is loaded, showing the types of subsidence in different regions.
Areas with a value of 1 (no subsidence) are masked out for better visualization.
8. Clipping the Classification Dataset
javascript
Copy code
var clippedSubsidenceClassification = subsidenceClassification.clip(basinBoundaries);
Map.addLayer(clippedSubsidenceClassification, 
  {palette: ['white', 'yellow', 'orange', 'red', 'brown', 'black']}, 
  'Clipped Subsidence Classification', 
  false);
Similar to the probability dataset, the classification data is clipped to the basin boundaries for Nairobi.
The clipped classification data is displayed with the same color palette.
9. Exporting the Clipped Dataset
javascript
Copy code
Export.image.toDrive({
  image: clippedSubsidenceClassification,
  description: 'Land_Subsidence_Classification_Nairobi',
  scale: 100,
  region: basinBoundaries.geometry(),
  maxPixels: 1e13,
  crs: 'EPSG:4326',
  folder: 'Land_Subsidence_Analysis'
});
The clipped classification dataset is exported to Google Drive.
Export parameters:
image: The data to export (clipped classification dataset).
description: File name for the export.
scale: Pixel resolution in meters (100 meters per pixel).
region: The geometry of the selected basin boundaries (export area).
maxPixels: Maximum number of pixels allowed for export.
crs: Coordinate Reference System (WGS 84 EPSG:4326).
folder: Google Drive folder where the file will be saved.
