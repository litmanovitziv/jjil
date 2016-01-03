JJIL is a Java image processing library. It includes an image processing architecture and over 60 routines for various image processing tasks.


JJIL is particularly targeted towards mobile applications. It includes interfaces so images can be converted to and from native formats for J2ME, Android, and J2SE.


JJIL includes some sample applications for face detection and EAN-13 (including UPC) barcode reading. The barcode reader requires high resolution images (currently beyond a typical cameraphone, but perhaps not Android) but the face detection code works well with any cameraphone. It can isolate any reasonable frontal view of a face in a few seconds.

**Update, 12-22-08**: I uploaded a new version of the JJIL Android library which allows images to be converted from Android bitmaps into JJIL RgbImages. I've also updated
the FaceDetect-Android application so it is works for the current Android SDK (and
runs on the T-Mobile G1). It is a simple face detector. Capture an image and it draws
a green box around the face, or a big red X if it doesn't find anything. This is, unfortunately, redundant with the G1 since it comes with face detection code built in but the code is perhaps still useful since the Haar classifier used for face detection can also be targeted to other classifiers, and in any case face detection is cool and it's good to have more than one. You will have to get FaceDetect-Android from the source tree to get the code, I couldn't figure out how to upload an app you can run on your phone from here (without signing issues, etc.)

**Update, 10-16-2008**: I added many classes to JJIL and made the naming more consistent.
This is a major update which will break existing code, please be cautious. New classes
include code for manipulating color images, doing various kinds of warps, filling
polygons, and masking images. Classes are now named consistently: a pipeline class which takes a Gray8Image always starts with 'Gray8', so it is Gray8Abs, not GrayAbs.

**Update, 05-15-2008**: I divided JJIL into a core library, still called JJIL, and two architecture-dependent libraries, JJIL-J2ME and JJIL-J2SE. This affected the source tree structure and the downloads. You will have to link with JJIL and JJIL-J2ME if you are
building J2ME-dependent code, and JJIL + JJIL-J2SE if you are building for J2SE.
