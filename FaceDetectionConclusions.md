# Conclusions #

Haar-based face detection is a highly successful application as implemented here for cellphones. The Haar cascade included in the source (which is the converted haarcascade\_frontalface\_alt.xml from the OpenCV) works on a wide variety of faces, working on any frontal view of a reasonably well-lit, reasonably vertically-oriented, face. It operates in a few seconds on a Nokia 6620, which has a modest 52 MHz Java virtual processor (according to http://www.club-java.com/TastePhone/J2ME/MIDP_Benchmark.jsp).

Below are some sample outputs, created by running the face detector on images randomly chosen from the Internet:

![http://jjil.googlecode.com/files/image004.jpg](http://jjil.googlecode.com/files/image004.jpg)
![http://jjil.googlecode.com/files/image006.jpg](http://jjil.googlecode.com/files/image006.jpg)
![http://jjil.googlecode.com/files/image008.jpg](http://jjil.googlecode.com/files/image008.jpg)
![http://jjil.googlecode.com/files/image010.jpg](http://jjil.googlecode.com/files/image010.jpg)

This application demonstrates that it is possible to do interesting image processing on currently-available cellphones. It also shows the potential as cellphones gain in processing power. The exponential increase in processing power which applied to desktop computers is also in progress for cellphones. As processing power increases, more and more sensory capabilities will be added to cellphones, since inherent limitations on size and the need for easy to use interfaces will require the cellphones themselves to become more intelligent. Already speech understanding for voice dialing is a standard feature; soon, image processing and computer vision will be integrated. JJIL provides a path for integrating existing image processing and computer vision algorithms into cellphone software by providing a consistent image processing architecture and a core set of useful algorithms.