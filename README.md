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

This method uses the Otsu method of thresholding in order to generate a mask for the parts of the image considered the foreground.
```python
tsfm = tr.ForegroundMask(background='<', bins=128)
img, msk = tsfm(image)
```
The original image as well as the resulting image and mask overlayed are shown below.

<img src="/Images/original/lung_original.png" alt="Original Image" width="250"/> <img src="/Images/foreground/otsu.png" alt="Original Mask" width="250"/>

3. Cropping

There are several techniques to crop the images and masks. Random crop will crop an image randomly, but with a specific size while center crop will crop at the center of the image (and mask if provided). Random safe crop will ensure that at least some of the regions of interest (specified by having a mask) will be retained when the image is cropped. If not using safe crop, the cropped area may not include any part of the mask. The size of the cropped area is specified using a tuple containing the x, y, and z dimensions desired for the cropped area. Below we demonstrate how to call these particular transformations. The images shown include the axial, sagital, and coronal views of the image/mask pair, respectively.

* Random Crop
```python
size = (250, 250, 50)
tsfm = tr.RandomCrop(size, p=1)
img, msk = tsfm(image, mask)
```

<img src="/Images/crop/randomcrop.png" alt="Random Crop" width="700"/>

* Center Crop
```python
size = (150, 150, 50)
tsfm = tr.CenterCrop(size, p=1)
img, msk = tsfm(image, mask)
```

<img src="/Images/crop/centercrop.png" alt="Center Crop" width="700"/>

* Random Safe Crop
```python
size = (50, 50, 50)
tsfm = tr.RandomSegmentSafeCrop(crop_size=size, include=[1], p=1.0)
img, msk = tsfm(image, mask=mask)
```

<img src="/Images/crop/safecrop.png" alt="Random Safe Crop" width="700"/>

4. Rotating

Rotate the given CT image by constant value x, y, z angles. These angles can be specified as a tuple. If all the angles are equal, providing a single value for the angle is sufficient. Below we show both ways to provide the angles for the rotation.

```python
tsfm = tr.Affine(angles=(180, 180, 180), translation=0, scale=1,
                         interpolator=sitk.sitkLinear, image_background=-1024,
                         mask_background=0, reference=None, p=1)
img, msk = tsfm(image=image, mask=mask)
```

<img src="/Images/rotate/rotate180.png" alt="Rotate 180" width="700"/>

```python
tsfm = tr.Affine(angles=45, translation=0, scale=1,
                         interpolator=sitk.sitkLinear, image_background=-1024,
                         mask_background=0, reference=None, p=1)
img, msk = tsfm(image=image, mask=mask)
```

<img src="/Images/rotate/rotate45.png" alt="Rotate 180" width="700"/>

5. Invert Intensity
```python
tsfm = tr.Invert(maximum=1, p=1.0)
img, msk = tsfm(image, mask=mask)
```

6. Clipping
```python
LOWER = 0
UPPER = 150
OUTSIDE = 0
tsfm = tr.IsolateRange(lower_bound=LOWER,
                               upper_bound=UPPER,
                               image_outside_value=OUTSIDE,
                               recalculate_mask=False,
                               p=1.0)
img, msk = tsfm(image, mask=mask)
```

7. Intensity Range Transfer
```python
LOWER = 0
UPPER = 1
tsfm = tr.IntensityRangeTransfer(interval=(LOWER, UPPER),
                                         cast=None, p=1.0)
img, msk = tsfm(image, mask)
```
9. Histogram Equalization
```python
tsfm = tr.AdaptiveHistogramEqualization(alpha=1.0, beta=0.5,
                                                radius=2, p=1.0)
img, msk = tsfm(image, mask=mask)
```

10. Mask Image
This method allows for the masked and unmasked portions of an image to be isolated. This can be useful when there are large areas in the image that are not useful for a particular task, which can make it more difficult to train a model successfully.
```python
LABEL = 0
OUTSIDE_MASK_LABEL = 1
BACKGROUND = -1024
tsfm = tr.MaskImage(segment_label=LABEL,
                            image_outside_value=BACKGROUND,
                            mask_outside_label=OUTSIDE_MASK_LABEL,
                            p=1.0)
img, msk = tsfm(image, mask=mask)
```
<img src="/Images/maskimage/maskimage.png" alt="MaskImage" width="300"/>

If we run the same code with LABEL = 1 and OUTSIDE_MASK_LABEL = 0, then we can isolate the portion of the image that contained the mask, in this case the lungs.

<img src="/Images/maskimage/maskout.png" alt="MaskOut" width="300"/>


11. Blur
```python
image = sitk.Cast(image, sitk.sitkInt32)
tsfm = tr.BionomialBlur(repetition=3, p=1.0)
img, msk = tsfm(image, mask=mask)
```
<img src="/Images/blur/blur.png" alt="Blur" width="300"/>

12. Noise
* Salt and Pepper
```python
image = sitk.Cast(image, sitk.sitkInt32)
min_value, max_value = -1, 3
tsfm = tr.SaltPepperNoise(noise_prob=0.2,
                                  noise_range=(min_value, max_value),
                                  random_seed=1, p=1.0)
img, msk = tsfm(image, mask=mask)
```

<img src="/Images/saltpepper/saltpepper.png" alt="saltpepper" width="300"/>

* Gaussian
```python
tsfm = tr.AdditiveGaussianNoise(mean=0.0, std=1.0, p=1.0)
img, msk = tsfm(image, mask=mask)
```

<img src="/Images/gaussian/gaussian.png" alt="gaussian" width="300"/>

13. Resizing
14. Flipping
Visualizing in applications such as 3d Slicer will not show flipping properly so we visualize these image and masks using matplotlib. A list of 3 Boolean values indicate the axes that the image and mask (if provided) should be flipped. Here, we are flipping the image and mask around the y axis.

```python
tsfm = tr.Flip(axes=[False, True, False], p=1)
img, msk = tsfm(image=image, mask=mask)
```

## Combining a Set of Augmentations/Transformations

These augmentations can also be applied consecutively on the same image or a list of possible augmentations can be applied in a probabilistic fashion. The figures below represent these types of combined transformations.

First, we make a list of 5 potential transformations. More or fewer transformations could be included in this list depending on the kind of transformations a user wishes to apply to their images and masks. Furthermore, we are now changing the values of the parameter p, which is the probability of applying the transformation to an image/mask pair. In all previous examples we set p to 1 for all transformations meaning that the transformation would be applied in each case.

```python
tsfms = [tr.Affine(angles=45, translation=0, scale=1,
                         interpolator=sitk.sitkLinear, image_background=-1024,
                         mask_background=0, reference=None, p=0.5),
         tr.SaltPepperNoise(noise_prob=0.2,
                            noise_range=(min_value, max_value),
                            random_seed=1, p=0.75),  
         tr.RandomSegmentSafeCrop(crop_size=(250, 250, 75), include=[1], p=0.4),
         tr.BionomialBlur(repetition=3, p=0.75),
         tr.Invert(maximum=1, p=0.2)]

print(tsfms)
# [Affine (angles=[(-0.7853981633974483, 0.7853981633974483), (-0.7853981633974483, 0.7853981633974483), (-0.7853981633974483, 
# 0.7853981633974483)], interpolator=2, p=0.5), 
# SaltPepperNoise (noise_prob=0.2, random_seed=1, p=0.75), 
# RandomSegmentSafeCrop (min size=[250 250 75], interesting segments=[1], p=0.4), 
# BionomialBlur (repetition=3, p=0.75), 
# Invert (maximum=1, p=0.2)]
```

As shown in the variable tsfms, the transformations in the list are rotate, salt and pepper noise, random safe crop, blur, and invert.

**Compose** will apply the transforms sequentially to the image and mask.

**RandomChoices** will select transforms from the list of possible transforms. As an example, lets select 3 random transforms from the list to potentially apply to our image/mask.
```python
K = 3
tsfm = tr.RandomChoices(tsfms, k=K, keep_original_order=True)
img, msk = tsfm(image, mask=mask)
```
Below are some potential images that could be obtained from selecting random transformations from our predefined list of transformations.

**OneOf** will select only one transform from the list.
```python
tsfm = tr.OneOf(tsfms)
img, msk = tsfm(image, mask=mask)
```     

**RandomOrder** will apply the transforms from the list in a random order. Equivalent to RandomChoices is K was set to 5 in this example and the original order is not kept (keep_original_order=False).
```python
tsfm = tr.RandomOrder(tsfms)
img, msk = tsfm(image, mask=mask)
```


