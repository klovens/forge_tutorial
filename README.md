# Forge: A 3D augmentation package for image analysis

Welcome to Forge, a 3D data augmentation package designed for medical imaging analysis.
A development version of Forge is available using the following command. 

```
pip install forge
```
Forge requires torch and SimpleITK.

In order to ensure that the package is working on your system, tests can be run as follows.


```
python run_tests.py
```

Here, we provide a demonstration of many of the possible augmentations available in the package as well as different techniques that may be used to apply and combine augmentations. In order to visualize these augmentations, we utilize the Lung 2017 whole lung segmentation dataset from TCIA available [here](https://wiki.cancerimagingarchive.net/display/Public/Lung+CT+Segmentation+Challenge+2017). We isolated the lung segmentations from this dataset as a binary mask in order to simplify the visualizations. The image and mask from can be loaded as follows.

```python
import transform as tr
reader = tr.Reader()
image, mask = reader('image.nrrd', 'primary-seg.nrrd')
```

The file path to the image is ‘image.nrrd’ and the path to the mask is ‘primary-seg.nrrd’. These parameters can be set to the path of your own image and mask for augmentation. The original image and corresponding mask we will be using for this tutorial are shown below:

<img src="/Images/original/lung_original.png" alt="Original Image" width="250"/> <img src="/Images/original/lung_mask_only.png" alt="Original Mask" width="250"/> <img src="/Images/original/lung_overlay.png" alt="Original Overlay" width="250"/>

Note that we show a single 2D slice for the majority of the augmentations, but the image and mask can be visualized in 3D to view the entire transformation. 
In this tutorial we will cover examples of many of the 3D augmentations available in the package as well as how to combine multiple transformations to apply to a single image and mask.

## Example Augmentations/Transformations

Currently, the available augmentations available using Forge include:

1. Padding

Padding adds a specified amount of pixels to an image. In order to demonstrate padding clearly, we set the intensity of the pixels to a value that will stand out from the background.
```python
PADING = [20, 20, 20]
CONSTANT = 1024
tsfm = tr.Pad(PADING, method='constant', constant=CONSTANT,
                      background_label=0, pad_lower_bound=True,
                      pad_upper_bound=True, p=1.0)
img, msk = tsfm(image, mask=mask)
```
<img src="/Images/pad/pad.png" alt="Padded Image" width="250"/>

2. Masking the Foreground

This method uses the Otsu method of thresholding in order to generate a mask for the parts of the image considered the foreground.
```python
tsfm = tr.ForegroundMask(background='<', bins=128)
img, msk = tsfm(image)
```
The original image as well as the resulting image and mask overlayed are shown below.

<img src="/Images/original/lung_original.png" alt="Original Image" width="250"/> <img src="/Images/foreground/otsu.png" alt="Original Mask" width="250"/>

3. Cropping

There are several techniques to crop the images and masks. These methods can be useful for applications that involve techniques such as curriculum learning strategies in order to focus on a region of interest without the added complexity of surrounding tissue. Random crop will crop an image randomly, but with a specific size while center crop will crop at the center of the image (and mask if provided). Random safe crop will ensure that at least some of the regions of interest (specified by having a mask) will be retained when the image is cropped. If not using safe crop, the cropped area may not include any part of the mask. The size of the cropped area is specified using a tuple containing the x, y, and z dimensions desired for the cropped area. Below we demonstrate how to call these particular transformations. The images shown include the axial, sagital, and coronal views of the image/mask pair, respectively.

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
<img src="/Images/invert/invertintensity.png" alt="Invert" width="300"/>

6. Clipping

Clipping will extract any intensity value in the image between an upper and a lower bound. From the voxels that fulfill this criteria the mask can be recalculated to only mask the voxels that are within this intesity range and part of the original mask.
```python
LOWER = 0
UPPER = 500
OUTSIDE = 0
tsfm = tr.IsolateRange(lower_bound=LOWER,
                               upper_bound=UPPER,
                               image_outside_value=OUTSIDE,
                               recalculate_mask=True,
                               p=1.0)
img, msk = tsfm(image, mask=mask)
```

<img src="/Images/clipping/clipping.png" alt="Clip" width="700"/>

```python
LOWER = -1038
UPPER = 1
OUTSIDE = 0
tsfm = tr.IsolateRange(lower_bound=LOWER,
                               upper_bound=UPPER,
                               image_outside_value=OUTSIDE,
                               recalculate_mask=True,
                               p=1.0)
img, msk = tsfm(image, mask=mask)
```
<img src="/Images/clipping/clipping2.png" alt="Clip 2" width="700"/>

7. Intensity Range Transfer
```python
LOWER = 0
UPPER = 10
tsfm = tr.IntensityRangeTransfer(interval=(LOWER, UPPER),
                                         cast=None, p=1.0)
img, msk = tsfm(image, mask)
```

<img src="/Images/rangetransfer/rangetransfer.png" alt="Transfer" width="700"/>

9. Histogram Equalization
```python
tsfm = tr.AdaptiveHistogramEqualization(alpha=1.0, beta=0.5,
                                                radius=2, p=1.0)
img, msk = tsfm(image, mask=mask)
```

<img src="/Images/histogram/histogram.png" alt="Histogram" width="300"/>

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

Resizing can be done in various ways using Forge. The image/mask can expand or shrink by a factor or the they can be resized to a particular dimension using Resize as shown.
```python
output_size = [2 * x for x in image.GetSize()]
              tsfm = tr.Resize(size=output_size,
                         interpolator=sitk.sitkLinear,
                         default_image_voxel_value=0,
                         default_mask_voxel_value=0,
                         p=1.0)
 img, msk = tsfm(image, mask=mask)
```

We do not show the visualizations for resizing here as it is not obvious other than the resolution going up or down (looks similar to blur when shrinking, for example). However, we can confirm that the dimensions of the image and mask have changed.
```python
shrinkage_factors = [5, 5, 1]
tsfm = tr.Shrink(shrinkage=shrinkage_factors, p=1)
img, msk = tsfm(image, mask=mask)

# Dimensions of the image and mask after shrink has been applied
>>> img.GetSize()
(102, 102, 140)
>>> msk.GetSize()
(102, 102, 140)

#original dimensions of the image and mask
>>> image.GetSize()
(512, 512, 140)
>>> mask.GetSize()
(512, 512, 140)
```

Further, the mask alone can be resized in order to segment less or more of the area of interest. Two methods are available to increase or decrease the overall area of the mask. To demonstrate when this may be useful, we make use of a image and mask from the NSCLC-Radiomics dataset from TCIA available [here](https://wiki.cancerimagingarchive.net/display/Public/NSCLC-Radiomics). Increasing or decreasing the mask of a tumor area may be desired if the boundaries around the tumors tend to be unclear. Features may be included and excluded from the mask area to determine how informative they are for endpoint prediction.

**Erode**
```python
image, mask = reader('image_tumor.nrrd','primary-tumor.nrrd')
tsfm = tr.BinaryErode(background=0, foreground=1,
                 radius= (5, 5, 1))
img, msk = tsfm(image=image, mask=mask)
```

**Dilate**
```python
tsfm = tr.BinaryDilate(background=0, foreground=1,
                 radius= (5, 5, 2))
img, msk = tsfm(image=image, mask=mask)
```

<img src="/Images/resize/originaltumorseg.png" alt="origtumor" width="200"/><img src="/Images/resize/erode.png" alt="randommethod" width="200"/><img src="/Images/resize/dilate.png" alt="randommethod" width="200"/><img src="/Images/resize/alltumorsegs.png" alt="randommethod" width="200"/>

From left to right, the images above show the original tumor segmentation (green), the resulting mask after erosion is applied (red), the mask after dilate is applied (yellow), and all of the segmentations layered to show the difference between their boundaries before and after the transformations have been applied.

15. Flipping

Visualizing in applications such as 3d Slicer will not show flipping properly so we visualize these image and masks using matplotlib. A list of 3 Boolean values indicate the axes that the image and mask (if provided) should be flipped. Here, we are flipping the image and mask around the y axis.

```python
tsfm = tr.Flip(axes=[False, True, False], p=1)
img, msk = tsfm(image=image, mask=mask)
```
<img src="/Images/flip/fliporiginal.png" alt="origtumor" width="250"/><img src="/Images/flip/flipimage.png" alt="origtumor" width="250"/><img src="/Images/flip/flipmask.png" alt="origtumor" width="250"/>

16. Resampling

Forge includes methods that allow for fixing pixel spacing issues andd general inconsistencies that may arise when analysing and work with images and their segmentations.

**Resample**

Resample and image (mask optional) according to specified origin, spacing and dimensions. The method will transform the image by mapping the points from the original coordinate system to the new specified coordinate system by the user (or default system). An Interpolator is used to obtain the intensity values at arbitrary points in the coordinate system from the values of the points defined by the Image.
```python
resample = tr.Resample()
img, msk = resample(image, mask)

img.GetSpacing()
# (1.0, 1.0, 1.0)

img.GetOrigin()
# (-249.51171875, -440.51171875, -655.0)

img.getSize()

resample = tr.Resample(interpolator=sitk.sitkBSpline,
                 output_spacing: tuple = (1, 1, 2),
                 default_image_voxel_value=0,
                 default_mask_voxel_value=0,
                 output_image_voxel_type=None,
                 output_mask_voxel_type=None,
                 output_direction=None,
                 output_origin=None)
                 
img, msk = resample(image, mask)
```

**Isotropic**

This method can make an image (with or without a mask) isotropic, i.e. equally spaced in all directions. We show the original spacing followed by the spacing after applying the method. 
```python
image.GetSize()
# (512, 512, 140)

image.GetSpacing()
# (0.9765625, 0.9765625, 3.0)

ist = tr.Isotropic()
img, _ = ist(image)

img.GetSize()
# (500, 500, 420)

img.GetSpacing()
# (1.0, 1.0, 1.0)

img, msk = ist(image, mask)
```
Further, this should not make the image and mask look significantly different

## Combining a Set of Augmentations/Transformations

These augmentations can also be applied consecutively on the same image or a list of possible augmentations can be applied in a probabilistic fashion. The figures below represent some of these methods of combining transformations.

<img src="/Images/methods/randomselection.png" alt="randommethod" width="800"/>

Each augmentation in this scenario has a probability attached to it, which is the likelihood of applying the transformation to a image/mask pair. Forge contains methods to request a particular number of augmentations be applied from a list as well as applying one or all of the augmentations in a predefined list provided by the user. Below we show an example of augmentations being applied to an image/mask consecutively.

<img src="/Images/methods/transformationmethod2.png" alt="randommethod" width="800"/>

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
```python
tsfm = tr.Compose(tsfms)
img, msk = tsfm(image, mask=mask)
```

**RandomChoices** will select transforms from the list of possible transforms. As an example, lets select 3 random transforms from the list to potentially apply to our image/mask.
```python
K = 3
tsfm = tr.RandomChoices(tsfms, k=K, keep_original_order=True)
img, msk = tsfm(image, mask=mask)
```
Below are some potential images that could be obtained from selecting random transformations from our predefined list of transformations.

<img src="/Images/randomchoice/randomchoice1.png" alt="random 1" width="250"/> <img src="/Images/randomchoice/randomchoice2.png" alt="random 2" width="250"/> <img src="/Images/randomchoice/randomchoice3.png" alt="random 3" width="250"/>

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

## Write to a file

To write any image or mask created from the transformations to a file the **Writer** class can be used.
```python
wr = tr.Writer(dir_path='.', image_prefix='image', image_postfix='',
            mask_prefix='mask', mask_postfix='', extension='nrrd')
wr(image=img, mask=msk)
```


