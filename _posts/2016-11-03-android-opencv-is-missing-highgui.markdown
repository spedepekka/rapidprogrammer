---
layout: post
title: "Android OpenCV is missing Highgui"
date: "2016-11-03 12:58:46 +0200"
permalink: /:title
---

Many of the samples out there tell to use Highgui module to read image with OpenCV. There is no Highgui module in OpenCV 3.0+, at least not in Java/Android. I found out this from [http://stackoverflow.com/a/25943085/5749443](http://stackoverflow.com/a/25943085/5749443)

## Migration from OpenCV 2.X to 3.X (Java)

Old

    Highgui.imread(fileName, Highgui.CV_LOAD_IMAGE_GRAYSCALE)
    Highgui.imread(fileName)

New

    Imgcodecs.imread(fileName, Imgcodecs.CV_LOAD_IMAGE_GRAYSCALE)
    Imgcodecs.imread(fileName)

Most of the Core-stuff is at Imgproc. Remember to check videoio module as well.
