# Image Classification


- Assigning an input image one label from a fixed set of categories
- Many other CV tasks (object detection, segmentation) reduced to image
  segmentation

Ex: Image classification model taking a single image to assign probability to
4 labels {cat, dog, hat, mug}
- computer image is 3D array (W x H x RGB)
- So 240 x 400 x 3 image = 297,600 numbers
- Each number is an integer from 0 (black) to 255 (white)
- Task is to take these 297600 numbers into a single label such as 'cat'

Challenges
- Viewpoint variation
- Scale variation
- Deformation (many objects are not rigid)
- Occlusion (only a small portion visible)
- Illumination conditions
- Background clutter (object may blend into their environment)
- Intra-class variation (many types of chairs, etc)

Good classification model must be invariant to the cross product of all these
variations, while simultaneously retaining sensitivity to the inter-class
variations

Data-driven approach
- Give computer many examples of each class
- Training dataset

Image classfication pipeline (Array of pixels (single image) and assign a label)
- Input: set of N images, each labeled with one of K different classes (training
  set)
- Learning: training a classifier / learning a model
- Evaluation: for a new set of images the classifier has never seen before.
  Hope that a lot of predictions match up with true answers (ground truth)

Hyperparameters: distance functions

Training Set
Validation Set (fake test set, small subset of training set)
Test Set (only use once, at the end)

Cross-validation
- Small training data
- Randomly pick validation set and the rest training set
- Try multiple validation sets
- 5-fold cross validation (divide into equal parts; 1 part validation, 4 parts
  training; repeat 5 times changing the validation set)
- Average performance across the 5 different folds

# Linear Classification

Score function - map raw image pixels to class scores (eg, linear function)

Loss function - measures quality of particular set of parameters based on how well the induced scores agreed with ground truth labels in the training data

# Optimization 1

Gradient Descent (Vanilla)

```
while True:
    weights_grad = evaluate_grandient(loss_fun, data, weights)
    weights += - step_size * weights_grad # perform parameter update
```

Simple loop is core of all NN libraries.  Gradient Descent is by far the most common optimization.

Mini-batch Gradient Descent
- Size of mini-batch is a hyperparameter usually based on memory constraints
- Or set some value (eg, 32, 64, 128), use power of 2 because many vectorized operation implementations work faster when their inputs are  sized in powers of 2
```
while True:
    data_batch = sample_training_data(data, 256) # sample 256 examples
    weights_grad = evaluate_gradient(loss_fun, data_batch, weights)
    weights += - stepsize * weights_grad # perform parameter updates
```

Stochastic Gradient Descent (SGD)
- Extreme case where mini-batch contains only single example
- AKA on-line gradient descent

# Backpropagation

- Way of computing gradients of expressions through recursive application of chain rule


Problem Statement: Given some function f(x) where x is a vector of inputs and we
are interested in the computing gradient of f at x

Motivation
- f will correspond to the loss function (L) and the inputs x will consist of
  training data and the neural network weights

Gradient calculations

f(x,y) = xy -> df/dx = y and df/dy = x

Compound expressions with chain rule

```
# set some inputs
x = -2; y = 5; z = -4

# perform forward pass
q = x + y # q becomes 3
f = q * z # f becomes -12

# perform backward pass (backprop) in reverse order:
# first backprop through f = q * z
dfdz = q # df/dz = q, so gradient on z becomes 3
dfdq = z # df/dq = z, so gradient on q becomes -4

# now backprop through q = x + y
dfdx = 1.0 * dfdq # dq/dx = 1. And the multiplication here is the chain rule
dfdy = 1.0 * dfdq # dq/dy = 1

This extra multiplcation (for each input) due to the chain rule can turn
a single and relatively useless gate into a cog into a complex circuit such as
an entire neural network

Patterns in backward flow
- add gate
- max gate
- multiply gate

# Neural Networks 1

```
class Neuron(object):
  def forward(inputs):
  """ assume inputs and weights are 1D numpy arrays and bias isa number """
  cell_body_sum = np.sum(inputs * self.weights) + self.bias
  firing_rate = 1.0 / (1.0 + math.exp(-cell_body_sum)) # sigmoid activation
  return firing_rate
```

## Single neuron as a linear classifier

Binary Softmax classifier (AKA logistic regression prediction based on whether
neuron is greater than 5)

Binary SVM

Regularization interpretation

## Commonly used activation functions

Sigmoid

Tanh

*ReLU* - be careful of learning rates and % dead units

Leaky ReLU

Maxout

## NN Architectures

Most common layer type is fully-connected layer

Naming conventions: N-layer neural network (exclude input layer)

Output layer: Most commonly doesn't have activation function

Sizing:
- Number of neurons
- Number of parameters (weights + biases)

## Example

```
# forward-pass of a 3-layer neural network
f = lambda x: 1.0/(1.0 + np.exp(-x)) # activation function (use sigmoid)
x = np.random.randn(3, 1) # random input vector of three numbers (3x1)
h1 = f(np.dot(W1, x) + b1) # calculate 1st hidden layer activations (4x1)
h2 = f(np.dot(W2, h1) + b2) # calculate 2nd hidden layer activations (4x1)
out = np.dot(W3, h2) + b3 # output neuron (1x1)
```

# NN 2

## Setting up the data and the model

Data Preprocessing

- Mean subtraction
- Normalization
- PCA and whitening

Mean must only be computed on the training data, and then applied to the
validation / test data

## Weight Initialization

Pitfall: all zero initialization

Small random numbers

Calibrating the variances with 1/sqrt(n)

Sparse initialization

Initializing the biases

IN PRACTICE: current recommendation is to use ReLU units and use the

wp = np.random.randn(n) * sqrt(2.0/n)


Batch Normalization

## Regularization

Controlling the capacity of NN to prevent overfitting

L2 regularization
- Most common
- Penalizing the squared magnitude of all parameters directly in the objective
- Add 1/2 * lambda * w^2 to objective, where lambda is regularization strength
- 1/2 useful because gradient of this term is 2 * lambda * w
- Intuitively: heavily penalizing peaky weight vectors and preferring diffuse
  weight vectors.
- Due to multiplicative interactions between weights and inputs, this has the
  appealing property of encouraging the network to use all of its inputs
  a little than some of its inputs a lot.
- Notice gradient descent parameter update, using L2 regularization ultimately
  means that every weight is decayed linearly: W += -lambda * W towards 0

L1 regularization
- Add term lambda * |w| to the objective
- Can combine L1 and L2: lambda_1 * |w| + lambda_2 * w^2 (called elastic
  regularization)
- L1 leads weight vectors to become sparse during optimization (very close to
  exactly zero)
- Neurons with L1 regularization end up using only a sparse subset 
of their most important inputs and become nearly invariant to 'noisy' inputs
- L2 usually better than L1

Max norm constraints
- Enforce absolute upper bound on the magnitude of weight vector for every
  neuron and use projected gradient descent to enforce the constrant
- Corresponds to performing the parameter update as normal, and then enforcing
  the constraint by clamping the weight vector w of every neuron to satisfy
  ||w||2 < c 
- Typical values of c are on orders of 3 or 4
- Some people report improvements when using this form of regularization
- Network cannot 'explode' even when learning rates are set too high because
the learning rates are set too high because updates are always bounded

Dropout
- Complements other regularization techniques
- Dropout implemented by only keeping a neuron active with some probability
  p (hyperparameter), or setting it to zero otherwise
- Use inverted dropout, to have better forward pass performance during 
test time, and prediction code can remain untouched

```
# Inverted Dropout

```

Theme of noise in forward pass
- Introduce stochastic behavior in the forward pass of the network
- During testing, the noise is marginalized over analytically or numerically by
  sampling, by performing several forward passes with different random decisions
  and then averaging over them

Bias regularization

Per-layer regularization

IN PRACTICE: 
- Use single, global L2 regularization strength that is cross-validated
- Common to combine this with dropout applied after all layers
- Value of p = 0.5 is reasonable default, but can be tuned on validtion data

## Loss functions

Classification
Regression
Structured Prediction
- Labels can be arbitrary structures such as graphs, trees, and other complex
  objects
- Usually ssumes that the space of structures is very large and not 
easily enumerable


# NN 3

## Learning parameters and finding good hyperparameters

### Gradient checks

Performing a gradient check is as simple as comparing the analytic gradient to
the numerical gradient

- Use the centered formula
- Use relative error for the comparison
- Use double precision
- Stick around active range of a floating point
- Kinks in the objective (non-differentiable parts of an objective fn introduced
  by functions such as ReLU. SVM loss, Maxout neurons, etc)
- Use only a few datapoints
- Be careful with the step size h (to avoid numerical precision problems)
- Gradcheck during a 'characteristic' mode of operation
- Don't let the regularization overwhelm the data
- Remember to turn off dropout/augmentations
- Check only few dimensions


### Before learning sanity checks

- Look for correct loss at chance performance (make sure you're getting the loss
  you expect when you initialize with small parameters; better to check the data
  loss alone so set regularization strength to zero)
- Overfit a tiny subset of data: Before training on the full dataset, try to
  train a tiny portion (eg, 20 examples) of your data and make sure you can
  achieve zero cost.  For this experiment it's also best to set regularization
  to zero, otherwise this can prevent you from getting zero cost.  Note it may
  happen that you can overfit a very small dataset but still have an incorrect
  implementation.

### Babysitting the learning process

Loss function
- First quantity to track
- Evaluated on the individual batches during the forward pass

Train/Val accuracy
- Plot can give insights into amount of overfitting in your model

Ratio of weights:updates

Activation / Gradient distributions per layer

First-layer visualizations

## Parameter updates

### SGD and bells and whistles

Vanilla update

```
x += - learning_rate * dx
```

Momentum update

```
v = mu * v - learning_rate * dx # integrate velocity
x += v
```

mu = hyperparameter (momentum)

With momentum update, the parameter vector will build up velocity in any
direction that has consistent gradient

Nesterov momentum

### Annealing the learning rate

Usually helpful to anneal the learning rate over time

Decay it slowly and you'll be wasting computation bouncing around chaotically
with little improvement over time

But decay too aggressively and the system will cool too quickly, unable to reach
the best position it can

Three types of learning rate decay
1. Step decay: reduce learning rate by some factor every few epochs
2. Exponential decay
3. 1/t decay

### Second order methods

### Per parameter adaptive learning rate methods


## Hyperparameter optimization

Most common hyperparameters:
- initial learning rate
- learning rate decay schedule (eg, decay constant)
- regularization strength (L2 penalty, dropout strength)

Workers and master

Prefer one validation to cross-validation

Hyperparameter ranges: search for hyperparameters on log scale

Prefer random search to grid search

Careful with best values on border

Stage your search from coarse to fine

Bayesian hyperparameter optimization

## Model evaluation

### Model ensembles

Train multiple independent models, and at test time average their predictions

As the number of models in the ensemble increases, the performance typically
monotonically improves (though with diminishing returns)


#### Approaches

Same model, different initialization

Top models discovered during cross-validation

Different checkpoints of a single model

Running average of parameters during training


Dark Knowledge: distill a good ensemble back to a single model by incorporating
the ensemble log likelihoods into a modified objective





