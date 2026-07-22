---
layout: post
---

Firstly install ImageMagick and ghostscript, both are available with Homebrew.

Then add the following two functions into ~/.zshrc

```
function pdf2jpg () { magick "$1" -density 300 -background white -alpha remove +adjoin -verbose -quality 100 -sharpen 0x1.0 "$1-%04d.jpg"; }

function pdf2jpgpw () { magick "$1" -authenticate "$2" -density 300 -background white -alpha remove +adjoin -verbose -quality 100 -sharpen 0x1.0 "$1-%04d.jpg"; }
```

Same function but easier understand form as below:

```
function pdf2jpg () {
    magick "$1" \
        -density 300 \
        -background white \
        -alpha deactivate \
        +adjoin \
        -verbose \
        -quality 100 \
        -sharpen 0x1.0 \
        "$1-%04d.jpg"
}

function pdf2jpgpw () {
    magick "$1" \
        -authenticate "$2" \
        -density 300 \
        -background white \
        -alpha deactivate \
        +adjoin \
        -verbose \
        -quality 100 \
        -sharpen 0x1.0 \
        "$1-%04d.jpg"
}
```

Once functions added, reload shell or `source ~/.zshrc` to get effected.

Then these commands can be called to generate jpgs from multiple pages PDF:

`pdf2jpg xxx.pdf`

which will generate a bunch of JPG files under the same folder; or, if the PDF is password protected,

`pdf2jpg xxx.pdf password`

will have the same effect.
