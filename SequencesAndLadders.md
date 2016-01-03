# Sequences and Ladders #

A single image processing algorithm is fine, but real image processing algorithms consist of multiple steps. JJIL supports the construction of multiple-step image processing algorithms through two classes (both in jjil.core): Sequence and Ladder.

## Sequence ##

A Sequence is just what its name implies, a sequence of image processing algorithms, in other words, a sequence of PipelineStages. Sequence is itself a PipelineStage. So Images can be passed through a series of image processing algorithms simply by constructing the Sequence, and then using Push() on the Sequence.

An example is shown below. This sequence (from a barcode reading system) converts a color image to gray by selecting the green component, crops the gray image, and then applies a horizontal Canny edge detection operation to the result:
```
    Sequence seq = new Sequence();
    seq.Add(new RgbSelect2Gray(RgbSelect2Gray.GREEN));
    seq.Add(new GrayCrop(dTopLeftX, dTopLeftY, cWidth, cHeight));
    CannyHoriz canny = new CannyHoriz(cCannyWidth);
    seq.Add(canny);
    seq.Push(imageInput.Clone());
    if (seq.Empty()) {
        /* error -- no output from Canny */
    }
    Image imageResult = seq.Front();
```
Once a Sequence is constructed, it can be used over and over to process images. And the logic in the Sequence class’s implementation of Push() handles Empty() and Front() properly so that if a PipelineStage produces more than one output, each output will be passed to later stages in the Sequence, so that a Sequence can produce as many outputs as are provided by the PipelineStages it is made from.

## Ladder ##

Some image processing pipelines are more complex than a simple linear order: they combine multiple images to produce a result. Ladder provides a simple mechanism for combining two PipelineStages (which may, of course, be Sequences) into a new PipelineStage. It takes the two PipelineStage objects as well as a special class derived from Ladder.Join. It is constructed as follows:

  1. `Ladder(PipelineStage pipeFirst, PipelineStage pipeSecond, Ladder.Join join)`

A subclass of Ladder.Join must implement this method:

> 2. `Image DoJoin(Image imageFirst, Image imageSecond)`

This method takes two images as parameters and combines them to produce a single image as an output.

Ladder is itself a PipelineStage. Its Push method works as shown below:
```
               --- First PipelineStage ---
              /                           \
Input Image   : split (Clone)             : DoJoin --- Output Image
              \                           /
               --- Second PipelineStage --
```
As the diagram illustrates, the input image is first copied (using Clone) so that each PipelineStage gets its own copy. This way, one PipelineStage can freely alter the image data without affecting the other. After processing the resulting Image objects are combined using the Join operation.

The two Sequences do not need to be the same length. One can do a series of operations on its input, while the other might do something much simpler. Ladder handles Empty() and Push() properly in manipulating the outputs of its PipelineStages, but it does require that an output Image be available from one PipelineStage whenever one is available from the other PipelineStage, so the number of images resulting from a Push operation on each pipeline must be the same. Otherwise it could not call DoJoin at the appropriate time.

A simple example of the use of Ladder is using Gaussian blurring to sharpen vertical features in an image. The idea behind this technique is to subtract a horizontally-blurred image from itself. This will tend to sharpen features since out-of-focus areas of the image will not change much in the blurred image (and so be set to a small value in the output) while sharp areas of the image will be blurred significantly in the blurred image (so their difference from the original will be large).

The code for this Ladder uses the GraySub class from jjil.algorithm, which implements DoJoin by taking the difference of its two Gray8Image inputs. It also makes use of GaussHoriz, which performs a horizontal blur operation on its Gray8Image, and Copy, which does a shallow copy (i.e., not creating new pixels; it is not necessary to replicate the pixels because Ladder’s Push operation does that) of the input to the output:
```
    /* Create Copy PipelineStage */
    Copy c = new Copy();
    /* Create Gauss blur PipelineStage */
    GaussHoriz gh = new GaussHoriz(10);
    /* Create Join object */
    GraySub gs = new GraySub();
    /* Create Ladder */
    Ladder lad = new Ladder(c, gh, gs);
```
Images can be deblurred using this Ladder simply by
```
    lad.Push(imageIn);
    if (lad.Empty()) {
    /* error, no output */
    }
    Image imageOut = lad.Front();
```
After using this Ladder for a while, we might notice that the output tends to be pretty dark. This is because we are subtracting two images with nearly the same value. A simple way to fix this is to perform histogram equalization, using GrayHistEq from jjil.algorithm. The resulting code can be written as simply as:
```
    /* Create Ladder */
    Ladder lad = new Ladder(
        new Copy(),
        new GaussHoriz(10),
        new GraySub());
    /* Create Sequence */
    Sequence seq = new Sequence(lad);
    seq.Add(new GrayHistEq());
```
The entire sequence can be executed simply by
```
    seq.Push(imageIn);
    if (seq.Empty()) {
        /* error, no output */
    }
    Image imageOut = seq.Front();
```
Of course, once a Ladder like this is constructed, it can be used over and over to process images, just as a Sequence can be.

See [FaceDetectionHighLevel](FaceDetectionHighLevel.md) for an explanation of how Sequences and Ladders are used to implement face detection in JJIL, or [DetectingBarcodeDetail](DetectingBarcodeDetail.md) for how Sequences are used to detect barcodes.