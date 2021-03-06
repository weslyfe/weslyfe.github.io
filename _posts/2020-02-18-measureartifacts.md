---
layout: post
title: A quarter and a camera ─ Measuring relative abundance of artifacts with computer vision in Python
---

<a href="https://www.digitalarchns.com/refinemesh/" target="_blank">Previously</a>, I showed how to pull some extra quality out of photogrammetric models.

This post is going to take a short dive into the world of <a href="https://en.wikipedia.org/wiki/Computer_vision" target="_blank">computer vision</a>.
You'll learn how to deploy a <a href="https://www.python.org/" target="_blank">Python</a> script using 
<a href="https://opencv.org/" target="_blank">OpenCV</a> to take measurements of many artifacts 
in a single photo. We will modify a <a href="https://www.pyimagesearch.com/2016/03/28/measuring-size-of-objects-in-an-image-with-opencv/" target="_blank">script</a> 
originally developed by Adrian Rosebrock at <a href="https://www.pyimagesearch.com/" target="_blank">PyImageSearch</a> 
to measure objects in an image. The modified script additionally calculates 2D surface area for each 
artifact and exports the measurements to a CSV file. 

<a href="https://github.com/weslyfe/weslyfe.github.io/blob/master/download/artifact-area.zip?raw=true" download>Download the script here.</a>

<h2> Why use surface area?</h2>

Artifact surface area can be used to give a measure of 
**relative abundance** of artifact types in an analysis that is 
less susceptible to error from fragmentation than a simple count. Weight, 
although a valient contender, can run into issues when comparing between 
artifacts made of materials with different densities. As good as it 
sounds, measuring the surface area of irregular objects is a task barely 
worth the effort using analogue methods. 

The first encounter I had with surface area being used in artifact analysis was 
in a <a href="https://www.tandfonline.com/doi/abs/10.1179/009346997792208168" target="_blank">1997 article 
by John Byrd and Dalford Owens</a>. Byrd and Owens use what they term *Effective Area* 
as a substitute for surface area. 

Though the measurement of *Effective Area* is one measure of surface area. The 
technique actually has the potential to be quite desctrutive as it "*drops the 
sherd through a set of nested screens*" similar to screening soils for grain 
size analysis. 

Byrd and Owens focus their attention specifically on sherds. 
However, the technique could theoretically be applied to any assemblage of 
artifacts. Given the right research question. 

It's easy to see why this technique hasn't been applied to a whole slew of artifacts 
as it has the potential of destroying or damaging fragmented remains. The intention 
of Byrd and Owens wasn't to provide *the best* solution, but to present a solution 
that gives a better measurement of **relative abundance** than a simple counts and 
in some cases, weights. They explain the reasons they believe surface area hadn't been adopted:

---
 <h3> <p style="text-align: center;"> "<i>Measuring surface area ... will not likely be adopted by archaeologists unless a relatively simple technique is developed for obtaining such data.</i>" </p></h3>
 <h4> <p style="text-align: center;"> <i>Byrd & Owens, 1997</i></p></h4>
 ---

Well I'm happy to say thanks to Adrian at <a href="https://www.pyimagesearch.com/" target="_blank">PyImageSearch</a> 
for creating his blog that allowed me to begin to adopt computer vision and machine learning 
in python for my own archaeological uses. The following tutorial brings us one step closer to 
the *simple technique* alluded to by Byrd and Owens using only **a quarter and a camera!**


<h2> Computer vision?</h2>

Computer vision is an interdisciplinary field of research devoted to “extract[ing] meaning from pixels” (<a href="https://towardsdatascience.com/computer-vision-an-introduction-bbc81743a2f7" target="_blank">Singh, 2019</a>). 

I won't spend too much time on the details of what else computer vision can do here. You can 
learn more about that at <a href="https://www.pyimagesearch.com/" target="_blank">PyImageSearch</a>. 
I will give a run down on what this particular script is doing, and how you can use it though!

I'm assuming you already have `Python 3` and `OpenCV` installed, but if you don't 
please take the time to do so now. OpenCV now installs with `pip` so you can 
install it when installing the other required modules.

<h2> The script</h2>

Okay, so here we go. You can either read along and copy each code block 
into your favorite text editor saving the file with a `.py` extension, or 
you can <a href="https://github.com/weslyfe/weslyfe.github.io/blob/master/download/artifact-area.zip?raw=true" download>download the source code here</a>. 

<h3> File directory</h3>

The source code comes as a `.zip` folder containing the `artifact-area.py` file 
and two subdirectories called `images` and `csv`.

The subfolder `images` is where you will place the image you would like to 
analyze, and `csv` is where the `.csv` file is saved to after measuring.

Inside `images` you will find a single image named `test_01.jpg`. This 
is the image used in the tutorial below. The image contains a Canadian quarter 
and a series of lithic flakes produced preparing the biface for the 
<a href="https://skfb.ly/6QrSw" target="_blank">3D model</a> I knapped, 
shown in <a href="https://www.digitalarchns.com/Masking3dModels/" target="_blank">a previous 
post</a>.

<p align="center">
<img src="/images/test_01.JPG"> 
</p>
<p align="center">
<i><small>Tutorial image.</small></i>
</p>

A few things worthy of a note about the photo:

The background heavily contrasts the colour of the material being measured.

The lighting is not great.

The quarter and its placement to the far left of the image. The quarter is used as a scale bar to convert one pixel to a measurement. 
Canadian quarters have a known diameter of 2.381cm.

The 90 degree angle of the photo. I know it is second nature for artifact photography to take the photo at 90 degrees. Here it is 
critical, as it impacts the accuracy of the resulting measurements. And I estimated. This definitely impacted the results, but the 
resulting accuracy is pretty good. A quick way to get your photos to 90 degrees without a tripod is to use a hot shoe bubble level.

You likely can't notice, but I had to turn the photo quality on my Canon EOS REBEL T5i to the lowest setting to obtain photos without 
heavy noise during processing. It seems counter-intuitive to take low quality photos, but the extra detail at the distance I was created 
a lot of false objects in the results. This does raise the possibility of shooting high resolution images from a higher vantage point to 
measure many more artifacts in a single image.


<h3> Script usage and required modules</h3>

```py
"""
 Title: Automated Rapid Artifact Surface Area Measurement from Imagery using Computer Vision
 Author: Wesley Weatherbee
 Date: February 2020
 Description:This script is intended to rapidly collect measurments 
             from multiple artifacts in a simgle image using computer vision.
 
 Modified from: https://www.pyimagesearch.com/2016/03/28/measuring-size-of-objects-in-an-image-with-opencv/
 Original Author: Adrian Rosebrock
 Date: March 28, 2016 
"""

# USAGE
# open command-line (cmd.exe) and change the directory to the 
# directory where the file is loaded using: cd [path to directory here]
# after changing the directory, enter the following code in command-line:
# 
# python artifact-area.py --image images/test_01.jpg --width 2.381

# import the necessary packages
from scipy.spatial import distance as dist
from imutils import perspective
from imutils import contours
import numpy as np
import argparse
import imutils
import cv2
import csv
import pandas as pd
```

The above codeblock imports the required modules to run the script. The script 
is run from the command-line.

1. Change the current directory to the `artifact-area` folder using `cd` followed 
by the path.

2. Use the code `python artifact-area.py --image images/test_01.jpg --width 2.381` to 
run the script. `--image` is the argument to choose which image to use, and `--width` 
is the width of your reference object to the far left of the photo in whatever units 
you choose.



<h3> Define methods used in the script</h3>

```py
# define the automatic canny edge detection method
def auto_canny(image, sigma=0.33):
	# compute the median of the single channel pixel intensities
	v = np.median(image)

	# apply automatic Canny edge detection using the computed median
	lower = int(max(0, (1.0 - sigma) * v))
	upper = int(min(255, (1.0 + sigma) * v))
	edged = cv2.Canny(image, lower, upper)

	# return the edged image
	return edged

# define midpoint method
def midpoint(ptA, ptB):
	return ((ptA[0] + ptB[0]) * 0.5, (ptA[1] + ptB[1]) * 0.5)
```

This script uses two custom methods defined above: automatic canny edge detection 
`def auto_canny` (from <a href="https://www.pyimagesearch.com/2015/04/06/zero-parameter-automatic-canny-edge-detection-with-python-and-opencv/" target="_blank">PyImageSearch</a>), and a method that calculates the midpoint between two sets of 
coordinates, `def midpoint`. The second method enables us to measure the X, and Y dimensions of 
the minimum bounding box of each object. This is different than length and width 
because they don't follow the axis running from striking platform to termination 
of the flake. Still they are important metrics for checking the accuracy of your 
measurements using the quarter.

```py
# construct the argument parser and parse the arguments
ap = argparse.ArgumentParser()
ap.add_argument("-i", "--image", required=True,
	help="path to the input image")
ap.add_argument("-w", "--width", type=float, required=True,
	help="width of the left-most object in the image (in inches)")
args = vars(ap.parse_args())
```

The argument parser is now constructed and reads the `--image` and `--width` 
arguments defined in the command-line.

<h3> Loading and preprocessing the image</h3>

Now we use OpenCV to read the image defined in the command-line argument. We 
convert it to grayscale then add a gaussian blur twice. First with a 9x9 kernel, 
then a 5x5 kernel.

```py
# load the image, convert it to grayscale, and blur it slightly, twice
image = cv2.imread(args["image"])
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
gray = cv2.GaussianBlur(gray, (9, 9), 0)
gray = cv2.GaussianBlur(gray, (5, 5), 0)

# perform edge detection, then perform a dilation + erosion to
# close gaps in between object edges
edged = auto_canny(gray)
edged = cv2.dilate(edged, None, iterations=3)
edged = cv2.erode(edged, None, iterations=2)
```

The `auto_canny` method is deployed on the blurred grayscale image. Dialation 
and erosion are run on the resulting edge map to close the boundaries. 

<h3> Finding and sorting the contours</h3>

Here closed boundary contours are identified in the image from left to right 
in the image. We define the contours as the variable `cnt` and then sort them 
from left to right. 

```py
# find contours in the edge map
cnts = cv2.findContours(edged.copy(), cv2.RETR_EXTERNAL,
	cv2.CHAIN_APPROX_SIMPLE)
cnts = imutils.grab_contours(cnts)

# sort the contours from left-to-right and initialize the
# 'pixels per metric' calibration variable
(cnts, _) = contours.sort_contours(cnts)
pixelsPerMetric = None
```

Lastly, we create a list object called `measure` to hold the values measured 
from each contour in the image.

```py
# create list object
measure = []
```

Lets visualize what we've done so far.

<p align="center">
	<img src="/images/sequence.gif"> 
</p>
<p align="center">
<i><small>Visualization of the process up to this point.</small></i>
</p>

<h3> Looping through the contours in the image</h3>

The following `for` loop analyzes each of the contours individually. The first step is 
to ignore contours with an area less than 10000 pixels.

```py
# loop over the contours individually
for c in cnts:
	# if the contour is not sufficiently large, ignore it
	if cv2.contourArea(c) < 10000:
		continue
```

The next codeblock creates a properly rotated bounding box around each contour, 
and X and Y measurements in pixels. 

```py
# compute the rotated bounding box of the contour
orig = image.copy()
box = cv2.minAreaRect(c)
box = cv2.cv.BoxPoints(box) if imutils.is_cv2() else cv2.boxPoints(box)
box = np.array(box, dtype="int")

# order the points in the contour such that they appear
# in top-left, top-right, bottom-right, and bottom-left
# order.
box = perspective.order_points(box)

# unpack the ordered bounding box, then compute the midpoint
# between the top-left and top-right coordinates, followed by
# the midpoint between bottom-left and bottom-right coordinates
(tl, tr, br, bl) = box
(tltrX, tltrY) = midpoint(tl, tr)
(blbrX, blbrY) = midpoint(bl, br)

# compute the midpoint between the top-left and top-right points,
# followed by the midpoint between the top-right and bottom-right
(tlblX, tlblY) = midpoint(tl, bl)
(trbrX, trbrY) = midpoint(tr, br)

# compute the Euclidean distance between the midpoints
dA = dist.euclidean((tltrX, tltrY), (blbrX, blbrY))
dB = dist.euclidean((tlblX, tlblY), (trbrX, trbrY))
```

Below we communicate that if `pixelsPerMetric` doesn't yet have a value, 
then the computer must be reading the first contour. The first contour will 
be the object farthest to the left of the image. The `X` dimension of this 
contour in `pixels` divided by the `--width` argument provides the value 
of `pixelsPerMetric`. This is why it is integral to have the reference to the 
left in these images.


```py
# if the pixels per metric has not been initialized, then
# compute it as the ratio of pixels to supplied metric
# (in this case, inches)
if pixelsPerMetric is None:
	pixelsPerMetric = dB / args["width"]
```

Now we calculate our measurements using our (possibly) newly defined conversion 
factor of `pixelsPerMetric`.

```py
# compute the size and surface area of the object
dimA = (dA / pixelsPerMetric)
dimB = (dB / pixelsPerMetric)
SA = ((cv2.contourArea(c) / pixelsPerMetric) / pixelsPerMetric)
```

And, lastly, the measurements are appended to the python list object `measure` 
one by one.

```py
# add measurements to list
measure.append((dimA, dimB, SA))
```

<h3> Visualizing the result</h3>

To visualize the result, we first calculate the centre of the contour to anchor our ovaying text from. 
Next, `cv2.drawContours` draws the contours in red `(0, 0, 255)`, the surface area measurement 
is displayed in square centimetres with `cv2.putText`, the image is resized to fit within a bordered 
window fully with `cv2.resize`, and the image is shown with `cv2.imshow`.

```py
# compute centre of the contour
M = cv2.moments(c)
cX = int(M["m10"] / M["m00"])
cY = int(M["m01"] / M["m00"])

# draw contours in red
cv2.drawContours(orig, [c.astype("int")], -1, (0, 0, 255), 2)

# draw the object area on the image
cv2.putText(orig, "{:.2f}sqcm".format(SA),
	(int (cX), int (cY)), cv2.FONT_HERSHEY_SIMPLEX,
		0.65, (0, 0, 0), 3)
cv2.putText(orig, "{:.2f}sqcm".format(SA),
	(int (cX), int (cY)), cv2.FONT_HERSHEY_SIMPLEX,
		0.65, (255, 255, 255), 2)

# show the output image
origS = cv2.resize(orig, (1536, 1024))
cv2.imshow("Image", origS)
cv2.waitKey(0)
```

<p align="center">
	<img src="/images/contours.gif"> 
</p>
<p align="center">
<i><small>Surface area of each flake visualized.</small></i>
</p>

<h3> Accuracy check</h3>

One huge benefit to this method aside from the obvious non-destructive nature, scalability, 
and reproducability is the opportunity to **report the accuracy of your results!**

You'll notice in the above image that the surface area of the **quarter** is `4.56sqcm`. We can 
use this value to calculate the difference between the expected surface area for a circle with a 
`diameter` of `2.381cm` and the experimental result. Using the equation A = &pi;r<sup>2</sup>, the 
expected area is `4.45sqcm`, while our experimental result is `4.56sqcm`. 

---
 <h3> <p style="text-align: center;"> <i>We achieved a difference of <b>0.11cm<sup>2</sup></b> between expected and experimental results.</i></p></h3>
---

A difference of 0.11cm<sup>2</sup> equates to a 2.41% error in our measurements using the following 
formula:

---
 <h4> <p style="text-align: center;"> <i>PercentError = ((Experimental - Expected) / 100) / Expected</i> </p></h4>
---

That 2.41% error can be inversely expressed as **97.59% accuracy!** 

I think for not shooting the photo at exactly 90-degrees to the objects, and not 
doing a camera calibration to correct for distortion in the lens, we have achieved 
fairly exciting results! 

Of course, you can always improve the results by correcting for lens distortion and angle of capture.

<h3> Saving the data to csv</h3>

After the `for` loop runs its course, the following codeblock exports the information stored 
in the `measurements` list object to a `csv` file named `Measurements.csv` in the subdirectory `csv`.

```py
# take X, Y, and surface area measurements from the list and append them to a CSV
col_titles = ('X', 'Y', 'SA')
data = pd.np.array(measure).reshape((len(measure) // 1, 3))
pd.DataFrame(data, columns=col_titles).to_csv("csv/Measurements.csv", index=False)
```

Just like that we are now able to use **a quarter and a camera**, Python, and OpenCV to 
non-destructively derive a measure of **relative abundance** of artifacts using surface area. 
Not only that, but the ability of the script to measure many more artifacts than we did 
from a single image is very enticing.

<h2> The results are in...</h2>

...and we likely all agree that this method provides an effective, non-destructive 
alternative to the technique proposed by Byrd and Owens for measuring **relative abundance** 
in artifact types from surface area.

It's important to note that this application is just the tip of the iceberg. Applications of 
python and computer vision to archaeological workflows are numerous, yet infrequently utilized. 
However, I hope that as I continue to share information like the above tutorial, we will see 
a rise in computer literacy within archaeological communities. Strategically investing in computer 
literacy will inevitably free up time to **do more archaeology**. Whatever that means to you.

Maybe these newfangled computational methods actually do provide useful tools 
for archaeologists? I'd love to see archaeologists be able to do more with less 
funding **and** time. I believe it is valuable to invest in computer literacy to achieve this.

Next post I'll be showing a useful script I wrote to help estimate the resolution of ground penetrating
radar data in different soil conditions. As usual, if you have any comments don't hesitate to send them 
along on the social media post or through email on the <a href="https://www.digitalarchns.com/about/" target="_blank">about page</a>.

<a href="https://github.com/weslyfe/weslyfe.github.io/blob/master/download/artifact-area.zip?raw=true" download>Download the script here!</a>

*Wesley Weatherbee*


References:

<div class="csl-bib-body" style="line-height: 2; margin-left: 2em; text-indent:-2em;">
  <div class="csl-entry">Byrd, J. E., &amp; Owens, D. D. (1997). A Method for Measuring Relative Abundance of Fragmented Archaeological Ceramics. <i>Journal of Field Archaeology</i>, <i>24</i>(3), 315–320. <a href="https://doi.org/10.1179/009346997792208168">https://doi.org/10.1179/009346997792208168</a></div>
  <span class="Z3988" title="url_ver=Z39.88-2004&amp;ctx_ver=Z39.88-2004&amp;rfr_id=info%3Asid%2Fzotero.org%3A2&amp;rft_id=info%3Adoi%2F10.1179%2F009346997792208168&amp;rft_val_fmt=info%3Aofi%2Ffmt%3Akev%3Amtx%3Ajournal&amp;rft.genre=article&amp;rft.atitle=A%20Method%20for%20Measuring%20Relative%20Abundance%20of%20Fragmented%20Archaeological%20Ceramics&amp;rft.jtitle=Journal%20of%20Field%20Archaeology&amp;rft.volume=24&amp;rft.issue=3&amp;rft.aufirst=John%20E.&amp;rft.aulast=Byrd&amp;rft.au=John%20E.%20Byrd&amp;rft.au=Dalford%20D.%20Owens&amp;rft.date=1997-01-01&amp;rft.pages=315-320&amp;rft.spage=315&amp;rft.epage=320&amp;rft.issn=0093-4690"></span>
  <div class="csl-entry">Rosebrock, A. (2015, April 6). Zero-parameter, automatic Canny edge detection with Python and OpenCV. <i>PyImageSearch</i>. <a href="https://www.pyimagesearch.com/2015/04/06/zero-parameter-automatic-canny-edge-detection-with-python-and-opencv/">https://www.pyimagesearch.com/2015/04/06/zero-parameter-automatic-canny-edge-detection-with-python-and-opencv/</a></div>
  <span class="Z3988" title="url_ver=Z39.88-2004&amp;ctx_ver=Z39.88-2004&amp;rfr_id=info%3Asid%2Fzotero.org%3A2&amp;rft_val_fmt=info%3Aofi%2Ffmt%3Akev%3Amtx%3Adc&amp;rft.type=blogPost&amp;rft.title=Zero-parameter%2C%20automatic%20Canny%20edge%20detection%20with%20Python%20and%20OpenCV&amp;rft.description=Don't%20tune%20your%20Canny%20edge%20detector%20parameters%20by%20hand.%20In%20this%20article%2C%20I'll%20show%20you%20my%20automatic%2C%20parameter%20free%20Canny%20edge%20detector.&amp;rft.identifier=https%3A%2F%2Fwww.pyimagesearch.com%2F2015%2F04%2F06%2Fzero-parameter-automatic-canny-edge-detection-with-python-and-opencv%2F&amp;rft.aufirst=Adrian&amp;rft.aulast=Rosebrock&amp;rft.au=Adrian%20Rosebrock&amp;rft.date=2015-04-06&amp;rft.language=en-US"></span>
  <div class="csl-entry">Rosebrock, A. (2016, March 28). Measuring size of objects in an image with OpenCV. <i>PyImageSearch</i>. <a href="https://www.pyimagesearch.com/2016/03/28/measuring-size-of-objects-in-an-image-with-opencv/">https://www.pyimagesearch.com/2016/03/28/measuring-size-of-objects-in-an-image-with-opencv/</a></div>
  <span class="Z3988" title="url_ver=Z39.88-2004&amp;ctx_ver=Z39.88-2004&amp;rfr_id=info%3Asid%2Fzotero.org%3A2&amp;rft_val_fmt=info%3Aofi%2Ffmt%3Akev%3Amtx%3Adc&amp;rft.type=blogPost&amp;rft.title=Measuring%20size%20of%20objects%20in%20an%20image%20with%20OpenCV&amp;rft.description=Today%2C%20I'll%20demonstrate%20how%20you%20can%20compute%20the%20size%20of%20objects%20in%20an%20image%20using%20OpenCV%2C%20Python%2C%20and%20computer%20vision%20%2B%20image%20processing%20techniques.&amp;rft.identifier=https%3A%2F%2Fwww.pyimagesearch.com%2F2016%2F03%2F28%2Fmeasuring-size-of-objects-in-an-image-with-opencv%2F&amp;rft.aufirst=Adrian&amp;rft.aulast=Rosebrock&amp;rft.au=Adrian%20Rosebrock&amp;rft.date=2016-03-28&amp;rft.language=en-US"></span>
  <div class="csl-entry">Singh, R. (2019, June 10). <i>Computer Vision—An Introduction</i>. Medium. <a href="https://towardsdatascience.com/computer-vision-an-introduction-bbc81743a2f7">https://towardsdatascience.com/computer-vision-an-introduction-bbc81743a2f7</a></div>
  <span class="Z3988" title="url_ver=Z39.88-2004&amp;ctx_ver=Z39.88-2004&amp;rfr_id=info%3Asid%2Fzotero.org%3A2&amp;rft_val_fmt=info%3Aofi%2Ffmt%3Akev%3Amtx%3Adc&amp;rft.type=webpage&amp;rft.title=Computer%20Vision%20%E2%80%94%20An%20Introduction&amp;rft.description=Demystifying%20the%20meaning%20behind%20pixels&amp;rft.identifier=https%3A%2F%2Ftowardsdatascience.com%2Fcomputer-vision-an-introduction-bbc81743a2f7&amp;rft.aufirst=Ranjeet&amp;rft.aulast=Singh&amp;rft.au=Ranjeet%20Singh&amp;rft.date=2019-06-10&amp;rft.language=en"></span>
</div>

