# Examples of barcode detection #

We provide several sample images and show the detected barcodes outlined in red. The results show acceptable performance. All barcodes were detected with the same algorithm and tuning parameters.

http://jjil.googlecode.com/files/detect1.JPG
http://jjil.googlecode.com/files/detect2.JPG
http://jjil.googlecode.com/files/detect3.JPG

These are relatively simple images for the barcode detector. The rest of the image is fairly bland, with little horizontal contrast. Only the text on either side of the barcode can potentially be confused with the barcode itself.

http://jjil.googlecode.com/files/detect4.JPG
http://jjil.googlecode.com/files/detect5.JPG

Here the barcode detector worked in spite of the label text above the barcode, which also has high horizontal contrast. This type of image is what led to the introduction of the rectangle height/width ratio test. Without this test, it was too difficult to tune the horizontal/vertical contrast measure to detect the barcode in these images while also detecting it in the others.

http://jjil.googlecode.com/files/detect6.JPG

Here the barcode detector found the correct general location of the barcode but did not detect its horizontal extent properly, probably due to the wide light bar towards the right end of the barcode.

http://jjil.googlecode.com/files/detect7.JPG

Another relatively simple image. This image was created artificially by pasting a generated image of a barcode into another generated image.