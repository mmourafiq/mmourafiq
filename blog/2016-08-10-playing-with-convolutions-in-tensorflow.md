---
layout: blog
title: "Playing with convolutions in TensorFlow"
subtitle: "From a short introduction of convolutions to a complete model."
cover_image: images/blog/tensorflow.jpg
cover_image_caption: ""
tags: [deep-learning, python, tensorflow]
---
In this post we will try to develop a practical intuition about convolutions and visualize different steps used in convolutional neural network architectures. The code used for this tutorial can be found [here](https://github.com/mmourafiq/tensorflow-convolution-models).

This tutorial does not cover back propagation, sparse connectivity, shared weights and other theoritical aspects that are already covered in other courses and tutorials. Instead it focuses on giving a practical intuition on how to use tensorflow to build a convolutional model.

* So what are convoltions?

Convolution is a mathematical operation between two functions producing a third convoluted function that is a modefied version of the first function. In the case of image processing, it's the process of multiplying each element of matrix with its local neighbors, weighted by the kernel (or filter). For example, given a maxtrix $$ M $$ and kernel $$ c $$ as follow:

$$
M =
\left( \begin{array}{ccccc}
m_{00} & m_{01} & m_{02} & m_{03} & m_{04} \\
m_{10} & m_{11} & m_{12} & m_{13} & m_{14} \\
m_{20} & m_{21} & m_{22} & m_{23} & m_{24} \\
m_{30} & m_{31} & m_{32} & m_{33} & m_{34} \\
m_{40} & m_{41} & m_{42} & m_{43} & m_{44}
\end{array}\right)
$$

and

$$
c =
\left( \begin{array}{ccc}
c_{00} & c_{01} & c_{02} \\
c_{10} & c_{11} & c_{12} \\
c_{20} & c_{21} & c_{22}
\end{array}\right)
$$

The discrete convolution operation is defined as

$$
\begin{aligned}
conv(M, c)[1, 1] = & m_{11} * c_{00} + m_{12} * c_{01} + m_{13} * c_{02} + \\
               & m_{21} * c_{10} + m_{22} * c_{11} + m_{23} * c_{12} + \\
               & m_{31} * c_{20} + m_{32} * c_{21} + m_{33} * c_{22}
\end{aligned}
$$

To visualize how convolution slides to calculate the output matrix, it's good to look at a vizualization:

![Convolution with 3×3 Filter](/images/blog/convolution_schematic.gif)
Source: http://deeplearning.stanford.edu/wiki/index.php/Feature_extraction_using_convolution

In convolutional architectures it's also common to use pooling layer after each convolution, these pooling layers generally simplify the information of the convolution layer before, by choosing the most prominent value (max pooling) or averaging the values calculated in by the convolution (average pooling).

![Pooling](/images/blog/pooling_schematic.gif)
Source: http://deeplearning.stanford.edu/wiki/index.php/File:Pooling_schematic.gif

 * Computing convolutions and pooling:

First of all let's prepare the environment:

```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
import tensorflow as tf
from PIL import Image
import numpy
```

We need also a picture to play with

```python
img = Image.open('gray_kitten.jpg')
plt.imshow(img)
```

![Gray kitten](/images/blog/gray_kitten.jpg)

And in order to visualize the result of each operation, we need to write some utils functions

```python
def show_image_ops_gray(img, conv_op, sigmoid_op, avg_pool_op, max_pool_op):
    gs1 = gridspec.GridSpec(1, 5)
    plt.subplot(gs1[0, 0]); plt.axis('off'); plt.imshow(img[:, :], cmap=plt.get_cmap('gray'))
    plt.subplot(gs1[0, 1]); plt.axis('off'); plt.imshow(conv_op[0, :, :, 0], cmap=plt.get_cmap('gray'))
    plt.subplot(gs1[0, 2]); plt.axis('off'); plt.imshow(sigmoid_op[0, :, :, 0], cmap=plt.get_cmap('gray'))
    plt.subplot(gs1[0, 3]); plt.axis('off'); plt.imshow(avg_pool_op[0, :, :, 0], cmap=plt.get_cmap('gray'))
    plt.subplot(gs1[0, 4]); plt.axis('off'); plt.imshow(max_pool_op[0, :, :, 0], cmap=plt.get_cmap('gray'))
    plt.show()


def show_image_ops_rgb(img, conv_op, sigmoid_op, avg_pool_op, max_pool_op):
    gs1 = gridspec.GridSpec(1, 5)
    plt.subplot(gs1[0, 0]); plt.axis('off'); plt.imshow(img[:, :, :])
    plt.subplot(gs1[0, 1]); plt.axis('off'); plt.imshow(conv_op[0, :, :, :])
    plt.subplot(gs1[0, 2]); plt.axis('off'); plt.imshow(sigmoid_op[0, :, :, :])
    plt.subplot(gs1[0, 3]); plt.axis('off'); plt.imshow(avg_pool_op[0, :, :, :])
    plt.subplot(gs1[0, 4]); plt.axis('off'); plt.imshow(max_pool_op[0, :, :, :])
    plt.show()

def show_shapes(img, conv_op, sigmoid_op, avg_pool_op, max_pool_op):
    print("""
        image filters (shape {})
        conv_op filters (shape {})
        sigmoid_op filters (shape {})
        avg_pool_op filters (shape {})
        max_pool_op filters (shape {})
        """.format(
            img.shape, conv_op.shape, sigmoid_op.shape, avg_pool_op.shape, max_pool_op.shape))
```

Now that we have a way to visualize every step, let's create the tensorflow operations

```python
def convolve(img, kernel, strides=[1, 3, 3, 1], pooling=[1, 3, 3, 1], padding='SAME', rgb=True):
    with tf.Graph().as_default():
        num_maps = 3
        if not rgb:
            num_maps = 1  # set number of maps to 1
            img = img.convert('L', (0.2989, 0.5870, 0.1140, 0))  # convert to gray scale


        # reshape image to have a leading 1 dimension
        img = numpy.asarray(img, dtype='float32') / 256.
        img_shape = img.shape
        img_reshaped = img.reshape(1, img_shape[0], img_shape[1], num_maps)

        x = tf.placeholder('float32', [1, None, None, num_maps])
        w = tf.get_variable('w', initializer=tf.to_float(kernel))

        # operations
        conv = tf.nn.conv2d(x, w, strides=strides, padding=padding)
        sig = tf.sigmoid(conv)
        max_pool = tf.nn.max_pool(sig, ksize=[1, 3, 3, 1], strides=[1, 3, 3, 1], padding=padding)
        avg_pool = tf.nn.avg_pool(sig, ksize=[1, 3, 3, 1], strides=[1, 3, 3, 1], padding=padding)

        init = tf.initialize_all_variables()
        with tf.Session() as session:
            session.run(init)
            conv_op, sigmoid_op, avg_pool_op, max_pool_op = session.run([conv, sig, avg_pool, max_pool],
                                                                        feed_dict={x: img_reshaped})

        show_shapes(img, conv_op, sigmoid_op, avg_pool_op, max_pool_op)
        if rgb:
            show_image_ops_rgb(img, conv_op, sigmoid_op, avg_pool_op, max_pool_op)
        else:
            show_image_ops_gray(img, conv_op, sigmoid_op, avg_pool_op, max_pool_op)
```

The convolve function that we just built, will allow us to try any filter from this list of filters in the [gimp's documentation](https://docs.gimp.org/en/plug-in-convmatrix.html). We will try some of them here, but you can modify the kernels or other kernels from the list to see how image changes.

 * sharpen filter

```python
a = np.zeros([3, 3, 1, 1])
a[1, 1, :, :] = 5
a[0, 1, :, :] = -1
a[1, 0, :, :] = -1
a[2, 1, :, :] = -1
a[1, 2, :, :] = -1
convolve(img, a, rgb=False)
```

This gives the following results

```python
image filters (shape (134, 240))
conv_op filters (shape (1, 45, 80, 1))
sigmoid_op filters (shape (1, 45, 80, 1))
avg_pool_op filters (shape (1, 15, 27, 1))
max_pool_op filters (shape (1, 15, 27, 1))
```

![sharpen_filter](/images/blog/sharpen_filter_gray.png)

 * blure filter

```python
a = np.zeros([3, 3, 1, 1])
a[1, 1, :, :] = 0.25
a[0, 1, :, :] = 0.125
a[1, 0, :, :] = 0.125
a[2, 1, :, :] = 0.125
a[1, 2, :, :] = 0.125
a[0, 0, :, :] = 0.0625
a[0, 2, :, :] = 0.0625
a[2, 0, :, :] = 0.0625
a[2, 2, :, :] = 0.0625
convolve(img, a, rgb=False)
```

This gives the following results

```python
image filters (shape (134, 240))
conv_op filters (shape (1, 45, 80, 1))
sigmoid_op filters (shape (1, 45, 80, 1))
avg_pool_op filters (shape (1, 15, 27, 1))
max_pool_op filters (shape (1, 15, 27, 1))
```

![blure_filter](/images/blog/blure_filter_gray.png)

 * Convolutional model

After trying the filters in the example to see how they change the original image, we not only started to develop an intuition about how these operations work, but also we prepared the practical tools to build a convolutional neural network.

CNNs are a family of neural network architecture built essentially based on multiple layers of convolutions with nonlinear activation functions, e.g sigmoid, relu or tanh applied to the results, followed with either other convolutions layers or pooling layers and finally fully connected layers. In this section we will be using the high-level machine learning API `tf.contrib.learn`, `tf.contrib.layers` and `tf.contrib.layers` to create, train and configure our models.

 * LeNet model

LeNet model contains the essence of CNNs that are still used in larger and newer models. LeNet consists of 2 convolutional layers followed by a dense layer.

A convolution layer in LeNet model consists of a convolution operation followed by a max pooling operation:

```python
def lenet_layer(tensor_in, n_filters, kernel_size, pool_size, activation_fn=tf.nn.tanh,
                padding='SAME'):
    conv = tf.contrib.layers.convolution2d(tensor_in,
                                           num_outputs=n_filters,
                                           kernel_size=kernel_size,
                                           activation_fn=activation_fn,
                                           padding=padding)
    pool = tf.nn.max_pool(conv, ksize=pool_size, strides=pool_size, padding=padding)
    return pool
```

We need also to define our fully connected layer which could have some dropout:

```python
def dense_layer(tensor_in, layers, activation_fn=tf.nn.tanh, keep_prob=None):
    if not keep_prob:
        return tf.contrib.layers.stack(
            tensor_in, tf.contrib.layers.fully_connected, layers, activation_fn=activation_fn)

    tensor_out = tensor_in
    for layer in layers:
        tensor_out = tf.contrib.layers.fully_connected(tensor_out, layer,
                                                       activation_fn=activation_fn)
        tensor_out = tf.contrib.layers.dropout(tensor_out, keep_prob=keep_prob)

    return tensor_out
```

Using the layers defined earlier we can easily define LeNet model:

```python
def flatten_convolution(tensor_in):
    tendor_in_shape = tensor_in.get_shape()
    tensor_in_flat = tf.reshape(tensor_in, [tendor_in_shape[0].value or -1, np.prod(tendor_in_shape[1:]).value])
    return tensor_in_flat

    def lenet_model(X, y, image_size=(-1, IMAGE_SIZE, IMAGE_SIZE, 1), pool_size=(1, 2, 2, 1)):
        y = tf.one_hot(y, 10, 1, 0)
        X = tf.reshape(X, image_size)

        with tf.variable_scope('layer1'):
            """
            Valid:
             * input: (?, 28, 28, 1)
             * filter: (5, 5, 1, 4)
             * pool: (1, 2, 2, 1)
             * output: (?, 12, 12, 4)
            Same:
             * input: (?, 28, 28, 1)
             * filter: (5, 5, 1, 4)
             * pool: (1, 2, 2, 1)
             * output: (?, 14, 14, 4)
            """
            layer1 = lenet_layer(X, 4, [5, 5], pool_size)

        with tf.variable_scope('layer2'):
            """
            VALID:
             * input: (?, 12, 12, 4)
             * filter: (5, 5, 4, 6)
             * pool: (1, 2, 2, 1)
             * output: (?, 4, 4, 6)
             * flat_output: (?, 4 * 4 * 6)
            SAME:
             * input: (?, 14, 14, 4)
             * filter: (5, 5, 4, 6)
             * pool: (1, 2, 2, 1)
             * output: (?, 7, 7, 6)
             * flat_output: (?, 7 * 7 * 6)
            """
            layer2 = lenet_layer(layer1, 6, [5, 5], pool_size)
            layer2_flat = flatten_convolution(layer2)

        result = dense_layer(layer2_flat, [1024], activation_fn=tf.nn.tanh, keep_prob=0.5)
        prediction, loss = tf.contrib.learn.models.logistic_regression_zero_init(result, y)
        train_op = tf.contrib.layers.optimize_loss(
            loss, tf.contrib.framework.get_global_step(), optimizer='Adagrad',
            learning_rate=0.1)
        return {'class': tf.argmax(prediction, 1), 'prob': prediction}, loss, train_op
```

Finally after running our model, we can look at the loss and the model graph

![LeNet model graph](/images/blog/lenet_model.png)
![model loss](/images/blog/lenet_loss.png)
![model loss mean](/images/blog/lenet_loss_mean.png)

Here's also AlexNet's model graph

![AlexNet model graph](/images/blog/alexnet_model.png)


The full code used in this tutorial can be found in this github [repo](https://github.com/mmourafiq/tensorflow-convolution-models)
