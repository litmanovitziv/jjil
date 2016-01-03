# Implementing the Haar cascade #

A Haar cascade encodes a series of feature detection operations in a tree structure. The JJIL implementation of Haar cascades currently supports only stump-based Haar cascades, in which each decision point is pass/fail. That is, a stump-based Haar cascade works in a series of similar steps. Each step applies a group of feature detection operations to the image and sums the results. If the sum is greater than a threshold, the Haar classifier goes on to the next step. If any step fails, the image is judged not to pass – in this case, not to be a face.

The Haar cascade is implemented as several nested classes in HaarClassifierCascade (in jjil.algorithm). The lowest level feature-detection is done by HaarFeature. The results of the feature detection operations are aggregated in HaarWeakClassifierStump. A “step” in the Haar classifier cascade is implemented by HaarClassifierStump. The top-level interface is implemented by HaarClassifierStumpBase.

## HaarFeature ##

At the lowest level, the Haar cascade consists of feature detectors that are rectangular regions of the image which are summed, multiplied by a coefficient, and then added together. (Note: the current JJIL implementation does not include “tilted” features, in which the rectangular regions are rotated by 45 degrees.)

Haar features always have two or three such regions. The exact regions to be summed are specified in the Haar cascade XML file by giving the upper left coordinates of the rectangle, the rectangle width and height, and the coefficient, in that order. For example,
```
<feature>
    <rects>
        <_>3 7 14 4 -1.</_>
        <_>3 9 14 2 2.</_>
    </rects>
    <tilted>0</tilted>
</feature>
```
Specifies two rectangles: one starting at (3,7) and extending 14 pixels in width and 4 in height, which is multiplied by -1; the other starts at (3,9) and extends 14 pixels in width and 2 in height, and is multiplied by 2. It will be seen that the two rectangles overlap and form a horizontal edge detector as shown below:

| | | | | | | | | | | | | | | | | | | | |
|:|:|:|:|:|:|:|:|:|:|:|:|:|:|:|:|:|:|:|:|
| | | | | | | | | | | | | | | | | | | | |
| | | | | | | | | | | | | | | | | | | | |
| | | | | | | | | | | | | | | | | | | | |
| | | | | | | | | | | | | | | | | | | | |
| | | | | | | | | | | | | | | | | | | | |
| | | | | | | | | | | | | | | | | | | | |
| | | | -1 | -1| -1 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | | | |
| | | | -1 | -1| -1 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | | | |
| | | | +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| | | |
| | | | +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| +2-1=+1| | | |
| | | | | | | | | | | | | | | | | | | | |
| | | | | | | | | | | | | | | | | | | | |
| | | | | | | | | | | | | | | | | | | | |
| | | | | | | | | | | | | | | | | | | | |
| | | | | | | | | | | | | | | | | | | | |
| | | | | | | | | | | | | | | | | | | | |
| | | | | | | | | | | | | | | | | | | | |
| | | | | | | | | | | | | | | | | | | | |
| | | | | | | | | | | | | | | | | | | | |


The key optimization in computing these rectangular sums is to form a cumulative sum of the image from top to bottom and left to right. Then any rectangular sum can be found by two add and two subtract operations.

That is, given an input image I(i,j) we form the image
```
    S(k,l) = Sum(I(i,j), i<=k, j<=l)
```
We can then form the sum of the rectangle starting at (x,y) and extending w pixels horizontally and h pixels vertically by computing
```
    S(x+w, y+h) – S(x-1, y+h) – S(x+w, y-1) + S(x-1, y-1) (1)
```
Since JJIL uses fixed-point arithmetic, it is necessary to consider data range when computing S(k,l). The input image I(I,j) is a signed 8-bit value. The cumulative sum of an image of 2Wx2H pixels will require W+H+8 bits for accurate representation, and 2xW+2xH+8 bits for accurate calculation of operations on the image as a whole such as average. This means that the data type constrains the image size. Using 32-bit integers, we can handle image sizes up to 26x26 = 64x64. This is comfortably larger than the size used in face detection, 20x20.

Once the rectangles are summed they must be multiplied by the coefficients. The OpenCV Haar cascade uses floating-point values as coefficients (note the decimal point in the XML example), which must be converted to fixed point for use in JJIL. But examination of the Haar cascade code in the OpenCV shows that the coefficients are, in fact, always integral. They can be converted to integer simply by rounding or truncating.

One final complication in computing the Haar feature comes from a boundary condition. When a rectangle is at the top left or right side of the subimage (i.e., its x or y coordinate is 0), equation (1) refers to a cumulative sum value outside the subimage, leading to an invalid array reference. We address by creating an abstract class HaarRect and implementing different subclasses of it to compute the rectangular sums when the rectangle is entirely in the interior of the image, at the top edge, at the right edge, at the top right, and null. The null feature is used when only two of the three rectangles possible are computed as part of a HaarFeature. The result, from HaarClassifierCascade in jjil.algorithm, is shown below. The constructors and some code optimizing array address computation have been omitted:
```
    protected class HaarFeature {
        abstract class HaarRect {
            protected abstract int eval(Gray32Image i);
        }

        // HaarRectNone implements the case of a null rectangle
        class HaarRectNone extends HaarRect {
            public HaarRectNone() {
            }
            protected int eval(Gray32Image i) {
                return 0;
            }
        }

        // HaarRectAny implements the case of an interior rectangle
        class HaarRectAny extends HaarRect {
            ...
            public HaarRectAny(
                int tlx, int tly, int w, int h, int weight) {
            }
            protected int eval(Gray32Image image) {
                int data[] = image.getData();
                return weight * ( data[this.n1] + data[this.n2] -
                    data[this.n3] - data[this.n4] );
            }
        }

        // HaarRectLeft implements the case of rectangle at the left
        class HaarRectLeft extends HaarRect {
            public HaarRectLeft(int tly, int w, int h, int weight) {
            }
            protected int eval(Gray32Image image) {
                int data[] = image.getData();
                return weight * ( data[this.n2] - data[this.n3] );
            }
        }

        // HaarRectTop implements the case of a rectangle at the top
        class HaarRectTop extends HaarRect {
            public HaarRectTop(int tlx, int w, int h, int weight) {
            }
            protected int eval(Gray32Image image) {
                int data[] = image.getData();
                return weight * ( data[this.n2] - data[this.n4] );
            }
        }

        // HaarRectTopLeft implements the case of a rectangle
        // at the top left
        class HaarRectTopLeft extends HaarRect {
            public HaarRectTopLeft(int w, int h, int weight) {
            }
            protected int eval(Gray32Image image) {
                int data[] = image.getData();
                return weight * ( data[this.n2] );
            }
        }

        // Private variables in HaarFeature
        private boolean bTilted;
        private HaarRect rect[];
        // eval for HaarFeature: subclasses above implement special
        // cases for rectangles in different positions
        public int eval(Gray32Image image) {
            int nSum = 0;
            for (int i=0; i<rect.length; i++) {
                nSum += rect[i].eval(image);
            }
            return nSum;
        }
    };
```

## HaarWeakClassifierStump ##

The results from HaarFeature are added together to get a value which is compared to a threshold. If the value exceeds the threshold, one pre-set value is returned from HaarWeakClassifierStump; otherwise, another value is used.

The threshold used in the OpenCV Haar classifier is actually the product of the value from the XML file and the standard deviation of the image. One complication is that the threshold and pre-set return values are always specified in floating-point in the OpenCV’s Haar classifier. In the JJIL implementation we convert these values to fixed-point by multiplying by 4096 = 212.

The code is shown below.
```
    public int eval(Gray32Image image) {
        int nHf = this.feature.eval(image) << 12;
        if (nHf < this.modThreshold) {
            return a;
        } else {
            return b;
        }
    }
```

## HaarClassifierStump ##

HaarClassifierStump simply sums the results from HaarWeakClassifierStump and returns a boolean value indicating whether or not the result exceeds the threshold. As with HaarWeakClassifierStump, the threshold values in OpenCV are floating-point and have been scaled by 4096 for accuracy. The code is shown below.
```
    public boolean eval(Gray32Image image) {
        int stageSum = 0;
        for (int i=0; i<this.hwcs.length; i++) {
            stageSum += this.hwcs[i].eval(image);
        }
        return (stageSum >= this.threshold);
    }
```
## HaarClassifier ##

HaarClassifier forms the cumulative sum of the image (used by HaarFeature to compute the rectangular sums) with Gray8QmSum and computes its standard deviation (used by HaarWeakClassifierStump) with Gray8Statistics. It also passes the image width (if it has changed) to the HaarClassifierStump objects for use in optimizing array address computation. It then applies each HaarClassifierStump to the image and returns true if all steps pass. The code is shown below.
```
    private Gray8Statistics gs = new Gray8Statistics();
    private Gray8QmSum gcs = new Gray8QmSum();
    private int nWidth = 0;
    private HaarClassifierStump[] hsc;

    public boolean eval(Image image) throws IllegalArgumentException {
        if (!(image instanceof Gray8Image)) {
            throw new IllegalArgumentException(image.toString() + "" +
                " should be a Gray8Image, but isn't");
        }
        this.gs.Push(image);
        int stdDev = this.gs.getStdDev();
        int nWidth = image.getWidth();
        if (this.nWidth != nWidth) {
            for (int i=0; i<this.hsc.length; i++) {
                this.hsc[i].setWidth(nWidth);
            }
        }
        this.nWidth = nWidth;
        this.gcs.Push(image);
        Gray32Image g32 = (Gray32Image) this.gcs.Front();
        int stageSum = 0;
        for (int i=0; i<this.hsc.length; i++) {
            this.hsc[i].setStdDev(stdDev);
            if (!this.hsc[i].eval(g32)) {
                return false;
            }
        }
        return true;
    } 
```