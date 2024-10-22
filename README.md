# How you can calculate building footprint SQFT with satellite image recognition
building_footprint_SQFT_ML_model


## Introduction
In commercial real estate, accurate measurement of building size is vital, ranking as the fourth most important piece of information after the legendary “location, location, location.” However, for real estate managers overseeing numerous properties, physically measuring each one is impractical and time-consuming. Site visits, digging up architecture plans in archives, or even using Google Maps fall short when dealing with large portfolios.

To address this challenge, we have developed an algorithm that revolutionizes building measurement. This algorithm leverages satellite imagery and advanced data analysis techniques to swiftly determine a building’s square footage, saving valuable time for GIS and real estate professionals.


Our algorithm allows users to input an address, retrieve a satellite photo, isolate the target building, and calculate its square footage within seconds. This innovative approach eliminates the need for manual measurements, empowering professionals to focus on strategic tasks such as market analysis and portfolio optimization.
In this article, we delve into the development and implementation of our algorithm, discussing its methodology, benefits, and implications for real estate management.

## Setting up the algorithm
The building shape area model uses a hierarchical clustering and regression model to create a method that allows real estate companies to measure the area of the building based solely on a map image of the building address without any costly analyst work. The final model’s R squared value of 0.89 on a validation sample gives the user a reasonably accurate measurement that can be used to filter search results. It can also be used as an input for further analysis.

CNN architecture that we used in building feature extraction is not very useful in this case. Since it is indifferent to object size by design and thus not useful in predicting building shape areas. We had to take a different approach from the previous three models to measure building shape areas.

Here are some of the libraries we will use throughout the algorithm.
## Getting the data
For this algorithm, we used a subset of the City of Chicago Building dataset. You can use any dataset that has a building address, and size to customize for your use case and region. We did not use the actual building footprints from this data set in the training of this model. We let our semi-supervised algorithm build the shape object and compared the object size to the sqft metric in this dataset.
https://data.cityofchicago.org/Buildings/Building-Footprints-current-/hz9b-7nh8
https://medium.com/@skavinskyy/creating-a-dataset-of-satellite-images-from-address-data-in-python-for-your-ai-ml-recognition-951694d464ef

The main step to prepare the data is to download a Google Map image for each of the locations based on latitudes and longitudes. You can read a more detailed description of this in this article. Here we are bringing in the images we prepared before:

## Denoising the Image

We remove most noise from the image by isolating the color of our main object in the image. We know that the building of interest is in the middle of the image. We cannot assume that the color we want is the same between images and we cannot assume that the middle point is exactly on the main object because of an error in the geocode, but we can overcome this problem with a few workarounds. We find the median value of a small box around the middle of the image. We remove any other color from the image that is not the median that was found in the middlebox and we get results similar to Image 2. This allows us to remove a large portion of unnecessary detail from the images thus removing some complexity so that the clustering step performs better.


## Segmenting the buildings
There are often multiple buildings in our image. We use each denoised image as an input into our Ward hierarchical algorithm from Scikit-learn to separate each of the buildings.
Picking the building of interest
Then we use the same assumption that our building of interest is in the middle so we isolate only one cluster that is in the middle. We then calculate the number of pixels in the cluster that we believe to be our building of interest. The results of these steps are demonstrated in images 4 and 5.
![image](https://github.com/tskavinskyy/building_footprint_SQFT_ML_model/assets/4218519/b59aa833-67b7-4868-a346-9cc4babcf315)

We repeated the same procedure for each image creating an array of pixel counts of buildings of interest in each image. We calculated the number of pixels that we call pixel count value as our predictor of building shape area for our regression model.

![image](https://github.com/tskavinskyy/building_footprint_SQFT_ML_model/assets/4218519/e6881e15-485a-41f4-9aa1-e808152bf6f0)

## Calculating actual SQFT

We created an algorithm that outlines the main building and calculates the number of pixels in the outline. Now we need to convert the size in pixels to a known size in sqft by using a model. The regression model converts the pixel count into a real shape area and automatically gives us test metrics to evaluate the quality of our model.
![image](https://github.com/tskavinskyy/building_footprint_SQFT_ML_model/assets/4218519/74615ede-5ff7-4a3a-a991-0ef23a020eed)

When measuring the building’s shape area by our original model, we see a statistically significant relationship between derived pixel count and the actual building shape area with an R-squared value of 0.61. Better geocodes improved the R-squared value of our model to 0.79.

## Adding Cooks Distance.
We also used Cook’s distance method from Pastell (2013) to remove influential observations from our model sample.
Result:

We created a holdout sample with 30% of the observations of our total sample to test the results of our model.

## Result:

Our final model had a pixel count variable that we calculated from the image using our hierarchical clustering algorithm as a statistically significant predictor of building shape area. The model details are presented in table J. The final model’s R squared value was 0.86 on the training sample and 0.89 on a validation sample. Chart 4 shows a strong relationship between predicted versus actual building shape areas.

![image](https://github.com/tskavinskyy/building_footprint_SQFT_ML_model/assets/4218519/ba3d5484-d9d8-4858-b421-fb81e037fefc)
![image](https://github.com/tskavinskyy/building_footprint_SQFT_ML_model/assets/4218519/6ab33e30-a447-4458-a567-41ce0c11736a)


## Interpreting the results:
We can predict the building shape area from satellite images with a great degree of confidence. The building shape area model only predicts the size of the building footprint. If there is shared usage of the building or multiple stories, satellite imagery is not enough to allow identification of all the details of building size, but it can provide a useful approximation to generally distinguish between different building sizes.

Limitations of geocoding and outdated images bring a fair amount of noise to the data that is outside of the scope of the models. Showing the models more examples will help improve the accuracy and help them generalize.

This algorithm can substitute in-person site visits to identify building features at the preliminary stages of real estate selection thus saving companies time and resources. Real estate decision-makers can use the results of this algorithm to quickly filter and find desired buildings. Identifying these building features can be used as inputs to improve other analytics projects.

## Conclusion
The ability to swiftly identify building features, such as square footage, through our algorithm empowers real estate decision-makers to streamline their processes. By utilizing the algorithm’s results, professionals can efficiently filter and locate desired buildings, expediting the decision-making process and optimizing resource allocation.

Moreover, the identified building features serve as valuable inputs for enhancing other analytics projects within the real estate industry. This algorithm opens doors to new opportunities for data-driven insights and informed decision-making, ultimately leading to improved portfolio management, investment strategies, and market analysis.

As technology continues to advance, we anticipate further refinements and enhancements to our algorithm, unlocking even greater potential for the industry. The future holds immense possibilities for leveraging such algorithms in conjunction with artificial intelligence and machine learning to further enhance accuracy, automation, and efficiency in real estate management.

More Details https://medium.com/@skavinskyy/how-you-can-calculate-building-footprint-sqft-with-satellite-image-recognition-18e206302c7e
