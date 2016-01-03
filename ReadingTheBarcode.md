# Reading the barcode #

The principal problem with reading the barcode is dealing with an image that may be at the limits of the resolution required for correct recognition due to low resolution or poor focus. The quality of the barcode recognizer will primarily be determined by how well we deal with this step. At the same time, however, it must be admitted that the accuracy of the barcode reader is primarily determined by how well the barcode is in focus.

Given that the barcode can contain only a fixed number of different patterns, determined by the digit codes, the best possible algorithm will take advantage of this limitation and directly compare the image with the possible patterns. The degradation to the barcode image comes from poor focus, low resolution, and compression. All of these effects can be modeled as position-independent noise applied to the image. Correlation gives the best results when dealing with position-independent noise. As a result, the best result will be obtained by directly correlating the barcode image with the possible digit codes (once we know the position of each digit code).

This is in contrast to some other approaches to barcode recognition using image sensors.   For example, an obvious approach is to use edge detection between the white and black bars and then use the position of the edges to determine the barcode pattern. For this approach to work the barcode would have to be in sharp enough focus and at high enough resolution for the edges to be readily detected. Correlation can be used to choose between images where edges are highly blurred. We would expect, and experiments showed during the development of this algorithm, that correlation gives better results.

Here is the algorithm we will use for barcode reading:

  1. Determine the left and right edges of the barcode, which will be two nearly-vertical lines in the image, by filtering the image with a low-frequency edge detector and using the Hough transform to search for nearly vertical lines.
  1. For each row in the barcode:
    1. Determine the position of the start, middle, and end patterns, using correlation, with the estimated right and left edges of the barcode used as guides.
    1. For each digit:
      1. Calculate the digit position in the row.
      1. Correlate the row’s pixels with the possible digit codes.
      1. Determine the best match.
    1. Combine the best digits and apply the parity and check digit tests described below. If the barcode passes these tests increment its occurrence count.
  1. The most commonly-occurring barcode is the result.

It should be noted that many similar approaches were tested while developing this algorithm. For example:

  1. Several different ways of rectifying the trapezoidal barcode image (stretching it so it was perfectly rectangular) were tried. The reason for this was that the rectified barcode stripes would then be vertical, making them easy to manipulate. For example, several rows of the barcode image could be averaged, reducing noise. However, it was found that this did not improve performance, possibly because the filtering introduced while rectifying the image overwhelmed any noise reduction.

> 2. The left and right barcode digits were determined independently, and then any left-right combinations which passed the check digit test were counted. For example, if “0600341” and “0700352” were found to be possible matches for the left side and “478123” was found to be a possible match for the right side then “0600341478123” and “0700352478123” would be counted as possible barcodes, if they passed the check-digit test.

> It was found that this did not lead to improved performance because there were too many legal combinations of left-right digit groups. The main source of error when reading the barcode is when the start, middle, or end pattern is found at the wrong position in step 1 above; this throws off the placement of the digit codes and leads to the wrong code being read all the way across the barcode. A single row with a misplaced pattern can result in incorrect codes on both the left and right sides. These are unlikely (one in ten) to pass the check digit test. However, if there are only a few incorrect codes on each side and they are combined with other left or right codes, there is a good chance that at least one of the incorrect patterns will pass the check digit test, since if there are L left digit sequences and R right digit sequences there will be LR combinations and LR/10 will pass the check digit test. L and R need only be as large as 3 or 4 to give at least one incorrect barcode.

> 3. Adjacent digit codes can blur enough in poor focus so that some of the blur from an elementary bar in one digit extends over some of the elementary bars in the next digit. One way to deal with this is to artificially blur the previous digit and add the blurred result to the blurred image of the next digit before correlating the combined blurred digit pair with the image. The idea is that the ‘1’ in, say, ‘01’, will look differently from the ‘1’ in, say, ‘61’ because of the blur from the out-of-focus elementary bars at the right side of the digit code for ‘0’ is different from the blur from the out-of-focus elementary bars at the right side of the digit code for ‘6’.

> This leads to a difficult and expensive search through digit pairs to find the most likely sequence (because each digit match affects the possible matches for the next digit, so there is an exponential growth in the number of possible matches as we work our way across the barcode). Also, more significantly, even images that were highly defocused did not show better performance with this method. Simply correlating with individual digits worked just as well. It might be that handling focus in this way would help if the digits were far enough out of focus, but barcodes that far out of focus could not be read because the start and end patterns could not be located.

Even using correlation to read the barcode, it is still easy to make mistakes. EAN-13 barcodes include parity and check-digit tests to improve performance. See [ParityAndCheckDigitTests](ParityAndCheckDigitTests.md) for information on how these are implemented in JJIL.