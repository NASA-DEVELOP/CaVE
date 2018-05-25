Project Title: Classification and Verification Editor (CaVE)

Purpose and Description of Innovation/Software

This code is for use within Google Earth Engine API, an open-source software currently in beta. The purpose is to improve the efficiency of analyzing multiple classification methods 
in order to produce the most accuracy classified images for land use and land cover change. Instead of running multiple classifications, validations, and algorithms separately, this is all 
performed through one easy to use script with one condensed output located in the console.

The software allows for the addition of images (in particular Landsat and RapidEye) for classification analysis. Classified RapidEye imagery is used to validate the stated 
classifications for the seasonal years 2014 and 2015. Output maps can all be displayed easily on the interface and each map layer can be toggled on or off for easy comparison.

Documentation and references

Google Earth Engine API: https://earthengine.google.com/
Google Earth Engine online tutorial and guide: https://developers.google.com/earth-engine/getstarted


Code Details

The software includes embedded computer databases as it uses imagery from USGS stored data and with open-source software: Google Earth Engine API.

Link to the GEE Code Editor with the final code: https://code.earthengine.google.com/ec2c1e8a1d868f373ddf8231bd88bc08

Google Earth Engine allows for the use of vector data through kml files and uploaded as a fusion table. These fusion tables contain the training sites that the team has created for 
classification purpose. The results are displayed on a consol. This allows for ease when comparing validation confusion matrices and land changes since it is all located in one area. RapidEye imagery 
was imported into the CaVE script using the 'Assets' tab within Google Earth Engine. When classifying the RapidEye imagery within GEE, they were remapped purposely to avoid confusion during validation
since the field containing the classification numbers need to have a unique name from the Landsat imagery fields.

Example: 

// Remap the classification numbers for the clipped RapidEye image using maximum entropy classification
var ME = mecl.clip(outline)
  .remap([1,2,3,4],[0,1,2,3]); 

//For validation, 'remapped' is referring to the classified RapidEye image, and 'classification' is referring to the Classified Landsat Image 
var testAccuracy = validated.errorMatrix('remapped', 'classification');

Classifications are as follows:

0 - Water
1 - Rural/Non-forest (RNF)
2 - Forest
3 - Urban

As an overall side note, Noel Gorelick from GEE advises against for-loops. Therefore, each year was classified with a separate code for each classification method. 
Makes the code very lengthy and repetitive!

Code Outline with GEE Tutorial Examples

	a. Identify bands for prediction
		var bands = ['B2', 'B3', 'B4', 'B5', 'B6', 'B7', 'B10', 'B11'];
	b. Define variables for each images of areas of interest 
		var image = ee.Image('LANDSAT/LC8_L1T_TOA/LC82320672013207LGN00').select(bands);
	c. Load training polygons from a Fusion Table
		var polygons = ee.FeatureCollection('ft:table_source');
	d. Add an outline of the area of interest through fusion table
		var outline = ee.FeatureCollection('ft:table_source');
	e. Call the rapid eye image from the imports section and classify it for use in later validation 
	f. Clip the rapid eye image to the outline fusion table and remove the null values
	g. Mask out '0' values to remove the cloud class in rapid eye for exclusion from validation
	*For classifying images:
	h. Get the values for all pixels in each polygon in the training.
		var training = image.sampleRegions({
	i. Get the sample from the polygons FeatureCollection.
  	        collection: polygons,
  	j. Keep this list of properties from the polygons.
  	        properties: ['class'],
        k. Set the resolution of the image to the proper scale. Example below shows 30 meter resolution
  	        scale: 30
       	l. Make a classifier for each year and train it.
   		var classifier = ee.Classifier.randomForest(10)
    		   .train(training, 'Land_Cover_Type_1');
        m. Classify the input images.
 	    	var classified = input.classify(classifier);
	*For validation:
    	n. Sample the input with a random seed to get validation data for classified image.
	   	var validation = input.addBands(modis).sample({
  		numPixels: 5000,
  		seed: 1
  	o. Filter the result to get rid of any null pixels.
	  	}).filter(ee.Filter.neq('B1', null));
     	p. Classify the validation data.
	  	var validated = validation.classify(classifier);
    	q. Get a confusion matrix representing expected accuracy.
		var testAccuracy = validated.errorMatrix('Land_Cover_Type_1', 'classification');
		print('Validation error matrix: ', testAccuracy);
		print('Validation overall accuracy: ', testAccuracy.accuracy());
	*For displaying images:
     	r. Clip the classified Landsat images for display to the polygon outline of the area of interest.
     	s. Create a palette to display the classes.
		 	var igbpPalette = [
  				'aec3d4', // water
 				 '152106', '225129', '369b47', '30eb5b', '387242', // forest
 				 '6a2325', 'c3aa69', 'b76031', 'd9903d', '91af40',  // shrub, grass
 				 '111149', // wetlands
 				 'cdb33b', // croplands
  				'cc0013', // urban
  				'33280d', // crop mosaic
  				'd7cdcc', // snow and ice
  				'f7e084', // barren
  				'6f6f6f'  // tundra
			];
	t. Set the center of the map.
		Map.setCenter(-117.3, 33.30, 9);
	u. Add map layers for display
		Map.addLayer(classified, {min: 0, max: 10, palette: palette}, 'Vegetation Type');
	v. Calculate the areas for each class within each year in km^2 using the reducer
		var area = MaxEnt.eq([0, 1, 2, 3]).multiply(ee.Image.pixelArea()).multiply(1e-6);
		var reducer = area.reduceRegion({
  			reducer: ee.Reducer.sum(),
  			scale: 30,
  			geometry: outline
		});
	w. Create a fusion table of the areas for each year/class using Excel and saving as a csv file for use in Google Fusion Tables application. 
	x. Use the fusion table of areas to create a line graph.
		var chart2 = Chart.feature.byFeature(extremeCities, 'city_name')
    			.setChartType('ColumnChart')
    			.setOptions({
      			title: 'Average Monthly Temperatures, by City',
      			hAxis: {title: 'City'},
      			vAxis: {title: 'Temperature (Celsius)'}
		});
	y. Optional: Export images as .tif files
		Export.image(landsat, 'exportImageExample', {
  			scale: 30,
  			region: geometry
		});

  
Once all images are defined in the CaVE script, simple run the software using the 'Run' button at the top of the screen. Area and chart outputs will be displayed in the 'Console' tab. 
Clicking on anywhere on the map with the 'Inspector' tab open will allow the user to identify pixel data at clicked location.

Innovators and Contributions

All innovation preformed at DEVELOP National Program, NASA Langley Research 

Center, Hampton, VA. 23681

1. Amy Wolfe- Lead Software Developer
2. Britta Dosch- Project Manager
3. Jacob Patrick- Contributor, Software Testing


