# Canopy_Height In A BD Region
## Target
- Mean The Value Of Canopy Height
- Max and Min
- Visualize The Raster
## Methodology 
 - select the basins by using a link [link] (WWF HydroSHEDS Basins Level 6)
 - collecting data from [link](var canopy_ht = ee.ImageCollection("projects/sat-io/open-datasets/facebook/meta-canopy-height");)
 - visulaizing the Data with Roi,Histrogram
## Code
````
// Map.addLayer(table)
var roi = table.filterBounds(geometry);
Map.centerObject(roi)
// Map.addLayer(roi)
// add data 
var canopy = ee.ImageCollection("projects/sat-io/open-datasets/facebook/meta-canopy-height").mosaic();
// print(canopy)
Map.addLayer(canopy.clip(roi),{},'canopy_height',false)
// print(
//   ui.Chart.image.histogram(canopy,roi,100)
//   )
  // histogram formula (data,regional,resolution)
  // remove the nonforest which is height zero 
  var forest_thr=canopy.gt(0)  // here gt means greater than
  var forest =canopy.updateMask(forest_thr);
  Map.addLayer(forest.clip(roi),{},'forest',false);
  print(
  ui.Chart.image.histogram(forest,roi,100)
  )
  // output has no nonforest object colour ,all in black coloir 
  // get the mean value 
  var canopy_mean=forest.reduceRegion(
    {
      
      reducer : ee.Reducer.mean(),
      geometry:roi,
      scale:100
    }).values().get(0);
    
     print('canopy_mean',canopy_mean)
     
     
     
    // get the max canopyy
    var canopy_max=forest.reduceRegion(
    {
      
      reducer : ee.Reducer.max(),
      geometry:roi,
      scale:100
    }).values().get(0);
    
    
    print('canopy_max',canopy_max)
    
    
    
    // we need to find out 99 percent of tree and their maximum height due of 1 tress in 25 m
    
    // amon the 99% height canophy max value 
    var canopy_99=forest.reduceRegion(
    {
      
      reducer : ee.Reducer.percentile([99]),
      geometry:roi,
      scale:100
    }).values().get(0);
    
    
    print('canopy_max2',canopy_99)
    
    // minimum canophy 
     var canopy_min=forest.reduceRegion(
    {
      
      reducer : ee.Reducer.min(),
      geometry:roi,
      scale:100
    }).values().get(0);
    
    
    print('canopy_min',canopy_min);
    
    
    // classify the image
    
    var cons =ee.Image.constant(0)
    var class1 = cons.where(forest.lt(5),1);
    var class2=class1.where(forest.gte(5).and(forest.lt(10)),2)
    var class3=class2.where(forest.gte(10),3)
     Map.addLayer(class1.clip(roi),{},'class-1',false)
     Map.addLayer(class2.clip(roi),{},'class-2',false)
      Map.addLayer(class3.clip(roi),{min:0,max:3,palette:['black','red','orange','green']},'class-3',false)
     
    
    Export.image.toDrive({
      image:class3,
      description:"Canopy_Height",
      scale:30,
      region:  roi.geometry(),
      crs:class3.getInfo().crs,
      folder:'Canopy',
     })
    
    
  

````

## Reference 
- [Youtube] (https://youtu.be/OKrV5hudcQ4?si=zJciD4kasIpKtkHK)
