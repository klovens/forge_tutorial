# Welcome to Forge

Welcome to Forge, a 3D data augmentation package designed for medical imaging analysis.
A development version of Forge is available using the following command. 

```
pip install forge
```
Forge requires torch and SimpleITK.

In order to ensure that the package is working on your system, tests can be run as follows.


```
python test_transform.py
```

Here, we provide a demonstration of many of the possible augmentations available in the package as well as different techniques that may be used to apply and combine augmentations. In order to visualize these augmentations, we utilize the Lung 2017 whole lung segmentation dataset from TCIA available [here](https://wiki.cancerimagingarchive.net/display/Public/Lung+CT+Segmentation+Challenge+2017). We isolated the lung segmentations from this dataset as a binary mask in order to simplify the visualizations. The image and mask from can be loaded as follows.

```python
import transform as tr
reader = tr.ReadFromPath()
image, mask = reader('image.nrrd', 'primary-seg.nrrd')
```

The file path to the image is ‘image.nrrd’ and the path to the mask is ‘primary-seg.nrrd’. These parameters can be set to the path of your own image and mask for augmentation. The original image and corresponding mask we will be using for this tutorial are shown below:

<img src="/Images/original/lung_original.png" alt="Original Image" width="250"/> <img src="/Images/original/lung_mask_only.png" alt="Original Mask" width="250"/> <img src="/Images/original/lung_overlay.png" alt="Original Overlay" width="250"/>

Note that we show a single 2D slice for the majority of the augmentations, but the image and mask can be visualized in 3D to view the entire transformation. 
In this tutorial we will cover examples of many of the 3D augmentations available in the package as well as how to combine multiple transformations to apply to a single image and mask.

## Example Augmentations/Transformations

Currently, the available augmentations available using Forge include:

1. Padding
```python
PADING = [20, 20, 20]
CONSTANT = 1024
tsfm = tr.Pad(PADING, method='constant', constant=CONSTANT,
                      background_label=0, pad_lower_bound=True,
                      pad_upper_bound=True, p=1.0)
img, msk = tsfm(image, mask=mask)
```
2. Masking the Foreground
