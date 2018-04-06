# jjil
JJIL is a Java image processing library.
It includes an image processing architecture and over 60 routines for various image processing tasks.

JJIL is particularly targeted towards mobile applications.
It includes interfaces so images can be converted to and from native formats for J2ME, Android, and J2SE.

JJIL includes some sample applications for face detection and EAN-13 (including UPC) barcode reading.
The barcode reader requires high resolution images (currently beyond a typical cameraphone, but perhaps not Android) but the face detection code works well with any cameraphone.
It can isolate any reasonable frontal view of a face in a few seconds.
