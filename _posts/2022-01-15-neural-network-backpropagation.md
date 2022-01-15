---
layout: post
title: Forward and backward propagation for a simple neural network
category: neural
math: true
comments: true
---

This post shows you how to construct the forward propagation and backpropagation algorithms for a simple neural network.  It may be helpful to you if you've just started to learn about neural networks and want to check your logic regarding the maths.

Consider the neural network below.  It has two inputs (x<sub>1</sub> and x<sub>2</sub>), two hidden neurons (h<sub>1</sub> and h<sub>2</sub>), one output neuron (o<sub>1</sub>) and a series of weights and biases.

<img src="/assets/images/blog/nn/nn-1.png" alt="Our neural network">

We want to use this neural network to predict outputs and have four training examples which show the expected output y for different combinations of x<sub>1</sub> and x<sub>2</sub>.

| x<sub>1</sub> | x<sub>2</sub> | y |
|----|----|---|
| 0  | 0  | 0 |
| 0  | 1  | 1 |
| 1  | 0  | 1 |
| 1  | 1  | 1 |

Our neural network starts with a random set of weights between 0 and 1, and biases set to 0.  In this initial configuration, as you might expect, our neural network will predict outputs which bear no relation to the expected outputs.

To improve the prediction accuracy, we need to optimise the weights and biases so that they work together to predict outputs that are as close as possible to the expected outputs.

This optimisation process is called training and it involves two stages: forward propagation and backpropagation.

# Forward propagation

The forward pass stage moves through the neural network from left to right.

Each neuron is configured to sum its weighted inputs with its bias and to then pass that value z through an activation function $$\sigma$$ to produce an output a.

<img src="/assets/images/blog/nn/nn-2.png" alt="Our neural network with z and a for each neuron">

We can express the sum z and output a for each neuron as:

$$
\begin{aligned}
z_{h1} &= w_1x_1 + w_3x_2 + b_1 \\
a_{h1} &= \sigma(z_{h1}) \\ \\
z_{h2} &= w_2x_1 + w_4x_2 + b_2 \\
a_{h2} &= \sigma(z_{h2}) \\ \\
z_{o1} &= w_5a_{h1} + w_6a_{h2} + b_3 \\
a_{o1} &= \sigma(z_{o1})
\end{aligned}
$$

The activation function $$\sigma$$ is used to transform a continuous input into an output between 0 and 1, with 0 for large negative inputs and 1 for large positive inputs.

Here we'll apply the commonly-used [Sigmoid function](https://en.wikipedia.org/wiki/Sigmoid_function) for activation:

$$ \sigma(z) = \frac{\mathrm{1} }{\mathrm{1} + e^{-z}}  $$

<img src="/assets/images/blog/nn/sigmoid.png" alt="Sigmoid activation function">

With these equations, we can use the x<sub>1</sub> and x<sub>2</sub> from each of the training examples to predict an output (a<sub>o1</sub>).

To measure the prediction accuracy of our neural network across all of the training examples, we can the [Mean Squared Error function](https://en.wikipedia.org/wiki/Mean_squared_error) to find the total loss J:

$$ \displaystyle \text{Total loss, J} = \frac{1}{2m}\sum_{i=1}^{m}(a_{o1} - y)^2 $$

where m is the total number of training examples; in our case, m = 4.

To begin, the total loss J is almost certain to be greater than zero unless we have been particularly fortunate with our random weights and biases.

More likely we will need to optimise the weights and biases and to do that, we use backpropagation.

# Backpropagation

Backpropagation moves backwards through the neural network, from right to left.

Using equation from above for the total loss J, if we consider just a single training example with m = 1, we can define a loss L:

$$ \displaystyle \text{Loss, L} = \frac{1}{2}(a_{o1} - y)^2 $$

The loss L is dependent on a<sub>o1</sub> which itself is dependent on all of the weights and biases. It follows that optimising these weights and biases will reduce the loss L.

Using an iterative process, we can optimise each weight and bias using the principle of [gradient descent](https://saugatbhattarai.com.np/what-is-gradient-descent-in-machine-learning/) and a suitable learning rate $$ \alpha $$:

$$
\begin{aligned}
w_s &= w_s - \alpha\dfrac{\mathrm{d}L}{\mathrm{d}w_s} \text{  where s = 1, 2, ..., 6} \\ \\
b_t &= b_t - \alpha\dfrac{\mathrm{d}L}{\mathrm{d}b_t} \text{  where t = 1, 2, 3}
\end{aligned}
$$

To use these equations, we first need expressions for the derivative of the loss L with respect to each weight and bias.  These expressions are partial derivatives which we can derive using the chain rule.

Before we start, there are two derivatives that we will come in handy.  The first is the derivative of the Sigmoid activation function:

$$
\begin{aligned}
\sigma(z) &= \frac{\mathrm{1} }{\mathrm{1} + e^{-z}}  \\ \\
\dfrac{\mathrm{d}\sigma(z)}{\mathrm{d}z} &= \sigma(z)(1-\sigma(z))
\end{aligned}
$$

The second is the derivative of the loss L with respect to a<sub>o1</sub>:

$$
\begin{aligned}
\displaystyle L &= \frac{1}{2}(a_{o1} - y)^2 \\ \\
\dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}} &= (a_{o1} - y)
\end{aligned}
$$

Let's now start with weight w<sub>6</sub> and find the derivative of loss L with respect to w<sub>6</sub>:

$$ \dfrac{\mathrm{d}L}{\mathrm{d}w_6} $$

Using the chain rule, we can say that:

$$ \dfrac{\mathrm{d}L}{\mathrm{d}w_6} = \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}w_6} $$

Therefore:

$$ \dfrac{\mathrm{d}L}{\mathrm{d}w_6} =  (a_{o1} - y).\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}w_6} $$

This is progress but we need to go further with:

$$ \dfrac{\mathrm{d}a_{o1}}{\mathrm{d}w_6} $$

Using the chain rule again, we can say:

$$ \dfrac{\mathrm{d}a_{o1}}{\mathrm{d}w_6} = \dfrac{\mathrm{d}a_{o1}}{\mathrm{d}z_{o1}}.\dfrac{\mathrm{d}z_{o1}}{\mathrm{d}w_6} $$

where:

$$ \dfrac{\mathrm{d}a_{o1}}{\mathrm{d}z_{o1}} = \dfrac{\mathrm{d}\sigma(z_{o1})}{\mathrm{d}z_{o1}} = \sigma(z_{o1})(1-\sigma(z_{o1})) $$

and:

$$ \dfrac{\mathrm{d}z_{o1}}{\mathrm{d}w_6} = \dfrac{\mathrm{d}(w_5a_{h1} + w_6a_{h2} + b_3)}{\mathrm{d}w_6} = a_{h2} = \sigma(z_{h2}) $$

Combining these partial derivatives, we get an expression for w<sub>6</sub> that can be computed:

$$
\begin{aligned}
\dfrac{\mathrm{d}L}{\mathrm{d}w_6} &= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}w_6} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}z_{o1}}.\dfrac{\mathrm{d}z_{o1}}{\mathrm{d}w_6} \\ \\
&= (a_{o1} - y).\sigma(z_{o1})(1-\sigma(z_{o1})).\sigma(z_{h2})
\end{aligned}
$$

Using the same approach for w<sub>5</sub> gives:

$$
\begin{aligned}
\dfrac{\mathrm{d}L}{\mathrm{d}w_5} &= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}w_5} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}z_{o1}}.\dfrac{\mathrm{d}z_{o1}}{\mathrm{d}w_5} \\ \\
&= (a_{o1} - y).\sigma(z_{o1})(1-\sigma(z_{o1})).\sigma(z_{h1})
\end{aligned}
$$

For b<sub>3</sub>:

$$
\begin{aligned}
\dfrac{\mathrm{d}L}{\mathrm{d}b_3} &= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}b_3} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}z_{o1}}.\dfrac{\mathrm{d}z_{o1}}{\mathrm{d}b_3} \\ \\
&= (a_{o1} - y).\sigma(z_{o1})(1-\sigma(z_{o1}))
\end{aligned}
$$

which takes a shorter form because:

$$ \dfrac{\mathrm{d}z_{o1}}{\mathrm{d}b_3} = \dfrac{\mathrm{d}(w_5a_{h1} + w_6a_{h2} + b_3)}{\mathrm{d}b_3} = 1 $$

Next, we need to repeat the exercise with weights w<sub>4</sub>, w<sub>3</sub>, w<sub>2</sub>, w<sub>1</sub> and biases b<sub>2</sub> and b<sub>1</sub>.

Let's start with weight w<sub>4</sub> and find the derivative of loss L with respect to w<sub>4</sub>:

$$ \dfrac{\mathrm{d}L}{\mathrm{d}w_4} $$

Using the chain rule, we can say that:

$$ \dfrac{\mathrm{d}L}{\mathrm{d}w_4} = \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}w_4} $$

Therefore:

$$ \dfrac{\mathrm{d}L}{\mathrm{d}w_4} =  (a_{o1} - y).\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}w_4} $$

Once again, this is progress but we need to go further with:

$$ \dfrac{\mathrm{d}a_{o1}}{\mathrm{d}w_4} $$

Using the chain rule again, we can say:

$$ \dfrac{\mathrm{d}a_{o1}}{\mathrm{d}w_4} = \dfrac{\mathrm{d}a_{o1}}{\mathrm{d}a_{h2}}.\dfrac{\mathrm{d}a_{h2}}{\mathrm{d}w_4} $$

where:

$$ \dfrac{\mathrm{d}a_{o1}}{\mathrm{d}a_{h2}} = \dfrac{\mathrm{d}a_{o1}}{\mathrm{d}z_{o1}}.\dfrac{\mathrm{d}z_{o1}}{\mathrm{d}a_{h2}} = \dfrac{\mathrm{d}\sigma(z_{o1})}{\mathrm{d}z_{o1}}.\dfrac{\mathrm{d}(w_5a_{h1} + w_6a_{h2} + b_3)}{\mathrm{d}a_{h2}} = \sigma(z_{o1})(1-\sigma(z_{o1})).w_6 $$

and:

$$ \dfrac{\mathrm{d}a_{h2}}{\mathrm{d}w_4} = \dfrac{\mathrm{d}a_{h2}}{\mathrm{d}z_{h2}}.\dfrac{\mathrm{d}z_{h2}}{\mathrm{d}w_4} = \dfrac{\mathrm{d}\sigma(z_{h2})}{\mathrm{d}z_{h2}}.\dfrac{\mathrm{d}(w_2x_1 + w_4x_2 + b_2)}{\mathrm{d}w_4} = \sigma(z_{h2})(1-\sigma(z_{h2})).x_2 $$

Finally for w<sub>4</sub>:

$$
\begin{aligned}
\dfrac{\mathrm{d}L}{\mathrm{d}w_4} &= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}w_4} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}a_{h2}}.\dfrac{\mathrm{d}a_{h2}}{\mathrm{d}w_4} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}z_{o1}}.\dfrac{\mathrm{d}z_{o1}}{\mathrm{d}a_{h2}}.\dfrac{\mathrm{d}a_{h2}}{\mathrm{d}z_{h2}}.\dfrac{\mathrm{d}z_{h2}}{\mathrm{d}w_4} \\ \\
&= (a_{o1} - y).\sigma(z_{o1})(1-\sigma(z_{o1})).w_6.\sigma(z_{h2})(1-\sigma(z_{h2})).x_2
\end{aligned}
$$

Using the same approach for w<sub>3</sub> gives:

$$
\begin{aligned}
\dfrac{\mathrm{d}L}{\mathrm{d}w_3} &= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}w_3} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}a_{h1}}.\dfrac{\mathrm{d}a_{h1}}{\mathrm{d}w_3} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}z_{o1}}.\dfrac{\mathrm{d}z_{o1}}{\mathrm{d}a_{h1}}.\dfrac{\mathrm{d}a_{h1}}{\mathrm{d}z_{h1}}.\dfrac{\mathrm{d}z_{h1}}{\mathrm{d}w_3} \\ \\
&= (a_{o1} - y).\sigma(z_{o1})(1-\sigma(z_{o1})).w_5.\sigma(z_{h1})(1-\sigma(z_{h1})).x_2
\end{aligned}
$$

For w<sub>2</sub>:

$$
\begin{aligned}
\dfrac{\mathrm{d}L}{\mathrm{d}w_2} &= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}w_2} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}a_{h2}}.\dfrac{\mathrm{d}a_{h2}}{\mathrm{d}w_2} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}z_{o1}}.\dfrac{\mathrm{d}z_{o1}}{\mathrm{d}a_{h2}}.\dfrac{\mathrm{d}a_{h2}}{\mathrm{d}z_{h2}}.\dfrac{\mathrm{d}z_{h2}}{\mathrm{d}w_2} \\ \\
&= (a_{o1} - y).\sigma(z_{o1})(1-\sigma(z_{o1})).w_6.\sigma(z_{h2})(1-\sigma(z_{h2})).x_1
\end{aligned}
$$

For w<sub>1</sub>:

$$
\begin{aligned}
\dfrac{\mathrm{d}L}{\mathrm{d}w_1} &= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}w_1} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}a_{h1}}.\dfrac{\mathrm{d}a_{h1}}{\mathrm{d}w_1} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}z_{o1}}.\dfrac{\mathrm{d}z_{o1}}{\mathrm{d}a_{h1}}.\dfrac{\mathrm{d}a_{h1}}{\mathrm{d}z_{h1}}.\dfrac{\mathrm{d}z_{h1}}{\mathrm{d}w_1} \\ \\
&= (a_{o1} - y).\sigma(z_{o1})(1-\sigma(z_{o1})).w_5.\sigma(z_{h1})(1-\sigma(z_{h1})).x_1
\end{aligned}
$$

For b<sub>2</sub>:

$$
\begin{aligned}
\dfrac{\mathrm{d}L}{\mathrm{d}b_2} &= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}b_2} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}a_{h2}}.\dfrac{\mathrm{d}a_{h2}}{\mathrm{d}b_2} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}z_{o1}}.\dfrac{\mathrm{d}z_{o1}}{\mathrm{d}a_{h2}}.\dfrac{\mathrm{d}a_{h2}}{\mathrm{d}z_{h2}}.\dfrac{\mathrm{d}z_{h2}}{\mathrm{d}b_2} \\ \\
&= (a_{o1} - y).\sigma(z_{o1})(1-\sigma(z_{o1})).w_6.\sigma(z_{h2})(1-\sigma(z_{h2}))
\end{aligned}
$$

which takes a shorter form because:

$$ \dfrac{\mathrm{d}z_{h2}}{\mathrm{d}b_2} = \dfrac{\mathrm{d}(w_2x_1 + w_4x_2 + b_2)}{\mathrm{d}b_2} = 1 $$

For b<sub>1</sub>:

$$
\begin{aligned}
\dfrac{\mathrm{d}L}{\mathrm{d}b_1} &= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}b_1} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}a_{h1}}.\dfrac{\mathrm{d}a_{h1}}{\mathrm{d}b_1} \\ \\
&= \dfrac{\mathrm{d}L}{\mathrm{d}a_{o1}}.\dfrac{\mathrm{d}a_{o1}}{\mathrm{d}z_{o1}}.\dfrac{\mathrm{d}z_{o1}}{\mathrm{d}a_{h1}}.\dfrac{\mathrm{d}a_{h1}}{\mathrm{d}z_{h1}}.\dfrac{\mathrm{d}z_{h1}}{\mathrm{d}b_1} \\ \\
&= (a_{o1} - y).\sigma(z_{o1})(1-\sigma(z_{o1})).w_5.\sigma(z_{h1})(1-\sigma(z_{h1}))
\end{aligned}
$$

which also takes a shorter form because:

$$ \dfrac{\mathrm{d}z_{h1}}{\mathrm{d}b_1} = \dfrac{\mathrm{d}(w_1x_1 + w_3x_2 + b_1)}{\mathrm{d}b_1} = 1 $$

# Training

With expressions for the derivative of the loss L with respect to each weight and bias, we can now train our neural network using the gradient descent equations that we saw earlier:

$$
\begin{aligned}
w_s &= w_s - \alpha\dfrac{\mathrm{d}L}{\mathrm{d}w_s} \text{  where s = 1, 2, ..., 6} \\ \\
b_t &= b_t - \alpha\dfrac{\mathrm{d}L}{\mathrm{d}b_t} \text{  where t = 1, 2, 3}
\end{aligned}
$$

Here's the step-by-step process for training:

<ul>
    <li>Step 1. Take the first training example</li>
    <ul>
        <li>Calculate the current outputs from each neuron using the forward propagation equations</li>
        <li>Update the weights and biases using the gradient descent equations</li>
    </ul>
    <li>Step 2. Repeat step 1 with each of the remaining training examples</li>
    <li>Step 3. Repeat the entire process another 10,000 epochs</li>
</ul>

After 10,000 epochs, the total loss J of our neural network should reduce, leading to an improvement in prediction accuracy.

In a perfect scenario, plotting the total loss J against the number of iterations performed will reveal something like this:

<img src="/assets/images/blog/nn/total-loss.png" alt="Total loss J vs Number of epochs">

With this sort of outcome, our trained neural network should do pretty well at predicting outputs that are very close to the expected outputs.  For example:

| x<sub>1</sub> | x<sub>2</sub> | y | a<sub>o1</sub> |
|----|----|---|---|
| 0  | 0  | 0 | 0.04 |
| 0  | 1  | 1 | 0.98 |
| 1  | 0  | 1 | 0.98 |
| 1  | 1  | 1 | 1.00 |

If we find the total loss J has not reduced at all or sufficiently after training, further optimisation may be achieved by experimenting with [different learning rates](https://towardsdatascience.com/understanding-learning-rates-and-how-it-improves-performance-in-deep-learning-d0d4059c1c10) $$ \alpha $$ (e.g. 0.1, 0.01, 0.001).  That's another topic in its own right...

# Summary

Congratulations if you made it to the end and thanks for reading!

Here's what we covered, some in more detail than others:

<ul>
    <li>Forward propagation</li>
    <li>Sigmoid activation function</li>
    <li>Concept of loss L and total loss J</li>
    <li>Backpropagation using partial derivatives and the chain rule</li>
    <li>Training to minimise the total loss J</li>
</ul>

Please comment below if you found this post useful or if you've spotted an error.