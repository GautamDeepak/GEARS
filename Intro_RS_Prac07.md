![Shaun Levick](Logo3.png)

# Introductory Remote Sensing (ENV202/502)
Prac 7 - Working with SAR data in Google Earth Engine
--------------

### Acknowledgments
- Google Earth Engine Team
- Google Earth Engine Developers group

------

### Prerequisites
-------------

Completion of this Prac exercise requires the use of the Google Chrome browser and a Google Earth Engine account. If you have not yet signed up - please do so now in a new tab: [Earth Engine account registration](https://signup.earthengine.google.com/)

Once registered you can access the Earth Engine environment here: https://code.earthengine.google.com

Google Earth Engine uses the JavaScript programming language. We will cover the very basics of this language during this course. If you would like more detail you can read through the introduction provided here: [JavaScript background](https://developers.google.com/earth-engine/tutorials/tutorial_js_01)

------------------------------------------------------------------------

### Objective


The objective of this lab is to learn how to evaluate image classification results and conduct an accuracy assessment using independent validation data.

----------

## Visualising Sentinel-1 data

1. Open up Earth Engine and type "Sentinel-1" into the search bar. Click on the Sentinel-1 result and read through the background information on the satellite and image properties.

![Figure 1. Search for Sentinel-1 data](Prac7/search.png)

![Figure 2. Sentinel-1 information](Prac7/info.PNG)

2. Sentinel-1 has different polarisation options - remember that "VV" means vertically polarised signal transmitted out and vertically polarised signal received, whereas VH refers to vertically polarised signal transmitted out, and horizontally polarised signal is received.
3. First up we need to filter the Sentinel-1 image collection (COPERNICUS/S1_GRD), using the script below. Be sure to use the geometry tool to create a point geometry over your region of interest (we will use the Tully region of north Queensland, Australia, as an example. Tully has lots of sugercane and banana plantation) and rename it "roi".

![Figure 3. Tully](Prac7/roi.PNG)

```JavaScript
// Filter the collection for the VV product from the descending track
var collectionVV = ee.ImageCollection('COPERNICUS/S1_GRD')
	// Filter for the swath mode that we want
    .filter(ee.Filter.eq('instrumentMode', 'IW'))
	// filter for the polarisation that we want
    .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))
	// We are choosing the descending orbit here
    .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING'))
	// filter by the spatial location
    .filterBounds(roi)
	// select for the VV mode (vertical transmit vertical received)
    .select(['VV']);
print(collectionVV);

// Filter the collection for the VH product from the descending track
var collectionVH = ee.ImageCollection('COPERNICUS/S1_GRD')
	// Filter for the swath mode that we want
    .filter(ee.Filter.eq('instrumentMode', 'IW'))
	// filter for the polarisation that we want
    .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VH'))
	// We are choosing the descending orbit here
    .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING'))
	// filter by the spatial location
    .filterBounds(roi)
	// select for the VH mode (vertical transmit horizontal received)
    .select(['VH']);
print(collectionVH);
```

4. Navigate to the console and have a look at the information you printed. Using the drop down arrows you can assess how many images are present in teh collection for your region of interest.

![Figure 4. Console](Prac7/console.PNG)


5. Centre the map view over your region of interest. Note this script will always put ROI in the centre of your display. Play with the number "13" to see what happens. The zoom scale ranges from 1 to 22.

```JavaScript
//Let's centre the map view over our ROI
Map.centerObject(roi, 13);
```

6. Use the median reducer to obtain the median pixel value across the all years for each pixel.

```JavaScript
// compute the median VV values for each pixels of all the images
var medianVV = collectionVV.median();
```
7. Plot the median pixel values to the map view. Adjust the min and max visualisation parameters according to your chosen scene - us the inspectors to help you establish the value range.

```JavaScript
// Adding the VV layer to the map
Map.addLayer(medianVV, {min: -14, max: -7}, 'VV - median');
```

![Figure 6. Mapping VV](Prac7/VV.PNG)


8. Explore the image and examine which landscape features have high backscatter intensity (white), and which have low intensity (black).
9. Now derive the the VH median layer, and map it
```JavaScript
//Calculate the median VH values for each pixels from all the images 
var medianVH = collectionVH.median();
Map.addLayer(medianVH, {min: -20, max: -7}, 'VH - median');

```

![Figure 7. Mapping VH](Prac7/VH.PNG)

10. Explore how VV and VH differ in their sensitivity to different land surfaces

11. Next we will experiment with making an RGB composite from the SAR data. To do this we need to create three layers that we can place into the Red, Green, and Blue channels.

```JavaScript
// Create a 3 band layers by selecting from different periods (months)
var summer = ee.Image(collectionVV.filterDate('2019-12-01', '2020-02-28').median());
var autumn = ee.Image(collectionVV.filterDate('2020-03-01', '2020-05-30').median());
var winter = ee.Image(collectionVV.filterDate('2020-06-01', '2020-08-31').median());

//Add the temporal composite to the map
Map.addLayer(summer.addBands(autumn).addBands(winter), {min: -12, max: -7}, 'Season composite');
```

![Figure 9. Temporal RGB composite](Prac7/RGB.PNG)

12. Now try the same for VH
13. Experiment with mixing VV and VH in a RGB composite
14. Think about how this information differs to the optical data you have used so far, and how it could compliment it.

## 2. Complete script

```JavaScript
// Filter the collection for the VV product from the descending track
var collectionVV = ee.ImageCollection('COPERNICUS/S1_GRD')
	// Filter for the swath mode that we want
    .filter(ee.Filter.eq('instrumentMode', 'IW'))
	// filter for the polarisation that we want
    .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))
	// We are choosing the descending orbit here
    .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING'))
	// filter by the spatial location
    .filterBounds(roi)
	// select for the VV mode (vertical transmit vertical received)
    .select(['VV']);
print(collectionVV);

// Filter the collection for the VH product from the descending track
var collectionVH = ee.ImageCollection('COPERNICUS/S1_GRD')
	// Filter for the swath mode that we want
    .filter(ee.Filter.eq('instrumentMode', 'IW'))
	// filter for the polarisation that we want
    .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))
	// We are choosing the descending orbit here
    .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING'))
	// filter by the spatial location
    .filterBounds(roi)
	// select for the VH mode (vertical transmit horizontal received)
    .select(['VH']);
print(collectionVH);

//Let's centre the map view over our ROI
Map.centerObject(roi, 13);

// compute the median values for each pixels of all the images
var medianVV = collectionVV.median();

// Adding the median VV layer to the map
Map.addLayer(medianVV, {min: -14, max: -7}, 'VV - median');

//Calculate the median VH values for each pixels from all the images 
var medianVH = collectionVH.median();
Map.addLayer(medianVH, {min: -20, max: -7}, 'VH - median');

// Create a 3 band layers by selecting from different periods (months)
var summer = ee.Image(collectionVV.filterDate('2019-12-01', '2020-2-28').median());
var autumn = ee.Image(collectionVV.filterDate('2020-03-01', '2020-05-30').median());
var winter = ee.Image(collectionVV.filterDate('2020-06-01', '2020-08-31').median());

//Add the temporal composite to the map
Map.addLayer(summer.addBands(autumn).addBands(winter), {min: -12, max: -7}, 'Season composite');

```

------
### Thank you

I hope you found that useful. A recorded video of this tutorial can be found on my YouTube Channel's [Introduction to Remote Sensing of the Environment Playlist](https://www.youtube.com/playlist?list=PLf6lu3bePWHDi3-lrSqiyInMGQXM34TSV).

#### Kind regards, Shaun R Levick (edit Deepak Gautam)
------


### The end
