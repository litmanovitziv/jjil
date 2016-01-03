# Introduction #

JJIL consists of an image processing architecture and a significant library of well-known image processing routines. The library is freely available as open source under the Lesser GPL. It is designed for efficient (both in space and time) use on cellphones implementing Mobile Independent Device Profile 2.0 (MIDP 2.0) and Connected Limited Device Configuration 1.0 (CLDC 1.0), which is a widely-supported platform. These profiles impose certain significant restrictions on the library: in particular, they do not support floating-point computation, and certain useful features of the Java language, such as generic types, are not supported.

## Image ##

There are two core concepts in JJIL, defined in jjil.core: Image and PipelineStage. An Image is an object that stores image data and which has a definite width and height. Images also support access to their data as an array. There are a number of different types of images: gray (8-bit, 16-bit, and 32-bit); color (8-bit pixel), complex (32-bit pixel). There are also more specialized images, such as a sub-image type for taking a portion of an input image and keeping track of the location of that portion.

All Image types support these methods:

> The constructor `Image(int cWidth, int cHeight)` creates a new image of the given width and height.

> `Image clone()`: returns a “deep” copy of the image, i.e., one that actually creates a copy of the image pixels. Note that images processed in pipeline stages are usually passed using a “shallow” copy, allowing reuse of their pixels, for efficiency in space.

> `int getHeight()`: returns the image height.

> `int getWidth()`: returns the image width.

Image types also support a method for accessing their pixels. In order to make this as efficient as possible the image data can be accessed directly as a one-dimensional array. This method is called getData(). For example, Gray8Image (an image supporting signed 8-bit image pixels) implements this method:

> `byte[] getData()`

## PipelineStage ##

A PipelineStage is a single image processing algorithm. It takes a single image as a parameter and produces another image as output. Other parameters must be supplied through the constructor or auxiliary methods. The input and output types of PipelineStage are both Image; this makes it easier to compose multiple stages into a sequence (especially in the absence of generic types) but makes it necessary to check parameter types at runtime.

A PipelineStage must implement this method:

> `void push(Image imageIn)`

It may also implement these methods, or it can use the default implementations:

> `boolean isEmpty()`

> `Image getFront()`

The semantics of these methods are:

> `push(Image)`: takes an Image as input. After verifying that it is of the right type (for example, Gray8Image or RgbImage), do whatever processing is required for this input. The output, if any, is saved so it can be retrieved using Front() and the presence of an output, if there is one, is set so it can be retrieved using Empty().

> `isEmpty()`: returns a boolean value indicating whether or not an output is available from the PipelineStage. Each pipeline stage is allowed to return zero, one, or any other number of outputs given an input. The user of the PipelineStage is required to test for the presence of an output using Empty() before attempting to retrieve it.

> `getFront()`: returns the next Image resulting from processing an input using Push(). It is an error (which will throw IllegalStateException) to attempt to retrieve an image without testing for its presence with Empty(). When Front() is called the current image is “popped” from the PipelineStage’s internal storage; calling Front() again will return a new image (if there is one; otherwise Empty() will return true).

In addition to these public methods, PipelineStage implements a protected method to help the implementer of PipelineStage-derived classes. This is

> `setOutput(Image imageResult)`

This method is used in the common case where a Push() operation returns a single Image. The implementer of push() calls setOutput(imageResult) to set the output of the Push operation to imageResult, and then the default implementations of isEmpty() and getFront() will correctly provide the image to the caller.

A complete example (from jjil.algorithm) of a simple PipelineStage is shown below. It implements a conversion operation, converting a signed byte image (a Gray8Image) into a 32-bit image (a Gray32Image):
```
    public class Gray82Gray32 extends PipelineStage {
        /** Creates a new instance of Gray82Gray32 */
        public Gray82Gray32() {
        }

        /** Converts an 8-bit gray image into a 32-bit image by replicating
         * changing the data range of the bytes from -128->127 to 0->255.
         *
         * @param image the input image.
         * @throws IllegalArgumentException if the input is not a
         * Gray8Image
         */
        public void push(Image image) throws IllegalArgumentException {
            if (!(image instanceof Gray8Image)) {
                throw new IllegalArgumentException(image.toString() + "" +
                    " should be a Gray8Image, but isn't");
            }
            Gray8Image gray = (Gray8Image) image;
            byte[] grayData = gray.getData();
            Gray32Image gray32 = new Gray32Image(image.getWidth(), image.getHeight());
            int[] gray32Data = gray32.getData();
            for (int i=0; i<gray.getWidth() * gray.getHeight(); i++) {
                /* Convert from signed byte value to unsigned byte for
                 * storage in the 32-bit image.
                 */
               int grayUnsigned = ((int)grayData[i]) - Byte.MIN_VALUE;
               /* Assign 32-bit output */
               gray32Data[i] = grayUnsigned;
            }
            super.setOutput(gray32);
        }
    }
```
This PipelineStage implements only the push() method. It relies on the default implementations of isEmpty() and getFront().

The first step in push() is to verify that the argument is of the right type. This test has to be done at runtime because all PipelineStage’s take Image parameters:
```
    if (!(image instanceof Gray8Image)) {
        throw new IllegalArgumentException(image.toString() + "" +
            " should be a Gray8Image, but isn't");
    }
```
After this test we can safely get a pointer to the input parameter as a Gray8Image:
```
    Gray8Image gray = (Gray8Image) image;
```
Having the Gray8Image pointer gives us the ability to access the data (pixels) in the image:
```
    byte[] grayData = gray.getData();
```
We will need an output image to store the result. If the output was also a Gray8Image the normal thing to do would be to reuse the input (to save on memory, PipelineStage’s are allowed to modify their input. Callers should not assume the input will not be modified and must use Clone() to make a copy of the input if they need to keep the original data.) But this output is a Gray32Image, not a Gray8Image. So we must create a new image to hold the result:
```
    Gray32Image gray32 = new Gray32Image(image.getWidth(), image.getHeight());
```
We can get a pointer to the output data just as we did with the input image:
```
    int[] gray32Data = gray32.getData();
```
Now we will set the output pixels. The loop used iterates over all pixels by treating them as one large array. This is simpler than doing the arithmetic needed for a two-dimensional array:
```
    for (int i=0; i<gray.getWidth() * gray.getHeight(); i++) {
```
The actual conversion of an 8-bit signed value to a 32-bit integer adds an offset so the minimum value in the signed pixel (i.e., -128) maps to 0. This is a matter of convenience. Some image processing operations treat 0 differently from all other pixel values. It simplifies some algorithms to map that special value to and from the minimum byte value:
```
    int grayUnsigned = ((int)grayData[i]) - Byte.MIN_VALUE;
    /* Assign 32-bit output */
    gray32Data[i] = grayUnsigned;
```
Note that Byte.MIN\_VALUE = -128 so the subtraction in the first statement above actually adds 128 to the signed byte value after it is converted to integer.

The final step in the algorithm is to provide the output to the caller. This is done using PipelineStage’s protected setOutput method:
```
    super.setOutput(gray32);
```
This class can be used as follows to convert an 8-bit signed image to a 32-bit integer image:
```
    Gray8Image imGray8 = new Gray8Image(cWidth, cHeight);
    /* ... initialize imGray8... */
    Gray82Gray32 g8232 = new Gray82Gray32();
    g8232.push(imGray8);
    if (g8232.isEmpty()) {
        /* error, this should never happen */
    }
    Image imResult = g8232.getFront();
    if (!(imResult instanceof Gray32Image)) {
        /* error, Gray82Gray32 returned wrong type */
    }
    Image imGray32 = (Gray32Image) imResult;
    /* ... use imGray32 ... */
```
Once you are familiar with image processing algorithms you will want to combine them using [Sequences and Ladders](SequencesAndLadders.md) to make image processing pipelines.