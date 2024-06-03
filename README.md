# Sentinal-1
LULC
var ROI = ee.FeatureCollection("projects/ee-irfaqalu /assets/Swabi");
//var ROI = ee.FeatureCollection('projects/ee-kimutailawrence19/assets/ROI');
// Set map center to the aoi for making sure we have the correct study area
Map.centerObject(ROI, 9)
// Define period of analysis
var start = '2022-01-01';
var end = '2022-12-31';
var season = ee.Filter.date(start,end);
print(season);


// Import Sentinel-1 collection
var sentinel1 =  ee.ImageCollection('COPERNICUS/S1_GRD');
// Filter Sentinel-1 collection for study area, date ranges and polarization components
var sCollection =  sentinel1
                    //filter by aoi and time
                    .filterBounds(ROI)
                    .filter(season)
                    // Filter to get images with VV and VH dual polarization
                    .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))
                    .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VH'))
                    // Filter to get images collected in interferometric wide swath mode.
                    .filter(ee.Filter.eq('instrumentMode', 'IW'));
// Also filter based on the orbit: descending or ascending mode
var desc = sCollection.filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING'));
var asc = sCollection.filter(ee.Filter.eq('orbitProperties_pass', 'ASCENDING'));
// Inspect number of tiles returned after the search; we will use the one with more tiles
print("descending tiles ",desc.size());
print("ascending tiles ",asc.size());
// Also Inspect one file
print(asc.first());

// Create a composite from means at different polarizations and look angles.
var composite = ee.Image.cat([
  asc.select('VH').mean(),
  asc.select('VV').mean(),
  desc.select('VH').mean()
]).focal_median();
// Display as a composite of polarization and backscattering characteristics.
Map.addLayer(composite.clip(ROI), {min: [-25, -20, -25], max: [0, 10, 0]}, 'composite');
