#ConvolutionalNeuralNetwork
==========================

An API for a convolutional neural network implemented in C++ with the intent to increase and assist research on architectures of neural nets

##Static Library
==========================

The build and .h files for referencing as an external static library can be found in the Builds folder.


##What a Convolutional Neural Network is
==========================

Our brains work by a large web of connected neurons, or simple binary states. These neurons are connected by synapses, which have a strength associated with them. When a neuron fires, it's signal is sent through all of it's connecting synapses to other neurons to determine their value. When we learn, our brain adjusts the strengths of the associated synapses to limit the amount of activated neurons.

A neural network is a machine learning algorithm based off of the brain. Within a network, there are layers. Each of these layers has a number of neurons, which take on floating point values, and weights, symbolic of synapses, attached to the neurons in the next layer. These networks then run in a way similar to our brains, given an input, all neurons are fed forward to the next layer by summing the value of the neurons times the weights connecting two neurons. Commonly, a bias is an addition to the network which is used as a simple shift to neurons. The bias is added to the sum of the weights times the neurons to produce the output of the neuron, which is then commonly ran through a continuous activation function, such as a sigmoid, to bound the value of the neuron as well as give the network a differentiable property.

Weights can be connected between neurons in different ways. Most common are full connectivity layers and shared weight layers. Full connectivity layers have weights going from every input neuron to every output neuron, so every neuron in the layers are connected to every neuron in the layers above. Shared weights are a way of forming similar connections between different neurons by a common weight pattern. A common implementation of this is convolutional layers.

Convolutional layers make use of mathematical convolution, an operation used to produce feature maps, or highlights from an image. Convolution is formally defined as the sum of all values in the domains of two functions which are multiplied by one another. In real life cases, this is commonly discrete, and is most easily understood in images. Image convolution involves iterating a mask over an image to produce an output, where the output pixel values are equivelant to the sum of the mask multiplied by neighboring pixels in the input when anchored at the center of the mask. This operation draws features from the image, such as edges or curves, and is associated with the way our visual cortex processes imagery.

Networks learn through different algorithms, although the two implemented here are the up-down or wake-sleep algorithm and vanilla backpropagation. Backpropagation is an algorithm which computes the derivatives of the error with respect to the weights, and adjusts the weights in order to find a minimum in the error function. This is a way of approximating the actual error signal of every neuron, so a small step size is often used to prevent divergence. The wake-sleep or up-down algorithm trains the network without knowledge of data in an encoder-decoder format. The layers in the network are fed forward, backwards, and forwards again, before a difference is calculated to adjust the weights. 

##How this API is implemented
=================================

This API is based off of template meta-programming to optimize efficiency. Therefore, much of this API is based on the assumption that a network architecture will be defined at compile time rather than runtime. 

##Documentation
===============================

###Macros
===============================

These macros are used to signify layer types and activation functions. They are prefixed with `CNN_*`. Their name should explain their use. The available layers can be found below.

Available activation functions are linear (y = x), sigmoid (y = 1/(1 + exp(-x)), bipolar sigmoid (y = 2/(1 + exp(-x)) - 1), tanh (y = tanh), and rectified linear (y = max(0, x)).

Available cost functions are quadratic, cross entropy, and log loss.

###IMatrix
===============================

This class is merely a container for `Matrix2D<T, int, int>` so that matrix sizes "unknown" at compile time can be computed at runtime.

###Matrix2D<T, int, int>
===============================

This class is a simple matrix implementation, with some extra methods that can be used in situations outside of this neural network.

| Member/Method  | Type | Details |
|---------|------|---------|
| `data` | `std::array<T, rows * cols>` | holds the matrice's data in column major format |
| `at(int i, int j)` | `T` | returns the value of the matrix at i, j |
| `clone()` | `Matrix2D<T, rows, cols>` | creates a deep copy of the matrix |
| `rows()` | `int` | returns the amount of rows |
| `cols()` | `int` | returns the amount of cols |

<small>This table contains methods used only in the source code of the network</small>

###ILayer
===============================

This is the interface for all of the various layer types used in the network.

| Member/Method | Type | Details |
|--------|------|----------|
| `feature_maps` | `std::vector<IMatrix<float>*>` | Holds the data of the network |
| `recognition_weights` | `std::vector<IMatrix<float>*>` | The feed forwards weights |
| `generation_weights` | `std::vector<IMatrix<float>*>` | The feed backwards weights |
| `feed_forwards(std::vector<IMatrix<float>*> &output)` | `virtual void` | Feeds the layer forward |
| `feed_backwards(std::vector<IMatrix<float>*> &input, bool use_g_weights)` | `virtual std::vector<IMatrix<float>*>` | Feeds the layer backwards using generative or recognition weights |
| `wake_sleep(bool binary_net)` | `void` | Performs the wake-sleep (up-down) algorithm with the specified activation method |
| `backprop(std::vector<IMatrix<float>*> &data, &deriv, std::vector<IMatrix<float>*> &weight_gradient, &bias_gradient)` | `void` | Performs vanilla backpropagation witht the specified activation method |

###PerceptronFullConnectivityLayer<int features, int rows, int cols, int out_rows, int out_cols, int out_features, int activation_function>
===============================

Basic perceptron layer. Interprets architecture as a single dimension array.

Overloaded Methods

| Method | Difference |
|----------|-------------|
| `feed_forwards` | Uses standard sums for feeding forwards |
| `feed_backwards` | Uses standard sums for feeding backwards |

###ConvolutionLayer<int features, int rows, int cols, int recognition_data_size, int stride, int out_features, int activation_function>
===============================

Basic convolutional layer, masks or kernels must be square and odd.

Overloaded Methods

| Method | Difference |
|----------|-------------|
| `feed_forwards` | Uses convolution for feeding forwards |
| `feed_backwards` | Uses convolution for feeding backwards |

###MaxpoolLayer<int features, int rows, int cols, int out_rows, int out_cols>
===================================

Basic maxpooling layer.

Overloaded Methods

| Method | Difference |
|----------|-------------|
| `feed_forwards` | Uses maxpooling for feeding forwards |
| `feed_backwards` | N/A |

###SoftMaxLayer<int features, int rows, int cols>
=====================================

Basic softmax layer. This will compute derivatives for any cost function, not just log-likelihood. Softmax is performed on each feature map independently.

Overloaded Methods

| Method | Difference |
|----------|-------------|
| `feed_forwards` | N/A |
| `feed_backwards` | N/A |


###OutputLayer<int features, int rows, int cols>
=====================================

Basic output layer just to signify the end of the network.

Overloaded Methods

| Method | Difference |
|----------|-------------|
| `feed_forwards` | N/A |
| `feed_backwards` | N/A |

###NeuralNetwork
===============================

This is the class that encapsulates all of the rest. Has all required methods. Will add support for other error functions later.

| Member/Method | Type | Details |
|--------|------|----------|
| `learning_rate` | `float` | The learning term of the network. Default value is 0 |
| `momentum_term` | `float` | The momentum term (proportion of learning rate when applied to momentum) of the network. Normally between 0 and 1. Default value is 0 |
| `cost_function` | `int` | The cost function to be used |
| `use_batch_learning` | `bool` | Whether you will apply gradient manually |
| `use_dropout` | `bool` | Whether to train the network with dropout |
| `use_momentum` | `bool` | Whether to train the network with momentums |
| `weight_gradient` | `std::vector<std::vector<IMatrix<float>*>>` | The gradient for the weights |
| `bias_gradient` | `std::vector<std::vector<IMatrix<float>*>>` | The gradient for the biases |
| `layers` | `std::vector<ILayer*>` | All of the network's layers |
| `labels` | `std::vector<IMatrix<float>*>` | The current labels |
| `input` | `std::vector<IMatrix<float>*>` | The current input |
| `add_layer(ILayer* layer)` | `void` | Adds another layer to the network |
| `setup_gradient()` | `void` | Initializes the network to learn. Must call if learning |
| `apply_gradient()` | `void` | Updates weights |
| `apply_gradient(std::vector<std::vector<IMatrix<float>*>> &weights, &biases)` | `void` | Updates weights with custom gradients (use in parallelization) |
| `save_data(std::string path)` | `void` | Saves the data |
| `load_data(std::string path)` | `void` | Loads the data (<b>Must have initialized network and filled layers first!!!</b>) |
| `set_input(std::vector<IMatrix<float>*> input)` | `void` | Sets the current input |
| `set_labels(std::vector<IMatrix<float>*> labels)` | `void` | Sets the current labels |
| `discriminate()` | `void` | Feeds the network forward |
| `pretrain()` | `void` | Pretrains the network using the wake-sleep algorithm |
| `train(int iterations)` | `void` | Trains the network using backpropogation |
| `train(int iterations, std::vector<std::vector<IMatrix<float>*>> &weights, &biases)` | `void` | Trains the network using backpropogation with custom gradients (use in parallelization) |


###NeuralNetAnalyzer

This is a singleton static class. This class helps with network analysis, such as the mean squared error (MSE), and finite difference backprop checking.

| Member/Method | Type | Details |
|--------|------|----------|
| `sample_size` | `static int` | The sample size used to calculate the MSE |
| `approximate_weight_gradient(NeuralNet &net)` | `static std::vector<std::vector<IMatrix<float>*>>` | Uses finite differences for backprop checking |
| `approximate_bias_gradient(NeuralNet &net)` | `static std::vector<std::vector<IMatrix<float>*>>` | Uses finite differences for backprop checking |
| `mean_gradient_error(NeuralNet &net, std::vector<std::vector<IMatrix<float>*>> &observed_weight_gradient, &observed_bias_gradient)` | `static std::pair<float, float>` | Uses finite differences for backprop checking, returns mean difference in ordered pair (weights, biases) |
| `add_point(float value)` | `static void` | Adds a point for the running calculation of the MSE |
| `mean_squared_error()` | `static float` | Returns the running MSE |
| `save_mean_square_error(std::string path)` | `static void` | Saves all calculated MSEs |


#Usage
===============================

For an example of creating and using a network, see main.cpp.
