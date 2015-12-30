# Google Street View Locaiton Predictor
The emerging field of feature recognition in images is revolutionizing how well computers are able to understand the world around us. Inspired by [geoguessr](https://geoguessr.com/usa/play), my project uses Convolutional Neural Networks (CNN) to discern relevant features that correspond to geographic locations in Colorado. This type of modeling has applications for self-driving cars, where maintaining a keen sense of environment is vitally important. Distinguishing canyon roads from local streets and a clear day from a rainy one will be integral in making smarter autonomous vehicles.

| Section | Topic |
| --- | --- |
| 1 |  [The Data](#the-data) |
| 2 |  [Determining Image Likeness With Vector Based Methods](#determining image likeness with vector based methods) |
| 3 |  [Convolutional Neural Networks](#convolutional-nerual-networks) |


## The Data

15000 locations in Colorado, with google street view images facing North, South, East, and West (60000 total images). The gif below shows all of the locations plotted by Latitude/Longitude/Elevation. The structure of Colorado is quickly apparent; mountains to the West, a high density of roads along the Boulder-Denver-Colorado Springs corridor, and plains to the East.

![Image](/images_for_project_overview/data_animation.gif)

Below are a handful of examples of what the streetview images look like for a given location. The terrain (mountains, plains, etc.) and content (houses, trees, road type, cars, etc.) give clues as to the location in Colorado where each of these images were taken. These hints are easy for humans to pick up on, but difficult for computers.

![Image](/images_for_project_overview/pano_df_idx_5.png)
![Image](/images_for_project_overview/pano_df_idx_25.png)
![Image](/images_for_project_overview/pano_df_idx_340.png)
![Image](/images_for_project_overview/pano_df_idx_3100.png)

## Determining Image Likeness With Vector Based Methods

Describing features in images is possible with a multitude of vector based methods. I explored the use of HSV Color Histograms, Histograms of Oriented Gradients (HOG; Dalal, N. & Triggs, B., 2005), and Speeded-Up Robust Features (SURF; Bay, H., Tuytelaars, T., & Van Gool, L., 2006) for this project, but it has been quite some time since these methods were considered cutting edge, and each has significant limitations. CNNs mimic and surpass the abilities of these methods, but it was worth trying them out as a baseline. More on my implementation of CNN later. 

### HSV Color Histograms

Using a 3D color histogram allows us to describe the color density of various colors in our image. Rather than an RGB (red, green, blue) histogram, I analyzed the color distribution in HSV (hue, saturation, value) space, which represents colors in a way more similar to how humans perceive colors. The RGB color space is represented by a cube (linear additions and subtractions of each channel to form the output), whereas the HSV color space takes RGB intensities and maps them onto a cylinder. See images below. 

![Image](/images_for_project_overview/RGB_cube.png) ![Image](/images_for_project_overview/HSV_cylinder.png)

The number of pixels that fall into a each HSV bin give a quantitative way of describing the color distribution of an image without needing to look at every pixel. A 960 component vector is created by sampling 4 regions in each image, with 8 hue bins, 8 saturation bins, and 3 value bins, and we are able to quantify the likeness between two (normalized) vectors using Euclidean distance. It is significantly easier to compare a vector of this length than one with 768,000 elements, which is what it would be if we were comparing each pixel between images (640pix by 400 pix by 3 colors). This process doesn't sacrifice much detail. In fact, it helps generalize the process, as binning the pixel intensities for each region allows us to compare the color distribution of a given *region* rather than by individual *pixels* between images. Two pictures now need not be nearly identical in order to have a high likeness; rather, only the color distribution of each region needs to be similar. 

![Image](/images_for_project_overview/HSV_greener_usedincomplat_40.35530,long_-103.434_W .jpeg) ![Image](/images_for_project_overview/HSV_redder_usedincomp_lat_39.47983,long_-104.558_S.jpeg)

Pixels from the image on the left (circles) and on the right (diamonds) are plotted below in HSV color space. This isn't exactly the HSV histogram described above (for which the bin counts would be difficult to visualize), but it gives us a sense of the color distribution in HSV space. The sky portion of each image is very similar, filling out the blue region of the HSV space, but the green and red ground colors occupy distinctly different regions in the cylinder. 

![Image](/images_for_project_overview/HSV_image_comparison.gif)

Taking the images with the smallest euclidean distance between their color histogram vectors gives exciting results for determining where in Colorado a series of North, East, South, and West images was taken, as seen below. In each case presented, the first panorama acts as the 'search' set of images, and the second set are the most similar set of images found in the database. Often, the most similar set of images is remarkably close in location to the search images, as can be seen in the figure that follows the panoramas. This figure displays the true location of the 'search' panorama, the best guess based on most similar image in the database, and the rest of the nearest 10 locations.

![Image](/images_for_project_overview/pano_likeness_mountains.png)
![Image](/images_for_project_overview/pano_likeness_highway.png) 
![Image](/images_for_project_overview/pano_likeness_grasslands.png) 
![Image](/images_for_project_overview/pano_likeness_all_3.png) 

### Histograms of Oriented Gradients (HOG)
We can get a sense of the texture and simple features that make up an image by using an implementation of HOG, which calcuates the intensity gradient across a specified pixel region. The strength and orientation of these gradients can then be compared between images to quantify their similarity. Below is an example of HOG in action. 
![Image](/images_for_project_overview/raw_img_vs_hog_img_rotate.png)

### Speeded-Up Robust Features (SURF)
SURF also ascertains features in images by looking at intensity gradients. However, rather than looking at the entire image, the algorithm looks for regions of the highest intensity gradients, their scale, and their orientation. These regions are marked and vectorized as features, which can then be compared between images. In the examples below, each circle denotes a feature, with the line from the center denoting its orientation.
![Image](/images_for_project_overview/surf_examples.png)

### Pros and Cons of Vector Based Methods
Vector based methods can work remarkably well at matching images within a dataset. However, there are a series of stipulations to fulfill this *can*. There are enough images in this dataset that it is not uncommon for there to be pairs (or more) of images taken with the same google street view car on the same road during the same season at the same time of day. That's a lot of sames. Without these conditions, this technique simply wouldn't work. Change any one of these things and the similarity of the images, as determined by the computer, would plummet. 

Additionally, vector based methods are not translationally invariant at their core. If two otherwise quite similar images were shifted from one another, the computer would no longer see them as being so similar. While possible to account for transformations of various kinds (rotation, translation, reflection, scaling, etc.) when comparing images, this would drive the computational requirements through the roof for a dataset of this size. It simply isn't practical when you need to compare your 'search' image to thousands of other images. 

## Convolutional Neural Networks 
Description coming soon!

A teaser is below of how well my CNN is doing. The images on the left act as the 'search' images and come from the shaded county. The colors correspond to the predicted probabilities (according to the CNN) of the images coming from each of the other counties in Colorado.
![Image](/images_for_project_overview/county_model_v1.0_Denver_idx_5290.png)
![Image](/images_for_project_overview/county_model_v1.0_Weld_idx_286.png)
![Image](/images_for_project_overview/county_model_v1.0_Summit_idx_2072.png)





