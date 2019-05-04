---
title: Android Camera2 API Pipeline Manipulation
modified: 
categories: code
excerpt: Let's Take Some Picture!
tags:
- Android
- Camera
- OpenCV
- Computer Vision
- White Balance
date: 2016-08-27T08:00:00.000+00:00
header:
  overlay_image: "/assets/images/blog/2016-08-27-android-camera2-api-pipeline-app/hero.jpg"
  overlay_filter: 0.5
  show_overlay_title: true

---
Intro
-----

Ever since the birth of digital cameras, the ability to edit its image
processing pipeline has been held exclusively by the camera manufacturers.
Google is trying to change the game with the release of the Camera 2 API for
Android. This new program interface will enable developers to input functions
into the digital imaging pipeline (Figure 1) to customize and edit features.
Just like how we have phones which are 100 million times faster than the Apollo
11’s command module, the digital imaging pipeline can be altered on our phones,
once exclusively done by bulky and expensive hardware!

I made an application which runs a user customizable pipeline based on Hakki Can
Karaimer’s publication[1] (https://karaimer.github.io/camera-pipeline/). The
main objective of this project was to normalize (calibrate) different cameras by
calculating the optimal RGGB Channel Vector. Proprietary pipelines built by
different phone manufacturers often produced different images under identical
scenes, but by calibrating the pipeline, I was able to produce identical images.

![](/assets/images/blog/2016-08-27-android-camera2-api-pipeline-app/media/7e8d87d13c6ea7c1300e0c6a9c360cb6.png)

Figure 1 Karaimer H.C., Brown M.S. (2016) “A Software Platform for Manipulating
the Camera Imaging Pipeline”, European Conference on Computer Vision (ECCV\`16),
Oct 2016 (https://karaimer.github.io/camera-pipeline/)

Digital Imaging Pipeline
------------------------

As represented in Figure 1, the Digital Imaging Pipeline is split into 12
different steps; each step represents a function applied to the RAW image. I
focused on step 1, 2, 4, and 6 for my app.

### Reading Raw Image

After raw data is taken in by the camera sensor, the electronic signals are
converted into a grayscale image with a Bayer pattern.

### Black Light Subtraction

Black light subtraction is a method of brightening an image by reducing the
intensity of the black level. A black level is defined as the level of
brightness at the darkest part of the image.

### Demosaicing

Usually, in the form of Red Green Green Blue, the Bayer pattern must go through
a process called demosaicing before being processed further. This is represented
by Stage 4 of the pipeline and requires the green pixels in a four by four
matrix to be measured and averaged out.

I used OpenCV to accomplish this.

![](/assets/images/blog/2016-08-27-android-camera2-api-pipeline-app/media/dcdf0db1069713d9022b811534a3d21d.png)

Figure 2 Use this image to show demosaicing [http://www.ok.sc.e.titech.ac.jp/\~ymonno/researches.html](http://www.ok.sc.e.titech.ac.jp/\~ymonno/researches.html)

### White Balancing

The user selects the white area and the app calculates the RGGB Vector value
required to make the selected region white. This value is applied to the entire
RAW image to white balance. This process forces the selected region to have
identical Red, Green, and Blue pixels values.

![](/assets/images/blog/2016-08-27-android-camera2-api-pipeline-app/media/7c5c7e3f3d7b851ddde7ec1d0bc2b756.png)

Figure 3 Raw Image with gamma correction (Left/Stage 1), Black Light
Substraction (Middle/Stage 2), White Balance (Right/Stage 6)

### Other Abilities

-   Auto Focus

-   Manual Focus

-   Preset White Balance Profiles

-   Manual White Balance (User selects white area)

-   Preset Scene Modes

-   Raw capture

-   Manual ISO

-   Manual Shutter Speed

-   Intervalometer (Supports RAW)

-   Manual Colour Transformation Matrix

Applications
------------

**Computer Vision:** This default process isn’t always preferred for uses in
computer vision or medical imaging. It is particularly important in computer
vision where it is extremely crucial for objects to be recognized, identified,
and properly labelled. An image ran through a typical pipeline can be mildly
useful for computer vision, but with pipeline manipulation, the chances of
recognition are greatly increased. For example, an object that blends in with
the environment can be recognized easier if the Tone Curve Application (Step 10)
of the pipeline can be altered to contrast the differences.

**Medical Imaging:** Along the same lines of computer vision, medical imaging
can benefit greatly from pattern recognition. According to “Spectral Response of
Silicon Image Sensors” by Arnaud Darmont April 2009, CMOS sensors typically
found in our phones can actually see higher and lower wavelengths than the human
eyes. By omitting or altering the typical imaging pipeline, we may be able to
use various silicon-sensitive wavelengths to diagnose illnesses.

**Aesthetic Photography:** Post-shoot photograph manipulation has been around
for years but unfortunately it has been accessible to very few people. Many
applications nowadays are able to manipulate photos by adding filters or
mechanisms to make them look more appealing or by editing RAW files in
post-shoot. As it turns out, the most popular filters occur naturally in nature
and can be easily found by simply removing or altering a step in the pipeline.
For example, Natural Vignetting; In step 3 of the pipeline, light (or the lack
thereof) is automatically compensated for through lens correction. Due to
smartphones having a low aperture, this step typically adds exposure to the
edges of the photo to match the exposure in the centre. Unknowingly to many of
the people posting Instagram pictures, the vignette does not require post-shoot
alterations, it only needs to be omitted by the pipeline.

Acknowledgements
---------------

This project would have been impossible without the expert knowledge of Michael
S. Brown, Hakki Can Karaimer, and Abdelrahman Kamel.

I would also like to thank the Lassonde School of Engineering at York University
by letting me present this project at the Lassonde Research Conference with the
Lassonde Undergraduate Research Award.