<!--<div align="center">
	<div class="TensorFlow">
  <img src="https://www.tensorflow.org/images/tf_logo_transp.png" style=": left; margin-left: 5px; margin-bottom: 5px;"><br><br>
   </div>
   <div class="TensorLayer"> 
    <img src="https://www.tensorflow.org/images/tf_logo_transp.png" style=": right; margin-left: 5px; margin-bottom: 5px;">
    </div>
</div>
-->


# TensorLayer: Deep learning and Reinforcement learning library for TensorFlow.

TensorLayer is a transparent deep learning and reinforcement learning library built on the top of [Google TensorFlow](https://www.tensorflow.org). It was designed to provide a higher-level API to TensorFlow in order to speed-up experimentations. TensorLayer is easy to extended and modified, suitable for both machine learning researches and applications. Welcome contribution!


TensorLayer features include:

- Fast prototyping through highly modular built-in neural network layers, pre-train metrices, regularizers, optimizers, cost functions...
- Implemented by straightforward code, easy to modify and extend by yourself...
- Other wraping libraries for TensorFlow are easy to merged into TensorLayer, suitable for machine learning researches...
- Many official examples covering Dropout, DropConnect, DropNeuron, Denoising Autoencoder, LSTM, ResNet... are given, suitable for machine learning applications...
- The running speed would not decrease compare with pure TensorFlow codes...

Now, go through the [Overview](#Overview) to see how powerful it is !!!

# Table of Contents
0. [Overview](#Overview)
0. [Library Structure](#Library-Structure)
0. [Easy to Modify](#Easytomodify)
0. [Installation](#Installation)
0. [Ways to Contribute](#Waystocontribute)

--
# Overview
More examples available *[here](https://www.xxx)*

0. [Fully Connected Network](#)
0. [Convolutional Neural Network](#)
0. [Recurrent Neural Network](#)
0. [Reinforcement Learning](#)
0. [Cost Function](#)

### *Fully Connected Network*
TensorLayer provides large amount of state-of-the-art Layers including Dropout, DropConnect, ResNet, Pre-train and so on.

**Placeholder**

All placeholder and variables can be initialized by the same way with Tensorflow's tutorial. For details please read *[tensorflow-placeholder](https://www.tensorflow.org/versions/master/api_docs/python/io_ops.html#placeholder)*, *[tensorflow-variables](https://www.tensorflow.org/versions/master/how_tos/variables/index.html)* and *[tensorflow-math](https://www.tensorflow.org/versions/r0.9/api_docs/python/math_ops.html)*.

```python
# For MNIST example, 28*28 images have 784 pixels, i.e, 784 inputs.
import tensorflow as tf
from tensorlayer import *
x = tf.placeholder(tf.float32, shape=[None, 784], name='x')
y_ = tf.placeholder(tf.int64, shape=[None, ], name='y_')
```

**Rectifying Network with Dropout**

```python
# Define the network
network = InputLayer(x, name='input_layer')
network = DropoutLayer(network, keep=0.8, name='drop1')
network = DenseLayer(network, n_units=800, act = tf.nn.relu, name='relu1')
network = DropoutLayer(network, keep=0.5, name='drop2')
network = DenseLayer(network, n_units=800, act = tf.nn.relu, name='relu2')
network = DropoutLayer(network, keep=0.5, name='drop3')
network = DenseLayer(network, n_units=10, act = identity, name='output_layer')
# Start training
...
```

**Vanilla Sparse Autoencoder**

```python
# Define the network
network = InputLayer(x, name='input_layer')
network = DenseLayer(network, n_units=196, act = tf.nn.sigmoid, name='sigmoid1')
recon_layer1 = ReconLayer(network, x_recon=x, n_units=784, act = tf.nn.sigmoid, name='recon_layer1')
# Start pre-train
sess.run(tf.initialize_all_variables())
recon_layer1.pretrain(sess, x=x, X_train=X_train, X_val=X_val, denoise_name=None, n_epoch=200, batch_size=128, print_freq=10, save=True, save_name='w1pre_')
...
```

**Denoising Autoencoder**

```python
# Define the network
network = InputLayer(x, name='input_layer')
network = DropoutLayer(network, keep=0.5, name='denoising1')   
network = DenseLayer(network, n_units=196, act = tf.nn.relu, name='relu1')
recon_layer1 = ReconLayer(network, x_recon=x, n_units=784, act = tf.nn.softplus, name='recon_layer1')
# Start pre-train
sess.run(tf.initialize_all_variables())
recon_layer1.pretrain(sess, x=x, X_train=X_train, X_val=X_val, denoise_name='denoising1', n_epoch=200, batch_size=128, print_freq=10, save=True, save_name='w1pre_')
...
```

**Stacked Denoising Autoencoders**

```python
# Define the network
network = InputLayer(x, name='input_layer')
# denoise layer for Autoencoders
network = DropoutLayer(network, keep=0.5, name='denoising1')
# 1st layer
network = DropoutLayer(network, keep=0.8, name='drop1')
network = DenseLayer(network, n_units=800, act = tf.nn.relu, name='relu1')
x_recon1 = network.outputs
recon_layer1 = ReconLayer(network, x_recon=x, n_units=784, act = tf.nn.softplus, name='recon_layer1')
# 2nd layer
network = DropoutLayer(network, keep=0.5, name='drop2')
network = DenseLayer(network, n_units=800, act = tf.nn.relu, name='relu2')
recon_layer2 = ReconLayer(network, x_recon=x_recon1, n_units=800, act = tf.nn.softplus, name='recon_layer2')
# 3rd layer
network = DropoutLayer(network, keep=0.5, name='drop3')
network = DenseLayer(network, n_units=10, act = identity, name='output_layer')

sess.run(tf.initialize_all_variables())

# Print all parameters before pre-train
network.print_params()

# Pre-train Layer 1
recon_layer1.pretrain(sess, x=x, X_train=X_train, X_val=X_val, denoise_name='denoising1', n_epoch=100, batch_size=128, print_freq=10, save=True, save_name='w1pre_')
# Pre-train Layer 2
recon_layer2.pretrain(sess, x=x, X_train=X_train, X_val=X_val, denoise_name='denoising1', n_epoch=100, batch_size=128, print_freq=10, save=False)
# Start training
...
```

### *Convolutional Neural Network*

Instead of feeding the images as 1D vectors, the images can be imported as 4D matrix, where [None, 28, 28, 1] represents [batchsize, height, width, channels]. Set 'batchsize' to 'None' means data with different batchsize can all filled into the placeholder.

```python
x = tf.placeholder(tf.float32, shape=[None, 28, 28, 1])
y_ = tf.placeholder(tf.int64, shape=[None,])
```
**CNNs + MLP**

A 2 layers CNN followed by 2 fully connected layers can be defined by the following codes:

```python
network = InputLayer(x, name='input_layer')
network = Conv2dLayer(network,
                      act = tf.nn.relu,
                      shape = [5, 5, 1, 32],  # 32 features for each 5x5 patch
                      strides=[1, 1, 1, 1],
                      padding='SAME',
                      name ='cnn_layer1')     # output: (?, 28, 28, 100)
network = Pool2dLayer(network,
                      ksize=[1, 2, 2, 1],
                      strides=[1, 2, 2, 1],
                      padding='SAME',
                      pool = tf.nn.max_pool,
                      name ='pool_layer1',)   # output: (?, 14, 14, 100)
network = Conv2dLayer(network,
                      act = tf.nn.relu,
                      shape = [5, 5, 32, 64], # 64 features for each 5x5 patch
                      strides=[1, 1, 1, 1],
                      padding='SAME',
                      name ='cnn_layer2')     # output: (?, 14, 14, 32)
network = Pool2dLayer(network,
                      ksize=[1, 2, 2, 1],
                      strides=[1, 2, 2, 1],
                      padding='SAME',
                      pool = tf.nn.max_pool,
                      name ='pool_layer2',)   # output: (?, 7, 7, 32)
network = FlattenLayer(network, name='flatten_layer')
network = DropoutLayer(network, keep=0.5, name='drop1')
network = DenseLayer(network, n_units=256, act = tf.nn.relu, name='relu1')
network = DropoutLayer(network, keep=0.5, name='drop2')
network = DenseLayer(network, n_units=10, act = identity, name='output_layer')
```



### *Recurrent Neural Network*

**LSTM**

Long Short-Term Memory is xxxxxx xjsdjansijdni


### *Reinforcement Learning*
To understand Reinforcement Learning, a Blog (*[Deep Reinforcement Learning: Pong from Pixels](http://karpathy.github.io/2016/05/31/rl/)*) and a Paper (*[Playing Atari with Deep Reinforcement Learning](https://www.cs.toronto.edu/~vmnih/docs/dqn.pdf)*) are recommended. To play with RL, use *[OpenAI Gym](https://github.com/openai/gym)* as benchmark is recommended.

**Pong Game**

Atari Pong Game is a single agent example. *[Pong from Pixels](http://karpathy.github.io/2016/05/31/rl/)* using 130 lines of Python only *[(Code link)](https://gist.github.com/karpathy/a4166c7fe253700972fcbc77e4ea32c5)* can be reimplemented as follow.

```python
# Policy network
network = InputLayer(x, name='input_layer')
network = DenseLayer(network, n_units= H , act = tf.nn.relu, name='relu_layer')
network = DenseLayer(network, n_units= 1 , act = tf.nn.sigmoid, name='output_layer')
```


### *Cost Function*

TensorLayer provides a simple way to creat you own cost function. Take a MLP below for example.

```python
network = InputLayer(x, name='input_layer')
network = DropoutLayer(network, keep=0.8, name='drop1')
network = DenseLayer(network, n_units=800, act = tf.nn.relu, name='relu1')
network = DropoutLayer(network, keep=0.5, name='drop2')
network = DenseLayer(network, n_units=800, act = tf.nn.relu, name='relu2')
network = DropoutLayer(network, keep=0.5, name='drop3')
network = DenseLayer(network, n_units=10, act = identity, name='output_layer')
```

After initializing the variables, the informations of network parameters can be observed by using **network.print_params()**.

```python
sess.run(tf.initialize_all_variables())
network.print_params()
>> param 0: (784, 800) (mean: -0.000000, median: 0.000004 std: 0.035524)
>> param 1: (800,) (mean: 0.000000, median: 0.000000 std: 0.000000)
>> param 2: (800, 800) (mean: 0.000029, median: 0.000031 std: 0.035378)
>> param 3: (800,) (mean: 0.000000, median: 0.000000 std: 0.000000)
>> param 4: (800, 10) (mean: 0.000673, median: 0.000763 std: 0.049373)
>> param 5: (10,) (mean: 0.000000, median: 0.000000 std: 0.000000)
>> num of params: 1276810
```

The output of network is **network.outputs**, then the cross entropy can be defined as follow. Besides, to regularize the weights, the **network.all_params** contains all parameters of the network. In this case, **network.all_params** = [W1, b1, W2, b2, Wout, bout] according to param 0, 1 ... 5 shown by **network.print_params()**. Then max-norm regularization on W1 and W2 can be performed as follow.

```python
y = network.outputs
cross_entropy = tf.reduce_mean(tf.nn.sparse_softmax_cross_entropy_with_logits(y, y_))
cost = cross_entropy
cost = cost + maxnorm_regularizer(1.0)(network.all_params[0]) + maxnorm_regularizer(1.0)(network.all_params[2])
```

Instance method **network.print_layers()** prints all outputs of different layers in order. To achieve regularization on activation output, you can use **network.all_layers** which contains all outputs of different layers. If you want to use L1 penalty on the activations of first hidden layer, just simply add **tf.contrib.layers.l2_regularizer(lambda_l1)(network.all_layers[1])** to the cost function.

```python
network.print_layers()
>> layer 0: Tensor("dropout/mul_1:0", shape=(?, 784), dtype=float32)
>> layer 1: Tensor("Relu:0", shape=(?, 800), dtype=float32)
>> layer 2: Tensor("dropout_1/mul_1:0", shape=(?, 800), dtype=float32)
>> layer 3: Tensor("Relu_1:0", shape=(?, 800), dtype=float32)
>> layer 4: Tensor("dropout_2/mul_1:0", shape=(?, 800), dtype=float32)
>> layer 5: Tensor("add_2:0", shape=(?, 10), dtype=float32)
```

# Library Structure
	-tensorlayer
		- README.md	(the current directory)
		- setup.py	()
		- examples	()
			-
		- img*		(images for README.md)


# Easy to Modify
**Modifying Pre-train Behaviour**

Greedy layer-wise pretrain is an important task for deep neural network initialization, while there are many kinds of pre-train metrices according to different architectures and applications.

For example, the pre-train process of *[Vanilla Sparse Autoencoder](http://deeplearning.stanford.edu/wiki/index.php/Autoencoders_and_Sparsity)* can be implemented by using KL divergence as the following code, but for *[Deep Rectifier Network](http://www.jmlr.org/proceedings/papers/v15/glorot11a/glorot11a.pdf)*, the sparsity can be implemented by using the L1 regularization of activation output.

```python
# Vanilla Sparse Autoencoder
beta = 4
rho = 0.15
p_hat = tf.reduce_mean(activation_out, reduction_indices = 0)
KLD = beta * tf.reduce_sum( rho * tf.log(tf.div(rho, p_hat)) + (1- rho) * tf.log((1- rho)/ (tf.sub(float(1), p_hat))) )
```

For this reason, TensorLayer provides a simple way to modify or design your own pre-train metrice. For Autoencoder, TensorLayer uses **ReconLayer.*__*init__()** to define the reconstruction layer and cost function, to define your own cost function, just simply modify the **self.cost** in **ReconLayer.*__*init__()**. To creat your own cost expression please read *[Tensorflow Math](https://www.tensorflow.org/versions/master/api_docs/python/math_ops.html)*. By default, **ReconLayer** only updates the weights and biases of previous 1 layer by using **self.train_params = self.all _params[-4:]**, where the 4 parameters are [W_encoder, b_encoder, W_decoder, b_decoder]. If you want to update the parameters of previous 2 layers, simply modify **[-4:]** to **[-6:]**. 

    
```python    
ReconLayer.__init__(...):
    ...
    self.train_params = self.all_params[-4:]
    ...
	self.cost = mse + L1_a + L2_w
```

**Adding Customized Regularizer**



# Installation

**TensorFlow Installation**

This library requires Tensorflow (version >= 0.8) to be installed: *[Tensorflow installation instructions](https://www.tensorflow.org/versions/r0.9/get_started/os_setup.html)*.


**GPU Setup**

GPU-version of Tensorflow requires CUDA and CuDNN to be installed.

*[CUDA, CuDNN installation instructions](https://www.tensorflow.org/versions/r0.9/get_started/os_setup.html#optional-install-cuda-gpus-on-linux)*.
	
*[CUDA download](https://developer.nvidia.com/cuda-downloads)*.

*[CuDNN download](https://developer.nvidia.com/cudnn)*.


**TensorLayer Installation**

```python
pip install git+https://github.com/xxx/xxx.git
```

Otherwise, you can also install from source by running (from source folder):

```python
python setup.py install
```


# Ways to Contribute

TensorLayer begins as an internal repository at Imperial College Lodnon, helping researchers to test their new methods. It now encourage researches from all over the world to publish their new methods so as to promote the development of machine learning.

Your method can be merged into TensorLayer, if you can prove your method xxx

Test script with detailed descriptions is required


