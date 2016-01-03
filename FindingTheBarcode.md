# Finding the barcode #

An important requirement for the barcode detection and localization step is that it be fast. The images we are targeting are large (an SXGA image is 1.3 MPixel) and the processor speed on a cellphone is limited. As a result the number of processing steps that can be applied per pixel is limited.

In order to detect the barcode we will take advantage of the high horizontal contrast in the barcode. A simple template for detection of regions of high contrast is:

  1. Measure horizontal and vertical contrast of the image.
  1. Threshold to select regions of high horizontal (across the bars) contrast but low vertical (along the bars) contrast.
  1. Perform connected components analysis.
  1. Extract largest region and determine its bounding box.

For speed we will perform horizontal contrast detection and extraction on a reduced size image. High horizontal contrast exists at multiple scales in an EAN-13 barcode due to the varying width of the barcode stripes, and low vertical contrast occurs at all scales. In normal image reduction we average pixels in order to avoid aliasing. But in this case, we will simply choose one of the pixels in the larger image and assign it to the pixel in the reduced image. This will create aliasing, but the barcode will still have high horizontal and low vertical contrast because the aliasing this step introduces will not reduce horizontal contrast – in fact, the aliasing will probably increase the contrast of the reduced image, making the barcode easier to detect, while vertical contrast will be unaffected by the aliasing.

Experimentation with this algorithm showed that it failed occasionally in areas where there was text which, like barcodes, had high horizontal contrast. The use of both horizontal and vertical contrast was not enough. An additional test on the expected shape of the barcode (a rectangle with a height/width ratio of 3:4) gave significantly improved performance.

We therefore have this series of steps to detect the barcode:

  1. Reduce in size (without filtering to reduce anti-aliasing) and convert to grayscale.
  1. Measure horizontal and vertical contrast.
  1. Compute a measure giving positive weight to the horizontal contrast and negative weight to the vertical contrast.
  1. Threshold.
  1. Perform connected components analysis.
  1. Extract the largest region with a roughly 3:4 height/width ratio and determine its bounding box.

These steps are implemented in the Push method of the DetectBarcode class of the Barcode library in JJIL. We will describe this method in more detail later.

See [ExamplesOfBarcodeDetection](ExamplesOfBarcodeDetection.md) for examples. If you want a more detailed explanation of the barcode detection algorithm you can find it at [DetectingBarcodeDetail](DetectingBarcodeDetail.md).

After the barcode is detected, it must be read. See [ReadingTheBarcode](ReadingTheBarcode.md) for information on how this is done.