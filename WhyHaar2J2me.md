# Introduction #

The Open Computer Vision library ([OpenCV](http://tech.groups.yahoo.com/group/OpenCV/)) provides code for training a 2-dimensional image recognizer based on a cascade of Haar feature detection. Once you have trained a Haar filter cascade you can use JJIL to implement a feature detector in J2ME. An example of this is FaceDetect, which uses the example face detector filter provided with OpenCV to detect faces.

Haar filter cascades work by specifying a large number of parameters. These parameters must be transferred to a mobile device, such as a cellphone. Unfortunately, the file size of useful Haar filters (such as those supplied with OpenCV) is too large to conveniently fit in the memory of some cellphones. Moreover, the Haar filter descriptions produced by the OpenCV use floating point, which is not implemented in MIDP 1.0.

Haar2J2me provides a workaround for this problem. It reads a file produced by OpenCV and produces a much smaller file with the same parameters, converting and scaling the floating point values so they can be read by the DetectHaar constructor. Once the file has been converted it can be supplied as a resource to your JJIL project and passed to DetectHaar to construct the Haar cascade.

**Note:** currently only stump-based Haar classifiers are converted. DetectHaar works only for these type classifiers. OpenCV can be used to generate different types of Haar classifiers, so for use with JJIL you should only generate stump-based classifiers.


# Details #

The source code for Haar2J2me is available [here](http://jjil.googlecode.com/files/Haar2J2me.zip). You must download and install the OpenCV in order to compile it. Once you've done this, you can run Haar2J2me with these parameters to convert a stump-based Haar cascade:
  * -data 

<classifier\_directory>


  * -w sample\_width
  * -h sample\_height
The 

<classifier\_directory>

 parameter is the name of the directory where the Haar cascade exists. The sample size is the size of the image to be processed by the Haar cascade.