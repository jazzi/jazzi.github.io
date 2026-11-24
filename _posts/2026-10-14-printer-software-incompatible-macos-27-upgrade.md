---
layout: post
---

After upgraded MacOS to version 27.0 the printing failed and shows error as below:

> Software incompatible

My printer is TSC TTP-244 Pro, the driver version is 1.29, apprantly it is built based on Intel chipset, and after upgrade my M1 Chipset based MacOS can not regonise it, so [installing Rosetta](https://pcglance.com/printer-not-working-macos-27/) is needed.

`softwareupdate --install-rosetta`

After that everything should work again.

## Printing fonts is too light

The solution is to change the default printing density, you can change the value and save it as new default, or you can change the PPD file through CUPS printing system which is the base of MacOS printing system.

```
lpstat -d # list default printer
lpoptions -d YOUR-printer -o PrintDensity=12
```

As my printer has no *PrintDensity*, so have to change it to *Darkness*

`lpoptions -d YOUR-printer -o Darkness=12*

Then reboot the system.

Hereby is the result of my printer:

```
lpoptions -d TSC_TTP_244_Pro -l    
PageSize/Page Size: Custom.WIDTHxHEIGHT *w2h4 w4h4 w4h6
Collate/Collate: *True False
PrintSpeed/Print Speed: 20 *30 40 50
Darkness/Darkness: 0 1 2 3 4 5 6 7 8 9 10 11 *12 13 14 15
AdjustHoriaontal/Horizontal Offset: *0 2.83 5.67 8.50 11.34 14.17 17.01 19.84 22.68 25.51 28.35 31.18 34.02 36.85 39.69 42.52
AdjustVertical/Vertical Offset: *0 2.83 5.67 8.50 11.34 14.17 17.01 19.84 22.68 25.51 28.35 31.18 34.02 36.85 39.69 42.52
MediaMethod/Method: *Normal Direct Transfer
PaperType/Type: *LabelGaps LabelMark Continue
MarkHeight/Gaps Height: 0 2.83 5.67 8.50 11.34 14.17 17.01 19.84 22.68 25.51 28.35 31.18 34.02 36.85 39.69 42.52 45.35 48.19 51.02 53.86 56.69
MarkOffset/Gaps Offset: *0 2.83 5.67 8.50 11.34 14.17 17.01 19.84 22.68 25.51 28.35 31.18 34.02 36.85 39.69 42.52
VerticalOffset/Vertical Offset: -72.00 -70.87 -68.03 -65.20 -62.36 -59.53 -56.69 -53.86 -51.02 -48.19 -45.35 -42.52 -39.69 -36.85 -34.02 -31.18 -28.35 -25.51 -22.68 -19.84 -17.01 -14.17 -11.34 -8.50 -5.67 -2.83 *0 2.83 5.67 8.50 11.34 14.17 17.01 19.84 22.68 25.51 28.35 31.18 34.02 36.85 39.69 42.52 45.35 48.19 51.02 53.86 56.69 59.53 62.36 65.20 68.03 70.87 72.00
Resolution/Resolution: *203dpi
ColorOption/ColorOption: *None GrayScale Halftone
PostAction/Post-Print Action: None *TearOff PeelOff Cut PartialCut
Occurrence/Occurrence: *Every Copies Job Specified
Interval/Interval: *1 2 3 4 5 6 7 8 9 10
FeedOffset/Feed Offset: -72.00 -70.87 -68.03 -65.20 -62.36 -59.53 -56.69 -53.86 -51.02 -48.19 -45.35 -42.52 -39.69 -36.85 -34.02 -31.18 -28.35 -25.51 -22.68 -19.84 -17.01 -14.17 -11.34 -8.50 -5.67 -2.83 *0 2.83 5.67 8.50 11.34 14.17 17.01 19.84 22.68 25.51 28.35 31.18 34.02 36.85 39.69 42.52 45.35 48.19 51.02 53.86 56.69 59.53 62.36 65.20 68.03 70.87 72.00
MirrorImage/Mirror Image: True *False
NegativeImage/Negative Image: True *False
DirectBuffer/Direct to Buffer: *AUTO
StoredGraphics/Stored Graphics: *AUTO
```
