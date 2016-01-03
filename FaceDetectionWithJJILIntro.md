# Why Face Detection? #
_Jon A. Webb, Ph.D._

Modern cellphones have an integrated camera, display, multiple communications interfaces, and processing power equal, at least, to a mid-1990’s PC. They are also more widely distributed (in terms of absolute numbers, range of users, and geographically) than any other class of computers. This makes them an attractive development platform for many image processing and computer vision algorithms.

One such algorithm is face detection. Face detection is an important early step in many computer vision systems, including:

> Video communication. Faces can be detected and then compressed at a lower compression rate than the background, giving a higher-quality user experience with fixed bandwidth.

> Super video compression. After detecting a face the face can be matched with several pre-recorded faces of the speaker and then the chosen face index transmitted and displayed, resulting in an acceptable video experience using almost no bandwidth.

> Face recognition. Once a face is detected, the face image can be compared with a database of known faces, and an appropriate response can be made when a face is recognized.

> Video surveillance. Detected faces can be tracked in order to determine when a person has entered an area of interest.

> Responsive user interface. A device may execute some preprogrammed behavior when a face is detected.

> Video-augmented speech understanding. Speech understanding can be made easier by detecting and then tracking face movements to determine which of several similar-sounding phonemes was actually being pronounced.

> Intelligent autofocus. Once a face is detected, a camera’s focus can be adjusted to bring the face into better focus.

Not all of these systems are appropriate for cellphones. But they show the importance of face detection. Moreover, the face detection algorithm implemented here can be used to detect any other object for which a Haar classifier cascade is available. Haar cascades encode a series of detection decisions to classify an object as belonging or not belonging to a training set. Routines in the Open Computer Vision Library (http://opencvlibrary.sourceforge.net/) can be used to create Haar cascades for virtually any object for which sufficient training data is available.

In order to provide a consistent framework for image processing and computer vision on cellphones, I developed the JJIL (Jon’s Java Imaging Library), a Java library incorporating an image processing architecture and a significant number of well-known image processing algorithms targeted towards mobile platforms. This paper serves as an [introduction to JJIL](IntroductionToJJIL.md) using face detection as an example.