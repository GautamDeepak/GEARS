![Shaun Levick](Logo3.png)

# Introductory Remote Sensing (ENV202/502)
Prac 4 - Image Classification - part 1
--------------

### Acknowledgments
- Google Earth Engine Team
- Google Earth Engine Developers group
- David Saah and Nicholas Clinton

------

### Prerequisites
-------------

Completion of this Prac exercise requires the use of the Google Chrome browser and a Google Earth Engine account. If you have not yet signed up - please do so now in a new tab: [Earth Engine account registration](https://signup.earthengine.google.com/)

Once registered you can access the Earth Engine environment here: https://code.earthengine.google.com

Google Earth Engine uses the JavaScript programming language. We will cover the very basics of this language during this course. If you would like more detail you can read through the introduction provided here: [JavaScript background](https://developers.google.com/earth-engine/tutorials/tutorial_js_01)

------------------------------------------------------------------------

### Objective


The objective of this prac is to gain understanding of the image classification process and explore ways of turning remotely sensed imagery into landcover maps.

----------

## 1. Loading up the the Landsat 8 image

1. The first step is to get a cloud free image with which to work. So far we have worked on the elevation data and the Sentinel-2 images. Todays Prac will be on Landsat 8 image. Look for USGS Landsat 8 Surface Reflectance Tier 1 imagery, import it, and rename the imageCollection to L8. Refer to previous labs if you dont know how to search for, import, and rename imageries.

2. Building on from last week, we can use the point drawing tool (teardrop icon) from the geometry tools and draw a single point in the region of interest - let's use the town of Cairns for this example.  Then 'Exit' from the drawing tools.  Note that a new variable is created in the imports section, containing the single point, imported as a Geometry.  Change the name of this import to "roi" - short for region of interest.

![Figure 1. Navigating to Cairns](Prac4/roi.PNG)

3. Filter the image collection spatially (using the filterBounds command), and temporally (using the filterDate command), and lastly sort the images by cloud cover (using the 'CLOUD_COVER' keyword) and extract the least cloudy scene (using the "first" command). Run the script below to extract our desired image from the Landsat 8 collection and add it to the map view as a true-colour composite:

```JavaScript
// Lets filter the image collection to get a single image
var anImage = L8
  // L8 is an image collection so lets  include a geographic filter to narrow the search to images at the location of our point
  .filterBounds(roi)
  // further filter the collection by the the date range we are interested in
  .filterDate('2016-05-01', '2016-06-30')
  // Next we will also sort the collection by a metadata property, in our case cloud cover is a very useful one
  .sort('CLOUD_COVER')
  // Now lets select the first image out of this collection - i.e. the most cloud free image in the date range and over the region of interest
  .first();

// lets display the filtered cloudfree image to our mapping layer using true color composite
Map.addLayer(anImage, {bands: ['B4', 'B3', 'B2'],min:0, max: 3000}, 'True colour image');
```

Question: In last prac we used "CLOUD_COVERAGE_ASSESSMENT" to sort the images by cloud cover, however, in this prac "CLOUD_COVER" is used. Can you find out why?

![Figure 2. Adding image to map view](Prac4/true_color.PNG)

4. Have a look around the scene and familiarise yourself with the landscape. You'll notice the image is quite dark image - we can adjust the brightness/contrast using the settings wheels for the layer we created in the Layers tab. Slide the Gamma adjuster slightly to the right (from 1.0 to 1.4) to increase the brightness of the scene. Alternatively, in the previous line of script, you can encode the gamma value in the visualisation parameter as: "{bands: ['B4', 'B3', 'B2'],min:0, max: 3000, gamma:1.4}" 

![Figure 3. Brightness adjustment](Prac4/gamma.PNG)

## 2. Gathering training data for classification

1. The first step in classifying our image is to collect some training data to teach the classifier what each landcover looks like.  We want to collect representative samples of reflectance spectra for each landcover class of interest. First lets gather some sample points to teach classifier what urban area is goig to look like. Hover on the 'Geometry Imports' box next to the geometry drawing tools and click '+ new layer.'

![Figure 3. New Geometry](Prac4/newgeometry.PNG)

3. The new layer will be imported to the "Geometry Import" box as well as to the import section of you script. Rename the new layer to 'urban'. Notice that the renameing will change the name of the layer in the "Geometry Imports" box.

![Figure 3. Rename Urban](Prac4/rename_urban.PNG)

4. Now, locate points in the map that represents the urban or built up areas (buildings, roads, parking lots, etc.). Clicking on the urban area will collect the training data for the urban. Sample minimum of 25 points. For robust classification make sure you are sampling from different types of urban areas (not just one). See example below.  

![Figure 3. Collect the urban class](Prac4/urbandata.PNG)

5. Next you need to configure the urban geometry import (cog-wheel, top of the script in imports section) as follows.  Click the cog-wheel icon to configure it, change 'Import as'  from 'Geometry' to 'FeatureCollection'.  Use 'Add property' landcover and set its value to 0.  (Subsequent classes will be 1, 2, 3 etc.). You can also change the color of the 'teardrops', here if you like. When finished, click 'OK'.

![Figure 5. The geometry dialogue box](Prac4/urbanConfigure.PNG)


6. Repeat the steps 1-5 for each land cover class that you wish to include in your classification. Add 'water', 'forest', 'agriculture' , and 'bareland' next - collect >25 points for each class. Use the cog-wheel to configure the geometries - change the type to FeatureCollection and set the property name to landcover with values of 1, 2, 3, and 4 for the different classes.

![Figure 6. Adding classes](Prac4/classes.PNG)

7. Now we have five classes defined (urban, water, forest, agriculture, bareland), but before we can use them to collect training data we need to merge the 5 landcover features into a single collection, called a FeatureCollection. Run the following line to merge the geometries into a single FeatureCollection:

```javascript
// Merge the 5 landcover class features into a single featureCollection
var LandCoverClasses = urban.merge(water).merge(forest).merge(agriculture).merge(bareland);
```

8. Print the feature collection and inspect the features.

```javascript
// Print the land cover feature collection
print('The land cover feature collection is: ',LandCoverClasses);
```
![Figure 7. Printing classes](Prac4/printclass.PNG)


## 3. Creating the training data

1. Now we can use the FeatureCollection we created to drill through the image and extract the reflectance data for each point, from every band. We create training data by overlaying the training points on the image.  This will add new properties to the feature collection that represent image band values at each point:

```javascript
// add new properties to the "LandCoverClasses" - the new property is the reflectance data from the above bands
LandCoverClasses = anImage.select(bands).sampleRegions({ // sample the reflectance from selected bands
  collection: LandCoverClasses, // save the reflectance to the LandCoverClasses
  properties: ['landcover'],
  scale: 30
});

// print our training dataset
print('The training dataset is: ', LandCoverClasses);
```

2. After running the script the training data will be printed to the console. You will notice that the 'properties' information has now changed, and in addition to the landcover class, for each point there is now a corresponding reflectance value for each of the selected band of the image.

![Figure 8. Printing training data](Prac4/training.PNG)


## 4. Train the classifier and run the classification

1. Now we can train the classifier algorithm by using our examples of what different landcover class look like from a multi-spectral perspective.

```javascript
// train our classifier. Here we used cart classifier.
var classifier = ee.Classifier.smileCart().train({
  features: LandCoverClasses,
  classProperty: 'landcover',
  inputProperties: bands
});

```

2. The next step is then to apply this knowledge from our training to the rest of the image (i.e. the pixels that we did not use for training) - using what was learnt from our supervised collection to inform decisions about which class other pixels should belong to.

```javascript
//Run the classification for the entire scene
var classified = anImage.select(bands).classify(classifier);
```

3. Display the results using the mapping function below. You may need to adjust  the colours, but if the training data have been created with urban=0, water=1, forest=2, and agriculture=3 - then the result will be rendered with those classes as yellow, blue and green, respectively.


```javascript
//Display classified map. The color scheme defined below is set to according to the numbering of the class. e.g. class0 was urban which is set to red
Map.centerObject(roi, 10);
Map.addLayer(classified, {min: 0, max: 4, palette: ['red', 'blue', 'darkgreen','lightgreen', 'gray']}, 'Classified map');
```

4. From the 'Geometry Import' box, untick all the geometries for better visualisation.


![Figure 9. Classified map](Prac4/classified.PNG)

5. Congratulations - your first landcover classification! Zoom in and explore the results of your classification. 
- Are you happy with the classification?
- How could it be improved?
- Try adding in some extra classes for landcover categories that show signs of confusion

We will look at how to refine this and discuss limitations and avenues for improvement next week.

### 5. Complete script (excluding the ImageCollection, roi, and landcover features)
```JavaScript
// Lets filter the image collection to get a single image
var anImage = L8
  // L8 is an image collection so lets  include a geographic filter to narrow the search to images at the location of our point
  .filterBounds(roi)
  // further filter the collection by the the date range we are interested in
  .filterDate('2016-05-01', '2016-06-30')
  // Next we will also sort the collection by a metadata property, in our case cloud cover is a very useful one
  .sort('CLOUD_COVER')
  // Now lets select the first image out of this collection - i.e. the most cloud free image in the date range and over the region of interest
  .first();

// lets display the filtered cloudfree image to our mapping layer using true color composite
Map.addLayer(anImage, {bands: ['B4', 'B3', 'B2'],min:0, max: 3000, gamma:1.4}, 'True colour image');

// Merge the 5 landcover class features into a single featureCollection
var LandCoverClasses = urban.merge(water).merge(forest).merge(agriculture).merge(bareland);

// Print the land cover feature collection
print('The land cover feature collection is: ',LandCoverClasses);

// These will be the bands whose reflectance data will be extracted from the image for training purpose
var bands = ['B2', 'B3', 'B4', 'B5', 'B6', 'B7'];

// add new properties to the "LandCoverClasses" - the new property is the reflectance data from the above bands
LandCoverClasses = anImage.select(bands).sampleRegions({ // sample the reflectance from selected bands
  collection: LandCoverClasses, // save the reflectance to the LandCoverClasses
  properties: ['landcover'],
  scale: 30
});

// print our training dataset
print('The training dataset is: ', LandCoverClasses);

// train our classifier. Here we used cart classifier.
var classifier = ee.Classifier.smileCart().train({
  features: LandCoverClasses,
  classProperty: 'landcover',
  inputProperties: bands
});

//Run the classification for the entire scene
var classified = anImage.select(bands).classify(classifier);



//Display classified map. The color scheme defined below is set to according to the numbering of the class. e.g. class0 was urban which is set to red
Map.centerObject(roi, 10);
Map.addLayer(classified, {min: 0, max: 4, palette: ['red', 'blue', 'darkgreen','lightgreen', 'gray']}, 'Classified map');


```
-------
### Thank you

I hope you found that useful. A recorded video of this tutorial can be found on my YouTube Channel's [Introduction to Remote Sensing of the Environment Playlist](https://www.youtube.com/playlist?list=PLf6lu3bePWHDi3-lrSqiyInMGQXM34TSV).

#### Kind regards, Shaun R Levick (edit Deepak Gautam)
------


### The end