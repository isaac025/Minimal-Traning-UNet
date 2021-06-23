# Post-estimation on animal Body-Parts using SLEAP

This project uses [the SLEAP](https://sleap.ai) deep-learning framework in order
to train a UNet to estimate on body-parts of animals, in this case of flies. Developed
by Talmo Pereira, Elena Sizikova, Carlos Fernandez, Sheik-Sedat Touray and Isaac Lopez.

## What is SLEAP?

SLEAP is a deep-learning framework that is built to estimate body-parts of animals
on a given dataset. It was developed by Talmo Pereira and his team.

Please visit and follow [SLEAP's Tutorial](https://sleap.ai/tutorials/tutorial.html)!

## The notebook

We provide a notebook with multiple functions that help manipulate the dataset
a lot more easier.
The dataset that sleap builds is a Labels-like container.
Basically these labels contain instances of each image taken from the original video
that when ran through SLEAP generate a file with multiple instances of an image and generate other
metadata.

### The Dataset

The dataset comes from a [dropbox link](https://www.dropbox.com/s/b990gxjt3d3j3jh/210205.sleap_wt_gold.13pt.pkg.slp?dl=1) 
at the start of the notebook.

*NOTE* This dataset belongs to Talmo Pereira and SLEAP.

Labels files ".slp" provide the complete description of a project.
Take the dataset used here of fruit-flies, when the file is loaded using 
the sleap.load_file function, that takes a .slp file only, when running
the code below we generate the following.

```py
labels = sleap.load_file("labels.slp")
labels.describe()
```
![labels_image](labels.jpg)

Each individual labels are frames and these frames contain
two instances of the image one labeled and one not. Also it
contains where exactly on the frame are the coordinates of the
label.
```py
labeled_frame = labels[0]
```
![labels_individual_image](labeled_frame.jpg)

We can take the one of the instances of the labeled frame
and use it for any purpose desired.
```py
instance = labeled_frame[0]
```

### The Network

The Network developed by Talmo is a minimal UNet that uses an architechture
from SLEAP. Click [here](https://sleap.ai/api/sleap.nn.architectures.unet.html#module-sleap.nn.architectures.unet) 
for more information of the architechture.

We create a UNet that is compatible with the input and output shapes that we're generating.
```py
# Instantiate the backbone builder.
unet = sleap.nn.architectures.unet.UNet(filters=32, filters_rate=1.5, down_blocks=4, up_blocks=3, up_interpolate=True)

# Create the input layer (see above for the dimensions)
x_in = tf.keras.layers.Input((160, 160, 1))

# Create the feature extractor backbone.
x_features, x_intermediate = unet.make_backbone(x_in)

# Do a 1x1 conv with linear activation to remap activations to the number of channels in
# the confidence maps (see above)
x_confmaps = tf.keras.layers.Conv2D(filters=13, kernel_size=1, strides=1, padding="same")(x_features)

# Create a Model that links the whole graph
model = tf.keras.Model(x_in, x_confmaps)
model.summary()
```

### Getting coordinates of Confidence Maps
For example, we can get the instance image of a video and its
confidence maps (X and Y coordinates of where precisely a body-part
is). The output pictures look like this.

![](Image2.jpg)

After training, we can get the X and Y coordinates of the confidence
maps generated and compare them to the ground truths. We developed a 
function:
```
def get_indx(list_of_GT, list_of_Pred)
```
that helps us get the X and Y indexes of the confidence
maps. With this we can easily compare them to the ground
truths and see how well our network is performing. 

![](Image1.jpg)


### Future development
Although A.I. and Machine Learning are in its initial stages, specifically on
production-side, software like SLEAP can be of great use on medicine. It could
potentially help doctors to better identify MRI scans, CT scans, X-rays, etc.,
by being faster and taking a load of work out of the doctor. 

## References
1. Deep Learning. (n.d.). https://www.deeplearningbook.org/. 

2. Pereira, T. D., Tabris, N., Li, J., Ravindranath, S., Papadoyannis, E. S., Wang, Z. Y., Turner, D. M., McKenzie-Smith, G., Kocher, S. D., Falkner, A. L., Shaevitz, J. W., &amp; Murthy, M. (2020, January 1). SLEAP: Multi-animal pose tracking. bioRxiv. https://www.biorxiv.org/content/10.1101/2020.08.31.276246v1. 

3. Pereira, T. D., Shaevitz, J. W., &amp; Murthy, M. (2020, November 9). Quantifying behavior to understand the brain. Nature News. https://www.nature.com/articles/s41593-020-00734-z.

