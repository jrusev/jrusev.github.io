+++
title = "Hacking MNIST in 30 lines of Python"
tags = ["neural networks", "machine learning", "python", "numpy", "MNIST"]
date = "2015-12-29"
categories = ["neural networks", "machine learning", "python"]
description = "In order to better understand neural networks, I wanted to see one implemented with the minimal amount of code"
+++

In order to better understand neural networks, I wanted to see one implemented
with a minimal amount of code. I quickly found a neural network in
[11 lines of Python](http://iamtrask.github.io/2015/07/12/basic-python-network/).
It's a great little piece of code that learns the XOR function and shows the
backpropagation in action. I felt however, that I needed a slightly more complex
example - a neural net that could recognize handwritten digits from the
[MNIST dataset](http://yann.lecun.com/exdb/mnist/):

![MNIST Digits](/img/mnist_digits.png)

Michael Nielsen's
[great book](http://neuralnetworksanddeeplearning.com/) on neural networks is
built upon just this classification problem, but the simplest implementation is
[140 lines](https://github.com/mnielsen/neural-networks-and-deep-learning/blob/master/src/network.py)
of code (or 74 without the comments). So I took that code and refactored it down
to 30 lines. Here it is (GitHub
[repo](https://github.com/jrusev/simple-neural-networks/blob/master/mlp_numpy.py)):

```python
import numpy as np
import mnist

def feed_forward(X, weights):
    a = [X]
    for w in weights:
        a.append(sigmoid(a[-1].dot(w)))
    return a

def grads(X, Y, weights):
    grads = np.empty_like(weights)
    a = feed_forward(X, weights)
    delta = a[-1] - Y # cross-entropy
    grads[-1] = np.dot(a[-2].T, delta)
    for i in xrange(len(a)-2, 0, -1):
        delta = np.dot(delta, weights[i].T) * d_sigmoid(a[i])
        grads[i-1] = np.dot(a[i-1].T, delta)
    return grads / len(X)

sigmoid = lambda x: 1 / (1 + np.exp(-x))
d_sigmoid = lambda y: y * (1 - y)

(trX, trY), _, (teX, teY) = mnist.load_data(one_hot=True)

weights = [
    np.random.randn(784, 100) / np.sqrt(784),
    np.random.randn(100, 10) / np.sqrt(100)]
num_epochs, batch_size, learn_rate = 30, 10, 0.2

for i in xrange(num_epochs):
    for j in xrange(0, len(trX), batch_size):
        X, Y = trX[j:j+batch_size], trY[j:j+batch_size]
        weights -= learn_rate * grads(X, Y, weights)
    prediction = np.argmax(feed_forward(teX, weights)[-1], axis=1)
    print i, np.mean(prediction == np.argmax(teY, axis=1))
```

If you run this code, you will get to about 97.8% accuracy on the test set.

![Neural Network Training](/img/nn_training.png)

The algorithm is practically the same as the original implementation. The net
has one hidden layer with 100 neurons and uses mini batch gradient descent
to learn the weights. It has two dependencies - `numpy`, which handles vector
and matrix operations, and `mnist` which is a simple
[script](https://github.com/jrusev/simple-neural-networks/blob/master/mnist.py)
that downloads the MNIST data and loads it to memory.

The major difference is that instead of loops I use matrix operations, which
means you can process the entire mini batch as a matrix instead of a single
training example. As a result, the network is about 5 times faster than the
original code, while you can still see what's going on under the hood. I've also
dropped the bias terms because in this problem they do not considerably improve
accuracy.

### Implementation with Theano

Once we understand how backpropagation works, we can hide it by using a library
such as Theano or Torch. So I implemented the exact same network as above using
Theano also in 30 lines of code. Theano can calculate the gradients
automatically after we compose the network graph. It's also about 2 times faster
than the numpy-only implementation. Here is the code
([source](https://github.com/jrusev/simple-neural-networks/blob/master/mlp_theano.py)):

```python
import theano
import theano.tensor as T
import numpy as np
import mnist

def init_weights(n_in, n_out):
    weights = np.random.randn(n_in, n_out) / np.sqrt(n_in)
    return theano.shared(np.asarray(weights, dtype=theano.config.floatX))

def feed_forward(X, w_h, w_o):
    h = T.nnet.sigmoid(T.dot(X, w_h))
    return T.nnet.softmax(T.dot(h, w_o))

(trX, trY), _, (teX, teY) = mnist.load_data(one_hot=True)

w_h, w_o = init_weights(784, 100), init_weights(100, 10)
num_epochs, batch_size, eta = 30, 10, 0.2

X, Y = T.fmatrices('X', 'Y')
out = feed_forward(X, w_h, w_o)
cost = T.nnet.categorical_crossentropy(out, Y).mean()

weights = [w_h, w_o]
grads = T.grad(cost=cost, wrt=weights)
apply_grads = theano.function(
    inputs=[X, Y],
    updates=[[w, w - g * eta] for w, g in zip(weights, grads)],
    allow_input_downcast=True)
predict = theano.function(inputs=[X], outputs=T.argmax(out, axis=1))

for i in range(num_epochs):
    for j in xrange(0, len(trX), batch_size):
        apply_grads(trX[j:j+batch_size], trY[j:j+batch_size])
    print i, np.mean(predict(teX) == np.argmax(teY, axis=1))
```
