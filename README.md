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

Here, we provide a demonstration of many of the possible augmentations available in the package as well as different techniques that may be used to apply and combine augmentations. In order to visualize these augmentations, we utilize the Lung 2017 whole lung segmentation dataset from TCIA available [here](). We isolated the lung segmentations from this dataset as a binary mask in order to simplify the visualizations. The image and mask from can be loaded as follows.

```python
import transform as tr
reader = tr.ReadFromPath()
image, mask = reader('image.nrrd', 'primary-seg.nrrd')

```

The file path to the image is ‘image.nrrd’ and the path to the mask is ‘primary-seg.nrrd’. These parameters can be set to the path of your own image and mask for augmentation. The original image and corresponding mask we will be using for this tutorial are shown below:
