![Shaun Levick](Logo3.png)

# Introductory Remote Sensing (ENV202/502)
Prac 6 - Image classification - part 2: validation and accuracy assessment
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

The objective of this lab is to further your understanding of the image classification process, improve the classification from last week, and learn how to evaluate image classification results and conduct an accuracy assessment using independent validation data.

----------

## 1. Load up your previous classification from last week

Open you script from last week's lab. If you did not save it, repeat the steps from [Lab 4](https://github.com/GautamDeepak/Intro_RS/edit/master/Intro_RS_Prac04.md) and be sure to save it this time.

I have provided the full code below, but remember that you need to manually collect the training data and assign landcover properties.
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

![Figure 1. Classified map](Prac5/l4_classified.png)

-----
## Improving the Classification

We ended last week with a discussion on whether or not we were happy with this classification. Even without any quantitative data, it was clearly lacking in some regions. How can we improve it? There are a few option we can explore:

1. Change the training sample size. We only sampled 25 pixels per class. This was a lot of clicking, but we could use polygons instead of points to sample more pixels for training.
2. Change the sampling strategy. We collected an even number of points per class, but some landcover class cover much more area than others. We could experiment with a stratified sampling approach instead.
3. Change the classifier. We used a CART classifier, we could try a different approach such as a support vector machine (SVM) or randomForest (randomForest) approach.
4. Change the bands. We could add ancillary information, such as elevation data, or a derived index such as NDVI to provide for information for class discrimination.
5. Change the image. We used a winter scene from Landsat-8. We could try a summer scene, or switch to a Sentinel-2 image.


-------
### Thank you

I hope you found that useful. A recorded video of this tutorial can be found on my YouTube Channel's [Introduction to Remote Sensing of the Environment Playlist](https://www.youtube.com/playlist?list=PLf6lu3bePWHDi3-lrSqiyInMGQXM34TSV) and on my lab website [GEARS](https://www.gears-lab.com).

#### Kind regards, Shaun R Levick
------
