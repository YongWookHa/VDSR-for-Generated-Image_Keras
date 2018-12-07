# Keras VDSR for Generated image

## Overview

This repository is based on [Keras-VDSR](https://github.com/GeorgeSeif/VDSR-Keras) written by [GeorgeSeif](https://github.com/GeorgeSeif).
The original model of VDSR is accessible on [The author's project page](http://cv.snu.ac.kr/research/VDSR/).

The codes in this repository are optimized to up-scale the images which are generated by VAE.
[CelebA](http://mmlab.ie.cuhk.edu.hk/projects/CelebA.html) datasets has been used for this model, and returned fine output.

## Files
- vdsr.py : main training file
- predict.py : test VDSR on an arbitrary image

## Usage

Before you run the model, make directories below.
* `checkpoints` : checkpoints are going to be saved in this directory
* `data ` : for image data you are using
* `model` : model architecture and weights are going to be saved in this directory

**vdsr.py**
> Prepare your own dataset.
> Set `DATA_X_PATH`, `DATA_Y_PATH`, `TARGET_IMG_SIZE` variables.
> `DATA_X_PATH` : Path of input images (jpg) (+) recommend to use blurred image of Y
> `DATA_Y_PATH` : Path of output images (jpg)
> `TARGET_IMG_SIZE` : Size of output imgaes
> Default size of input image data is 64x64(RBG), but you can use your own dataset. It will resize the image to `TARGET_IMG_SIZE` anyway.
> 
> You need to add code below to `keras/losses.py` file to use [PSNR](https://en.wikipedia.org/wiki/Peak_signal-to-noise_ratio) as loss function.
~~~
def tf_log10(x):
  numerator = tf.log(x)
  denominator = tf.log(tf.constant(10, dtype=numerator.dtype))
  return numerator / denominator
~~~
~~~
def psnr(y_true, y_pred):
    max_pixel = 1.0
    return 100 - 10.0 * tf_log10((max_pixel ** 2) / (K.mean(K.square(y_pred - y_true))))
~~~

**predict.py**
> Recommend to use this file in command prompt. This file has 4 argvs.
> `json_path` : model architecture (json)
> `w_path` : model weight (hdf5)
> `img_path` : arbitrary face images to refine
> `dst_path` : path to save the refined images
>
> if you are not using (128, 128) but others for output images, you should edit `target_size` as well.

## Model Structure

![image](https://user-images.githubusercontent.com/12293076/48262688-db30b680-e466-11e8-8d75-f2d8763af04c.png)

There's no ground truth image when you are up-scaling the generated image.
So, this model focuses on creating natural detail of input image.

The model has 5 residual blocks. Each block has 4 conv layers. 
With first and last conv layer, the model has total 22 conv layers.

It uses PSNR(Peak Signal to Noise Ratio) for loss function.
