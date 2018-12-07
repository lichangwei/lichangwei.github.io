---
title: 【翻译】图片优化工具
---

[原文地址](http://addyosmani.com/blog/image-optimization-tools/)

As we saw a few weeks ago, the weight of an average web page is now almost 1.5MB (median ~1MB), with > 50% of this being images. It’s a harsh reminder that many of our pages on the web are still quite fat, a big concern for slower mobile data connections.zipjszip

BigQuery calculated medians for a HTTP Archive run
BigQuery calculated medians for a HTTP Archive run thanks to Ilya Grigorik
There have been plenty of well documented cases of page weight being heavy, with the Oakley site Brad Frost mentioned in April clocking in at ~ 25MB worth of images alone. Insanity. Just think of this on mobile: slower data, CPU, GPU..and it’s just ONE page.

Images are a non-trivial problem to solve because they occasionally need to be high-res, but at the same time small enough to not kill your users mobile data cap. My hope is that srcset will help us improve this long-term. Thankfully Blink, WebKit and soon FF will have it.

The page cost of using images on the web is however not a new problem but we’re at least moving beyond blaming scripts as the main culprit. As a reminder, here’s a quote from Adam Sontag who suggested “One less JPG” as a solution to our bickering about framework sizes back in 2012:

Tools

Where possible, it’s best to try automating image optimization so that it’s a first-class citizen in your build chain. To help, I thought I’d share some of the tools I use for this.

As a general rule run lossy optimizers first, then lossless. Most developers forget that optimizers optimize a particular file rather than the image. This means that it doesn’t make sense to optimize an image file and then resize/crop or convert it as any changes to the file will completely undo lossless optimizations and make lossy ones a lot less effective.

Grunt tasks

Grunt is a fantastic task runner I use daily and there are a number of reliable tasks that can assist with image weight reduction:

Recompressing JPG/PNG/GIF to save on bytes:
OptiPNG/jpegtran/gifsicle: grunt-contrib-imagemin
ImageOptim-CLI companion: grunt-imageoptim
Convert to WebP: grunt-webp
Spriting to reduce HTTP requests:
grunt-spritefiles
grunt-montage
Prescaling (normalization) to avoid excessive image resize/decode work:
grunt-imageNormalize
grunt-image-resize
Responsive image generation/handling:
Generate multi-resolution images: grunt-responsive-images
Clowncar technique: grunt-clowncar
Inline images as data URIs (careful as costly on mobile):
grunt-image-embed
Of course, not everyone uses Grunt so let’s take a look at some individual tools you can use regardless of your tooling choices.

Individual tools

Some of the image compressions tools I recommend checking out include:

PNG:
pngcrush
optipng
advpng
For Windows, see pngout and pngwolf.
zopfli-png and Pngnq S9 by Kornel
PNG Quantizer
pngquant ~ recommended
pngnq
JPG:
jpegmini
jpegcrush
jpegoptim
jpgtran
GIF:
gifsicle
The Yeoman team have a Node.js wrapper called node-gifsicle that makes this available as a local dependency on OS X, Linux and Windows in case you’re interested.We have wrappers for optipng, jpegtran, pngquant too.

SVG
SVGO which has a Grunt task called grunt-svgmin
SVGCleaner
You may also find that removing EXIF data and unneeded color profile information from images also leads to some gains.

List of tools? Argh. What should I use?

Image compression expert Kornel Lesiński was kind enough to reach out with some recommendations for what to use based on research and usage of them. If opting for your own tooling chain:

For JPEG:

JPEGMini – lossy (30-50% reduction)
JPEGMini sets the quality of your JPEG to the lowest setting the human eyes can tolerate. It’s quite good at doing this. If you’re unable to use it, consider manually adjusting the quality as low as possible. Be careful though as you shouldn’t just save all JPEGs at “80%”. The quality setting is only a weak approximation and quality that you actually achieve can vary from image to image.

JPEGMini doesn’t really have an open-source/CLI equivalent (ImageOptim-CLI scripts it though) but the closest equivalent is adept-jpg-compressor.

jpegcrush (same as jpegrescan) is lossless (5-10% reduction), beating jpegoptim in 99% of cases. jpegcrush is a Perl script utilizing jpegtran, so there’s little need to use jpegtran separately.
For PNG:

There are 3 steps involved in PNG compression: first lossy conversion (50-70% reduction), then search for optimal filters (5-10%), and then optimal gzip (5-30%).

pngquant2 provides a competitive filesize and quality compression option for PNG. Windows users can use Tinypng.org which is pngquant2+optipng (and Kraken.io is the same thing again). Note that most “stable” Linux distributions ship pngquant 1.0. This is quite old and offers significantly poorer quality encodes. pngquant is worth using from version 1.6 up.
pngnq-s9, pngnq and Photoshop export (if you don’t have alpha) are also decent options worth trying (they’re okay). I would suggest staying away from RIOT, PHP-libgd and if at all possible ImageMagick and IrfanView as they aren’t great at PNG8 and don’t fully support alpha either.

cryopng is also worth checking out and (if you have time) pngwolf, which was mentioned earlier. Alternatively Optipng or pngcrush.
advpng probably has the best speed/compression ratio and I believe that’s what punypng and Kraken.io use too. If you have time, then Zopflipng is also worth considering. It’s quite slow, but beats everything else 95% of the time. PNGOUT is a close second (and pretty slow too).
Online tools

There are also a number of free online tools you can use for optimization including some of those mentioned already: Kraken.io, punypng, smush.it, tinypng and jpegmini. Also check out Spriteme for combining background images into CSS sprites.

Desktop tools

If you’re primarily a designer or don’t have a build process setup, please consider at least running your images through tools like ImageOptim or ImageAlpha as they will shave bytes off your images and keep your pages a little more lean.

You might also find this write-up on image compression for web developers by Colt McAnlis of interest.

mod_pagespeed

For those looking for a more automated server-side solution to image optimization, mod_pagespeed is an Apache module created by some of my colleagues at Google to speed up pages to reduce latency and bandwidth. A list of image optimization techniques it supports is available and includes inlining and recompression.

Others?

If there are other tools or Grunt tasks you’ve found helpful for image optimization, please feel free to share them. I know that both I and others are always interested in benchmarking new alternatives.

Wrapping up

Mobile users are the biggest victims of image bloat on web pages. They take ages to load on slow connections and when used without any optimization can make for a costly user experience.

Respect your user’s time, try to keep your pages lean and with some luck we’ll make the web just a little bit faster.
