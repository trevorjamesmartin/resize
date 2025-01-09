Resize (forked)
======

Image resizing for the [Go programming language](http://golang.org) with common interpolation methods.



Installation
------------

```bash
$ go get github.com/trevorjamesmartin/resize
```

It's that easy!

Usage
-----

This package needs at least Go 1.1. Import package with

```go
import "github.com/trevorjamesmartin/resize"
```

The resize package provides 2 functions:

* `resize.Resize` creates a scaled image with new dimensions (`width`, `height`) using the interpolation function `interp`.
  If either `width` or `height` is set to 0, it will be set to an aspect ratio preserving value.
* `resize.Thumbnail` downscales an image preserving its aspect ratio to the maximum dimensions (`maxWidth`, `maxHeight`).
  It will return the original image if original sizes are smaller than the provided dimensions.

```go
resize.Resize(width, height uint, img image.Image, interp resize.InterpolationFunction) image.Image
resize.Thumbnail(maxWidth, maxHeight uint, img image.Image, interp resize.InterpolationFunction) image.Image
```

The provided interpolation functions are (from fast to slow execution time)

- `NearestNeighbor`: [Nearest-neighbor interpolation](http://en.wikipedia.org/wiki/Nearest-neighbor_interpolation)
- `Bilinear`: [Bilinear interpolation](http://en.wikipedia.org/wiki/Bilinear_interpolation)
- `Bicubic`: [Bicubic interpolation](http://en.wikipedia.org/wiki/Bicubic_interpolation)
- `MitchellNetravali`: [Mitchell-Netravali interpolation](http://dl.acm.org/citation.cfm?id=378514)
- `Lanczos2`: [Lanczos resampling](http://en.wikipedia.org/wiki/Lanczos_resampling) with a=2
- `Lanczos3`: [Lanczos resampling](http://en.wikipedia.org/wiki/Lanczos_resampling) with a=3

Which of these methods gives the best results depends on your use case.

Sample usage:

```go
package main

import (
	"github.com/trevorjamesmartin/resize"
	"image"
	"image/gif"
	"image/jpeg"
	"image/png"
	"log"
	"os"
)

func makeThumbNail(imagePath, thumb string) {
	var err error

	file, err := os.Open(imagePath)

	if err != nil {
		log.Fatal(err)
	}

	defer file.Close()

	var img image.Image

	// decode into image.Image
	switch filepath.Ext(imagePath) {
	case ".jpg", ".jpeg":
		img, err = jpeg.Decode(file)
		if err != nil {
			log.Fatal(err)
		}
	case ".png":
		img, err = png.Decode(file)
		if err != nil {
			log.Fatal(err)
		}
	case ".gif":
		img, err = gif.Decode(file)
		if err != nil {
			log.Fatal(err)
		}
	default:
		log.Fatal("Unsupported file type")
	}
	// resize to 640x360 using Lanczos resampling
	// and preserve aspect ratio
	m := resize.Thumbnail(640, 360, img, resize.Lanczos3)

	// write new image to file
	out, err := os.Create(thumb)

	if err != nil {
		log.Fatal(err)
	}

	defer out.Close()

	err = jpeg.Encode(out, m, nil)

	if err != nil {
		log.Fatal(err)
	}

}

func main() {
	// thumbnail "test.jpg"
	makeThumbnail("test.jpg", "test_thmb.jpg")
}
```

Caveats
-------

* Optimized access routines are used for `image.RGBA`, `image.NRGBA`, `image.RGBA64`, `image.NRGBA64`, `image.YCbCr`, `image.Gray`, and `image.Gray16` types. All other image types are accessed in a generic way that will result in slow processing speed.
* JPEG images are stored in `image.YCbCr`. This image format stores data in a way that will decrease processing speed. A resize may be up to 2 times slower than with `image.RGBA`. 


Downsizing Samples
-------

Downsizing is not as simple as it might look like. Images have to be filtered before they are scaled down, otherwise aliasing might occur.
Filtering is highly subjective: Applying too much will blur the whole image, too little will make aliasing become apparent.
Resize tries to provide sane defaults that should suffice in most cases.

License
-------

Copyright (c) 2012 Jan Schlicht <janschlicht@gmail.com>
Resize is released under a MIT style license.
