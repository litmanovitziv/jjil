# Face Detection #

Implementing a face detection algorithm is made immeasurably simpler by the existence of the Open Computer Vision library or OpenCV (http://opencvlibrary.sourceforge.net/), an open-source library of computer vision algorithms, which includes object detection code (and trained Haar cascades for detecting faces) based on Haar classifiers.

The training and use of a Haar classifier in the OpenCV is explained in detail elsewhere (http://www.intel.com/technology/itj/2005/volume09issue02/art03_learning_vision/p04_face_detection.htm). The explanation here will focus on just those details needed to understand how face detection is implemented using JJIL.

At the highest level, a face detector based on a Haar classifier applies a series of simple feature detectors to an image of fixed size, for example, 20x20. Each feature detector is thresholded and the outputs are combined in a manner specified by the Haar cascade to yield a boolean output – the face is in the image or not.

In actual implementation, the image is processed at multiple scales and the face detector is scanned across the image so that a face of any size or position will be detected. The region where the face is detected is marked in the image. The resulting marked image may be input for further processing as described in the Introduction, or may be displayed as is.

The overall structure of the face detection algorithm as implemented using JJIL is:

For each scale, starting at the coarsest:
  1. Reduce the image to this scale.
  1. For each position in the reduced image:
    1. If a face has not previously been detected at this position:
      1. Apply the Haar classifier to the reduced image at this position.
      1. If a face is found, mark the image as having a face at this position.

(Note: this is somewhat different from the implementation in the OpenCV, which starts at the finest resolution and then aggregates detections. I believe the approach described here is simpler, more efficient, and leads to at least as good results as the approach in the OpenCV.)

The final output is an image showing the positions where a face was found. The result is shown as a masked image with the background dimmed and turned to grayscale, and the face shown in color. Note that the method described here can find multiple faces in the same image, as shown in the screenshot below:

![http://jjil.googlecode.com/files/image002.jpg](http://jjil.googlecode.com/files/image002.jpg)

Note that the use of a mask image as output makes it easy to display the result, but may require further processing for some applications, such as face tracking. Adding connected components extraction after face detection would make it possible to track faces in the image.

The JJIL implementation of face detection can be broken down into four components:

  1. [Implement Haar cascade.](ImplementingHaarCascade.md)
  1. [Capture image, preprocess, and display results.](CaptureToDisplay.md)

We will also need to address

> 3. [Converting and transferring a Haar cascade](ConvertingHaarCascade.md) to a J2ME-enabled device.