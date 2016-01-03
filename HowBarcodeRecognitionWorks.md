# How Barcode Recognition Works #

Barcode recognition can be broken down into two steps:
  * Barcode detection and localization: is there a barcode in the image, and where is it?

  * Barcode reading: given that a certain portion of the image contains a barcode, what are the contents of that barcode?

The first step can be eliminated if we assume the user positioned the camera so that the barcode falls into a set position in the image. However, because cellphone cameras have limited depth of field and generally do not support a short focal distance, we will require the user to position the cellphone camera far enough away from the barcode so that it is in focus. This means that the barcode will be a relatively small portion of the image. Requiring the user to position the camera so that the barcode fell into a small rectangle when the image was captured would make the barcode reader hard to use. It is better to give the user more freedom, and use software to find the barcode.

See [FindingTheBarcode](FindingTheBarcode.md) for a description of the barcode detection algorithm, [ReadingTheBarcode](ReadingTheBarcode.md) for a top-level description of the barcode reading algorithm, and [ParityAndCheckDigitTests](ParityAndCheckDigitTests.md) for a description of how the parity and check-digit tests are implemented.

Some sample images with the detected barcode outlined in them are available at [ExamplesOfBarcodeDetection](ExamplesOfBarcodeDetection.md).