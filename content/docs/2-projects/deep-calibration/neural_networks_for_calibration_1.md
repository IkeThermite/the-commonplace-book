---
title: "Neural Networks for Calibration: Part 1"
date: 2020-01-03
bookToC: 3
---

# Neural Networks for Calibration: Part 1

## Introduction
The purpose of this page is track my experiments with using neural networks to move the hard work of model calibration "offline".

My primary resource is the 2016 paper by Hernandez, ["Model Calibration with Neural Networks"]().
This served as the inspiration for a ["2017 Financial Mathematics Team Challenge Project"](), which was supervised by Josef Teichman. 
Finally, in 2018 Nicolau Haussamer wrote ["Model Calibration with Machine Learning"]() as his short dissertation, in partial fulfillment of his MPhil in Mathematical Finance at the University of Cape Town.

## A First Network
In statistics, you begin with the normal distribution.
In quantitative finance, you begin with the Black-Scholes model.

My first attempt is to learn the inverse of the Black-Scholes put option price as a function of a single parameter; the implied volatility $\sigma$.

I construct a simple neural network with two hidden layers of 16 nodes each, both with the SoftMax activation function. In the middle, I perform a [batch normalization](https://arxiv.org/abs/1502.03167), but I do not attempt to train the weights of the normalization layer.

In [PyTorch](https://pytorch.org/), the network is defined as:
```python
class Net(nn.Module):
    def __init__(self, num_neurons):
        super(Net, self).__init__()
        self.lin1 = nn.Linear(1, num_neurons)
        self.softplus1 = nn.Softplus()
        self.bn2 = nn.BatchNorm1d(num_neurons, affine=False)
        self.lin2 = nn.Linear(num_neurons, 1)
        self.softplus2 = nn.Softplus()
        
    def forward(self, out):
        out = self.softplus1(self.lin1(out))
        out = self.bn2(out)
        out = self.softplus2(self.lin2(out))
        return out
```
It can be easily visualized using [Netron]():
![The network for calibration the Black-Scholes model.](/img/bs_calibration_netron.png#center)

## Learning the Inverse of the Black-Scholes Pricing Function
For my training data, I use simulated at-the-money put options with 5 years to expiry. I generate 100 000 pairs of ``(price, implied_volatility)`` for $\sigma\in[0.05, 1.5]$.
Using standard stochastic gradient descent, this is what you get:

![Animation of learning the inverse Black-Scholes put option pricing function.](/img/bs_calibration.gif#center)

Not bad, considering that I haven't done any hyperparameter optimization, but this is the simplest example. 
Let's take one step up.

## Calibrating a Two-parameter Model
Consider the CEV model, a generalization of Black-Scholes:
$$ dS_t = rS_t dt + \sigma S^\alpha_t dW_t $$
Now, given an option price, I want my network to be able to determine the parameter set, $\Theta=(\sigma, \alpha)$ that produced it.

