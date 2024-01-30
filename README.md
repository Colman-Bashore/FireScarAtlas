# Machine Learning Based Classification and Web Visualization of Burn Scars in Washington State using Google Earth Engine and 40 years of LANDSAT Satellite Imagery

##### Author: Colman Bashore, Middlebury College
##### Advisor: Professor Jeff Howarth, Middlebury College

![Decades](/Assets/Decades_img.png)

[Link to interactive Washington Fire Scar Atlas web application](https://cbashore.users.earthengine.app/view/washington-fire-scar-atlas)

[Link to Google Earth Engine Code repository](https://code.earthengine.google.com/?accept_repo=users/cbashore/fireapp)


### Introduction 
Over the course of the past decade the Pacific Northwest has experienced a wave of wildfires that have destroyed towns, taken lives, and covered massive areas in dark smoke. While thousands of firefighters have worked tirelessly to contain these blazes, in many situations the largest fires cannot be stopped by humans alone. Often, a fire will only burn itself out when it runs into the burn scar of a previous fire, a barren area containing no potential fuel for the active fire. Knowledge of this pattern can help assist firefighters in the containment of fires, and mapping burn scars can give us an understanding of if this pattern actually holds true in a larger number of situations. If we map burn scars of an entire state over 40 years, can we expect to see that burn scars in consecutive years rarely overlap? Mapping burn scars using satellite imagery can also give us an idea of which areas burn over and over again over the course of many years. The advantage of using satellite imagery is the large temporal and areal extent that is accessible as compared to using drone or airplane images. Previous attempts to create burned area maps include the European Space Agency’s fire_cci layer which is a global burned area layer, however this layer is derived from the MODIS satellite which only has a spatial resolution of 250 meters. Long et al. (2019) created a product known as GABAM (global burned area map) using LANDSAT satellite imagery at a 30-meter spatial resolution. This project encountered many challenges, such as identifying burn scars in diverse biomes across the globe and differentiating between crop harvesting and burns. However, this product is the current best available burned area map and runs from 2000 to 2018. My project attempts to replicate some of the basic methods of the Long et al. study to create a 30 meter burned area dataset for the state of Washington across 40+ years. The challenges that confront anyone who tries to build a dataset such as this are many. First, satellite imagery is always subject to issues with cloud cover and anomalies in reflectance which will make it much harder for a classifier to correctly identify land cover types. Land cover types are already difficult to distinguish from imagery as is; bare ground areas and burn scars appear similar as do completely bare agricultural fields. Satellite imagery also faces the issue of poor spatial resolution. The LANDSAT satellite is fairly high resolution at 30 meters, but other products such as the fire_cci map suffer due to coarse spatial resolution. In addition to making static burned area maps, in order to produce a dataset of many maps over many years, images from several different individual satellites, with different reflectance bands will have to be used. All of these factors are threats to the completion of a product like GABAM or the collection of layers that this project intends to create. Within the overarching goal of this project there are four sub-goals:
Goals:
-	Develop an imagery classification algorithm that isolates burn scars for a given year and region from 30m LANDSAT imagery.
-	Create layers of burn scars for as many years as possible, including years that have not been covered by GABAM, by creating reusable scripts for different years and satellites.
-	Combine those raster images as static layers into a separate interactive web app. Create unique cartographic methods for the user to interactively study the yearly layers as well as add a layer of areas that have burned many times. 
-	Describe the process and challenges of creating a regional burned area map using satellite imagery and the added benefit for fire containment that this study provides.

### Methods

#### Analysis
The first stage of this project was to train a random forest classifier to create a regional single-year burnt area map derived from LANDSAT imagery. I tested my first round of classification on imagery from the LANDSAT 8 satellite from 2018. The first step was to load in the LANDSAT 8 collection and filter it to images from post-fire season 2018. I also clipped the imagery to the outline of the state of Washington and applied a cloud mask to remove as many clouds from the images as possible. I then had a mosaic of LANDSAT images from late 2018 in the shape of Washington State from which I planned to classify the landcover to determine fire scars. Long et al. use a total of 14 parameters in their classification. I used the spectral module to calculate the same spectral indices used by Long et al. These indices were: NBR, NBR2, BAI, GEMI, MIRBI, NDVI, SAVI, and NDMI. In addition to the spectral indices both Long et al. and myself also used the 6 surface reflectance bands native to LANDSAT. After calculating the 8 spectral indices for the imagery of Washington I then hand selected 175 feature points defined as either water, snow/ice, nonburn, or burn. This collection of features was then split into training and testing collections. I used a random forest classifier which I trained on the training points and using the 6 reflectance bands and 8 spectral indices. Once trained, I ran the classifier on my image mosaic so that it produced a classified raster layer of the state of Washington divided into water, snow/ice, nonburn, and burn. The classified image was then compared to the testing points collection and a confusion matrix and kappa coefficient were generated to measure accuracy.
With the aim of creating a layer of burn scars that is easier to visualize and less noisy, I then thresholded the image to isolate only the burn scars and then used a neighborhood (kernel) operation to clump the burn scars into distinct objects. Then, to reduce noise, I filtered the burn scars objects to only keep the burn scars larger than one square kilometer in area. This sequence of steps produced a single-year layer of burn scars in Washington. 

The second stage of the project was to reproduce my workflow for generating a single year of burn scars across 40 years of LANDSAT imagery. To accomplish this task, I broke down all of my steps into callable functions within Google Earth Engine and created two versions of one overarching function that could create a complete burn scar layer needing only the year I wanted to calculate for. I created one version for use on the Landsat 5 and 7 satellites and one version for the Landsat 8 and 9 satellites because these two pairs of satellites each had the same named reflectance bands. To accomplish this task of only needing on input I created a large dictionary of information about each year I needed to calculate, including which satellites imagery would be pulled from, and appropriate dates to filter by for that year. Then from this script I created burn scars layers from 1984 to 2023 and exported these as assets into my Earth Engine account such that they could then be loaded quickly into an app. 


#### Visualization
This project not only focused on creating a model to estimate burn scars from imagery but also aimed to create a web application for the general public that could be used to answer important questions about the trends of fire in Washington State. The emphasis of this visualization stage was on web-based cartography and storytelling through a Google Earth Engine app. Using my 40 layers of burn scars I created an app that answers questions such as, "do the spatiotemporal trends of fire in Washington match popular media headlines?", "what areas in the state have burned most recently or burned the most times?", and  "how have different regions in the state been impacted by fire, what are the natural causes of these differences, and what are the human impacts on these regions?" The app is designed as an atlas of burn scars that is highly interactive and geared towards the general public. The first component of this app is a page that introduces the user to the atlas and shows a chart of burned area over time, a map of what areas have burned most, and a map of when areas have been burned most recently. The goal behind this stage was to demonstrate the overall trend of increasing acreage burnt as well as allow the user to peruse the state as a whole and examine broad trends. The app is designed similarly to a story map in a software such as ArcGIS or Mapbox, such that the user steps through different stages of the app in a linear manner. The second stage is an examination of the state on a decade-by-decade basis. I aggregated by burn scar layers into 10 year chunks from 1984 to 2023 (exactly 40 years). These decade layers can then be toggled on an and off to make comparisons. The next three stages of the app focus on specific regions within the state that have experienced fire and deserve closer attention. I define the the three distinct regions I identified as having significant histories of fire as the "North Cascades Region", an area stretching from North Cascades National Park through the Methow valley, the "Chelan - Grand Coulee Region", which contains Wenatchee, Lake Chelan, and the Grand Coulee Damn, and finally the "Columbia Region" which I define as a broad region surrounding the Columbia River between Yakima, the Tri-Cities, and Hanford Reach National Monument. Each of these regions gets its own panel within the app and for each region the user can pull up burn scars from any year from 1984 to 2023. Part of the motivation for this project was to examine how new burns rarely overlap with recent burns because they run into areas of low fuel. To visualize this hypothesis when a year is selected its burn scars are shown at full saturation and the previous two years burn scars are also shown but in a much more subdued color. The user also has the ability to toggle on several reference layers, namely a Normalized Burn Ratio layer, the LANDSAT composite image that the model ran on, and a 2021 population layer. The goal of the first two reference layers is to give the user both an idea of what the model is based off of as well as get a better sense of the kind of terrain that the wildfires are burning on. The population layer aims to call out areas of close contact between human settlement and burns. While these areas luckily rarely overlap, the population layer and the basemap itself allow the user to examine what towns have been hazardously close to burns. 


### Results
##### Burn Scar Model Accuracy
The first goal of this project was to use a classifier to create burn scars based on satellite imagery, which required many tweaks in input data and bands and indices from which to classify an image. During this process I implemented a confusion matrix and kappa coefficient calculation within Google Earth Engine to measure the accuracy of my model. This was done by splitting my point dataset into training and testing points, training the classifier with the training points, classifying an image and then measuring its accuracy on guessing the nature of the testing points. For my model I was specifically testing on my first script which focused only on 2018 and was seperating images into water, burned, and notburned. My confusion matrix for my model trained and tested on 175 data points from 2018 produced an overall accuracy of 0.953 and a Kappa Coefficient of 0.912 implying that the model was able to predict burn scars above chance levels. 

##### Patterns of Burn Scars
On the visualization end of this project the burn scar layers created by the model and the subsequent app revealed several interesting trends. First the total areas determined as burned each year has seen an uptick over time. The following chart is included within the web app and exhibits the annual percent difference from median acres burned over 40 years. The median burned area for this dataset was approximately 54,000 acres.
![Time Chart](/Assets/ee-chart.png)
The year 2020 stands out above all other years with an anomaly of over 1000% more burned area than the median burned acreage. Verifying these specific statistics would require more fine tuning of our classification model, but the relative comparisons of yearly burns tell a compelling story. 
On a regional scale, there were several different patterns that emerged. The Columbia Region had the most recurring burns, with some areas burning four or five seperate times. The Chelan - Grand Coulee Region had relatively few major burns from 1984-2000, but since then has experienced several massive fires. Both the North Cascades and Chelan - Grand Coulee Regions have experienced major wildfires despite differing biomes, with the North Cascades being heavily forested and the Chelan-Grand Coulee being far less so and predominantly agricultural or arid. 

### Discussion
##### General Trends and Natural Conditions
As illustrated by figure one, the temporal story of wildfires in Washington has been one of increasing magnitude. Not only are there more fires in the most recent decade of data than ever before, but they also burn larger extents. Spatially, the large burns in the past 40 years in Washington have been exclusively east of the Cascade Range which runs the length of Washington north to south, cutting the state in half. Moisture from the Pacific Ocean is typically dumped in the rainy Puget Sound region and the Cascade Range creates a rain shadow behind it giving Eastern Washington a drastically more arid climate than its Western half. The specific regions of focus from my project also have their own unique natural conditions which contribute to or impact their susceptibility to fire.  
The North Cascades region is defined by large mountains and rugged terrain. While the western side of the range experiences a wetter, maritime climate, the eastern slope is more similar to a dry continental climate found across the Mountain West. The eastern slope is still predominantly forested, but it is a drier, ponderosa-pine ecosystem similar to conditions that might be found in Bend, Oregon. While the North Cascades region appears similar to other portions of the Cascades and the forests of northeast Washington and northern Idaho, none of the other forested, mountainous regions in Washington have experienced the impact of fire in the same way. The presence of the rain shadow creates distinctly different natural conditions. It would be worthwhile to conduct this same analysis over the state of Oregon and examine the region of the Deschutes National Forest which is also Ponderosa Pines stretching down the eastern slope of the Cascade Range. 
While geographically close to the North Cascades, the Chelan-Grand Coulee region is much flatter, less forested, and drier save for Lake Chelan itself and several dammed portions of the Columbia River. The transition between the North Cascades and the Chelan – Grand Coulee is characterized by thinning pines and mountain ridges giving way to grassland and rolling valleys. Despite being an extremely dry region, there is significant agricultural development due to irrigation from Lake Chelan and the Columbia River. Much of Washington’s apple crop, which makes up 90% of the nation’s total is grown in this region. The zone of influence of Lake Chelan is a small but dramatic one. The populated eastern tip of the lake is surrounded by green parks and fields for several hundred meters and then quickly transitions to arid conditions. For the inhabitants of Chelan, nestled between the lake and the river, fires would have to be particularly insidious to reach the town itself, but the lake does not offer much protection to the rest of the surrounding area. 
The Columbia River region spans several different biomes but is also an arid region with surprisingly green farmland stretching out from the major rivers that cut through the area. The region centers on a relatively uninhabited area that is dominated by a large military base and the Hanford Nuclear Site. Given the areas dry conditions, live fire exercises have posed a risk to starting brush fires such as the Range 12 fire in 2016 which burned over 175,000 acres. As in Chelan, irrigation from the Columbia River allows for massive agricultural undertakings. From satellite imagery this area looks almost comical, completely dry ridges neighbor bright green fields. The aura of the Columbia and Yakima rivers is directly visible in the agricultural fields that stretch out from them. The patterns of wildfire in this region in particular demonstrate that the dry hills have burned many times but almost never will the same area burn two years in a row and often a subsequent years fire will burn up until it touches a previous burn scar and runs out of fuel. 
##### Human History and Impact
A narrative of destruction and hazard has defined the narrative of fire in Washington for the past decade. My app allows the user to explore the interaction of burn scars with the populated landscape and note which communities fall into positions of risk. In the North Cascades, the most populated locations are nestled in the Methow Valley, largely a popular ski region in the winter for wealthy individuals. A survey of remote workers in the valley found a median income of $202,000 However, the median income for someone living and working year-round was only $57,000. Due to the valley’s winter popularity, it is often the service workers living there year-round who are going to experience the most wildfire impacts and an examination of this region in the app will make clear the number of fires that have grazed this area.  
A similar narrative drives the human inequality of fire in the Chelan-Grand Coulee region. While Lake Chelan is a popular tourist destination for wealthy Washingtonian’s, with a median home listing price of $907,000 in 2023, the average fruit worker earned barely $30,000 a year. Generally, fires have missed the major cities of the region, an examination of the burn scar layers will reveal massive blazes in this area, for example, the Pearl Hill fire of 2020 which stands out clearly between Lake Chelan and Banks Lake. 
The wildfires in southern Washington around the Columbia River have come quite close to human settlements in recent years. In 2018, the city of Kennewick had wildfires make it literally to the doorsteps of its residents. While Yakima itself is largely protected by farmland and the Yakima valley fires in the nearby dry ridges have come threateningly close and covered the valley in dark smoke. 
##### Impact of this Project and App
The main goals of this project were to create a tool that could be used to identify burn scars based only on satellite imagery, roughly following the methodology of Long et al, use this tool to study the fire history of Washington, and present the resulting data to the public. My scripts can be used to model the change in fire patterns over time, which could provide information to firefighters, planners, and climate scientists that would need data on the ways in which fires impact the landscape. This dataset improves upon previous burn scar layers that have only been created at a coarser resolution or not included as many years. Compared to official fire perimeter delineations this dataset also identifies which 30-meter pixels were fully burned which makes clear the differences between fires that burned forest stands down completely and those that create a mosaic of burn. Additionally, the narrative that wildfires have grown is one that is often backed up only with statistics and not interactive visuals. The goal of the web app is to allow the general public to explore the patterns of wildfires and find their own evidence for the claims that wildfires have grown in size.
While the classifier model and app provide value to the field of fire mapping and climate communication, this project can hopefully also serve as a model of undergraduate independent learning in remote sensing and web mapping. This project was my first time using a machine-learning based classifier and I had hardly used Earth Engine Apps, and certainly not attempted to recreate a story map format. My key takeaways from undertaking this project are that it was highly beneficial to take the time at the beginning of the project to gain a solid understanding of how to use a classifier for remote sensing, and it was crucial to focus first on getting the model to work on only a small region and then expanding this to my entire state of Washington. 


### Conclusion
This project covered two main endeavors. First, I developed an Google Earth Engine Script that takes LANDSAT imagery and produces a classified image of burn scars across the extent of a given state for every year that imagery is provided. Second, I created a Google Earth Engine web application that steps the user through the spatial and temporal patterns of burn scars in the state of Washington. Using the burn scar creation script I produced a large dataset of burn scars that is now accessible as an image collection in Google Earth Engine for rapid import and analysis. \
The image collection is accessible at this link: [Burn Scar Asset](https://code.earthengine.google.com/?asset=users/cbashore/fires). \
 The web application demonstrates fire trends over time at the statewide level and also allows users, who may range from the general public to fire and government officials, to explore burns at a closer level in several regions. 
\
While this project achieved its primary goals in only four weeks, the burn scar creation script could be both improved and expanded upon. For the purposes of visualization, the burn scars were filtered to remove any burn scars with a connected area less than one square kilometer. This ended up cutting out more potential burn scars than intended. For the general user this doesn't affect the app, but for future use of the burn scar algorithm to measure fire it would be more effective to reduce the intensity of the filtering. Overall, the classifier identified burn scars that visually aligned with major known fires, and had a high accuracy within its own testing set. However, there were some misidentifications and the results were not tested against some external dataset such the Monitoring Trends in Burn Severity dataset, against which Long et al. tested their results. In particular, the area around Mount St. Helens was often identified as a burn scar in the model. This area has not had major fires as are indicated by the data, rather the surrounding land was deforested and covered in magma and ash following the 1980 eruption of Mount St. Helens. This suggests that the spectral signatures of land impacted by Volcanic eruption is similar to land that has been severely burned. This erorr in the model would require further fine-tuning to clarify.  Overall, the algorithm would benefit from a larger training set and a more definitive dataset that clearly identifies burn scars for the model to be trained on. 
\
Not only would be classifier benefit from increased training but the burn scar algorithm could also be applied across more geographies now that it has been used on the state of Washington. The algorithm has been written so that it would take minimal adjustment to apply the code on a new region given that LANDSAT imagery has global coverage. The app could not be as easily transitioned to another state, however, future visualization work could include a greater emphasis on the non-burn-scar components of fire, such as smoke and particular matter that impact towns. While burn scars luckily rarely overlapped with populated areas directly, the impacts were felt strongly due to the effects of smoke, and a smoke layer in future iterations of the app would provide added value. Finally, future work should be dedicated towards teaching other undergraduate students the process of self-guided learning for remote sensing and cartography. I would like to pass on this description of my project as a way of documenting my methodology of asking a question about the natural world, developing a script to identify patterns in this phenomenon, and then visualize these patterns for others and walk them through my observations. 

### Bibliography
##### Previous Works

Long, T., Zhang, Z., He, G., Jiao, W., Tang, C., Wu, B., Zhang, X., Wang, G., & Yin, R. (2019). 30 m Resolution Global Annual Burned Area Mapping Based on Landsat Images and Google Earth Engine. Remote Sensing, 11(5), Article 5. https://doi.org/10.3390/rs11050489

##### Works Cited

Nature—North Cascades National Park (U.S. National Park Service). Retrieved January 30, 2024, from https://www.nps.gov/noca/learn/nature/index.htm

Tate-Libby, J. (2021). TwispWorks Comprehensive Economic Study of the Methow Valley. https://drive.google.com/file/d/1wpf1_fUfBjPbUL4xBWL8j5lYmZvoklQ1/view?usp=embed_facebook

Wildfires Lead to Helicopter Rescues in California and Destruction in Washington. (2020, September 8). The New York Times. https://www.nytimes.com/2020/09/08/us/wildfires-live-updates.html


##### Data Sources for Scripts
Landsat-5, Landsat-7, Landsat-8, and Landsat-9 Collection 2, Tier 1, Level 2 datasets courtesy of the U.S. Geological Survey

Weber, E., Moehl, J., Weston, S., Rose, A., Brelsford, C., & Hauser, T. (2022). LandScan USA 2021 [Data set]. Oak Ridge National Laboratory. https://
doi.org/10.48690/1527701 

##### Acknowledgements

This project would not have been possible without the guidance of Professor Jeff Howarth and the resources of the Middlebury College Geography Department.