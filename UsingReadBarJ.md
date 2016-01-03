# Introduction #

Not all image types work with all cellphone cameras, and there doesn't appear to be any good way to verify whether or not a particular image resolution, for example, will "work". So ReadBarJ provides a simple user interface to allow you to change the image source, format, and resolution to suit your cellphone camera.

When you start ReadBarJ you will see a simple menu (labeled "Options" in the emulator that comes with the NetBeans IDE). The menu choices there are in three groups:

  1. Image source: this can be nothing (uses default image capture), file, and other image sources provided by your cellphone such as image, video, and so on. Some cellphones have more than one camera so this allows you to choose the appropriate one. Note: ReadBarJ gets its image sources from a system-defined key, and not all the image sources may, in fact, be capable of capturing video (used in the preview).
  1. Image format: this depends on the cellphone. Some provide PNG or JPG only, others provide more of a choice. This may affect the compression applied to the image when it is captured.
  1. Image resolution: currently the resolution choices are 160x120, 320x240, 640x480 (VGA), and 1280x960. Some cellphones do not support all these resolutions and others may support capture but run out of memory (creating an unrecoverable program error) when actually processing the image.

When you make your selections from the menu the current selection will be shown prefixed by a '+' sign. The selections are saved from program run to program run.

Generally speaking you what to use the sharpest, highest resolution image you can for barcode capture. The barcode must be at least 200 pixels in width to be read and it has to be sharply defined and in focus.

After you've made your selections pressing the "FIRE" button on your cellphone (this is usually a centrally-located button) will capture and image, locate the barcode, and show its value. The area of the image containing the barcode will be shown on the screen and its approximate edges will be outlined with green lines.