---
layout: post
---

Just one command and you can comebine many jpg files into one pdf:

`magick -compress jpeg *.jpg output.pdf`

Or to ensure a natural, numeric order:

`magick -compress jpeg $(ls -v *.jpg) output.pdf`


Above command is based on two packages:

* imagemagick
* ghostscript
