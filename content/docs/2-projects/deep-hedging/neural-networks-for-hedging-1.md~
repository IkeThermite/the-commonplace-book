---
title: "Neural Networks for Hedging: Part 1"
date: 2019-10-22
---

# Table of Contents
1. [Introduction](#introduction)
2. [A First Network](#afirstnetwork) 
3. [Learning The Black-Scholes Delta](#blackscholesdelta)
4. [Learning A One-step Hedge](#hedge)
5. [Next Steps](#nextsteps)
6. [Resources](#resources)


# Introduction <a name="introduction"></a>
The purpose of this page is track my progress implementing the 2018 (published in 2019) paper by Beuhler et al., ["Deep Hedging"](https://www.tandfonline.com/doi/full/10.1080/14697688.2019.1571683).
In that paper they use a semi-recurrent deep neural network to calculate the appropriate hedging positions when hedging vanilla options.

They begin in a simulation setting using the Heston model and their network is implemented using [TensorFlow](http://tensorflow.org). 
Owing to the current [state of machine learning frameworks](https://thegradient.pub/state-of-ml-frameworks-2019-pytorch-dominates-research-tensorflow-dominates-industry/), as well as to the fact that one of my colleagues already has TensorFlow experience, I have decided to use [PyTorch](https://pytorch.org).

# A First Network <a name="afirstnetwork"></a>
Beuhler et al. use a neural network with two hidden layers to compute the necessary position (delta) in the underlying for that trading day.
The network is semi-recurrent because the delta for day $i$ is used as an input for the new network at day $i+1$.
It is also *very deep* as you essentially have two hidden layers per trading day.

To start, I construct the simplest neural network based on their hyperparameters. 
They use $d+15$ nodes per hidden layer, where $d$ is the number of inputs (the underlying assets to be traded).
Using PyTorch, the network is defined as follows:

```python
# Imports and Seeds
import numpy as np
import scipy.stats as stats
import matplotlib.pyplot as plt

import torch
import torch.nn as nn
import torch.optim as optim

np.random.seed(0)
torch.manual_seed(0)

# Construct Neural Net
class Net(nn.Module):
    
    def __init__(self):
        super(Net, self).__init__()
        self.lin1 = nn.Linear(1, 16)
        self.lin2 = nn.Linear(16, 1)
        self.sigmoid1 = nn.Sigmoid();
        
    def forward(self, S0):
        out = self.lin1(S0)
        out = self.lin2(out)
        out = self.sigmoid1(out)
        return out    

net = Net()         
```

Note that I'm using a sigmoid activation function on the second hidden layer, whereas Beuler et al. use ReLU. 
I'll get back to this.
This simple network can be represented graphically in its entirety:
![A simple neural network.](/img/simple_network_nn_svg.png#center)

The input (single node on the left) will be the normalized price of the underlying asset, whereas the output (single node on the right) would be optimal trading position in that asset.

The mathematical operations being performed by each layer can be easily 
visualized by saving the Pytorch model and then importing it into Netron.
(Note: In theory, TensorBoard is really the way to do this, but I was having compatibility issues.)

![A simple network represented as an execution path.](/img/simple_network_netron.png#center)

# Learning The Black-Scholes Delta <a name="blackscholesdelta"></a>
To check that the network is implemented correctly, I test whether it can approximate a *known* non-linear function. 
For this, I generate a random sample of initial stock price values and use as a target the corresponding Black-Scholes deltas for a call option struck at $100$, one week from maturity.

```python
# Functions
def d1(S0, K, T, r, sigma):
    return (np.log(S0 / K) + (r + 0.5 * sigma ** 2) * T) / (sigma * np.sqrt(T))

def d2(S0, K, T, r, sigma):
    return d1(S0, K, T, r, sigma) - sigma * np.sqrt(T)

def price_put_BS(S0, K, T, r, sigma):
    return (stats.norm.cdf(-d2(S0, K, T, r, sigma)) * K * np.exp(-r * T) - 
                           stats.norm.cdf(-d1(S0, K, T, r, sigma)) * S0)
def price_call_BS(S0, K, T, r, sigma):
    return (stats.norm.cdf(d1(S0, K, T, r, sigma)) * S0 - 
            stats.norm.cdf(d2(S0, K, T, r, sigma)) * K * np.exp(-r * T))

def delta_put_BS(S0, K, T, r, sigma):
    return -stats.norm.cdf(-d1(S0, K, T, r, sigma))

def delta_call_BS(S0, K, T, r, sigma):
    return stats.norm.cdf(d1(S0, K, T, r, sigma));
# Parameters
filename = 'bs_delta_1'
S0 = 100
K = 100
T = 1/50
r = 0.05
sigma = 0.2
num_samples = 1000;
num_epochs  = 15;
batch_size = 4;
S0_lower_bound = 90;
S0_upper_bound = 110;

uniform_samples = np.random.rand(num_samples, 1)
S0_values = (S0_upper_bound - S0_lower_bound) * uniform_samples + S0_lower_bound
delta_values = delta_call_BS(S0_values, K, T, r, sigma)            
```

I setup up easy-to-use iterables using ``torch.utils.data``.
As a loss function, I use the standard mean-squared error and for an optimizer I use stochastic gradient descent.

```python
# Create Data Loaders
training_set = torch.utils.data.TensorDataset(torch.Tensor(uniform_samples), 
        torch.Tensor(delta_values))
training_loader = torch.utils.data.DataLoader(training_set, batch_size=batch_size,
                                              shuffle=True)

# Define Loss Function and Optimizer
criterion = nn.MSELoss()
optimizer = optim.SGD(net.parameters(), lr=0.1)
```

With all of this in place, we can finally train the network to approximate the analytical delta.

```python
for epoch in range(num_epochs):
    running_loss = 0.0
    for i, data in enumerate(training_loader, 0):
        inputs, targets = data;
        # Zero the parameter gradients
        optimizer.zero_grad()
        
        # Forward + Backward + Optimize
        outputs = net(inputs)
        loss = criterion(outputs, targets)
        loss.backward()
        optimizer.step()
       
        # Print statistics
        running_loss += loss.item()
    
    print('[%d] loss: %.6f' % (epoch + 1, running_loss))
```

As can be seen below, our simple network manages to approximate the analytical Black-Scholes delta quite quickly.

![Learning the Black-Scholes Delta](/img/bs_delta_1.gif#center)

# Learning A One-step Hedge <a name="hedge"></a>
The next step is to attempt to learn the best hedge position without any knowledge of the analytical delta, but rather by trying to minimize the profit and loss.
The simplest case is a one-period model. 

At $t_0$ we sell a call option for $C_0$ and buy $\delta_0$ units of the underlying stock, $S_0$. 
Then at $T_1$ we have to pay out the payoff of the option (if positive) and close out our position. Thus, the function we want to minimize looks like

$$
\delta_0(S_1 - S_0) + C_0 - (S_1 - K)^+
$$

This will be close to the Black-Scholes delta, with a difference accounting for the discrete-time nature of the hedge.
We need to price the call options at $t_0$ as well as simulate realizations for the underlying asset at $t_1$. 
I also construct new data loaders.

```python
uniform_samples = np.random.rand(num_samples, 1)
normal_samples = np.random.randn(num_samples, 1)
S0_values = (S0_upper_bound - S0_lower_bound) * uniform_samples + S0_lower_bound
S1_values = S0_values * np.exp((r - 0.5 * sigma **2) * T 
                               + sigma * np.sqrt(T) * normal_samples)
call_values = price_call_BS(S0_values, K, T, r, sigma)

training_set = torch.utils.data.TensorDataset(torch.Tensor(uniform_samples),
        torch.Tensor(S0_values),                                      
        torch.Tensor(S1_values),
        torch.Tensor(call_values))
training_loader = torch.utils.data.DataLoader(training_set, batch_size=batch_size,
                                              shuffle=True)
```

There's no need to define a custom loss function, as you can cast the problem in terms of MSE.
Then we can train the network.

```python
for epoch in range(num_epochs):
    running_loss = 0.0
    for i, data in enumerate(training_loader, 0):
        inputs, S0, S1, C0 = data;
        # Zero the parameter gradients
        optimizer.zero_grad()
        
        # Forward + Backward + Optimize
        outputs = net(inputs)
        loss = criterion(outputs * (S1 - S0) + C0, 
                         torch.max(S1 - K, torch.zeros(batch_size, 1)))
        loss.backward()
        optimizer.step()
       
        # Print statistics
        running_loss += loss.item()
    
    print('[%d] loss: %.6f' % (epoch + 1, running_loss))
```

The optimization here is not as smooth as for the previous case, which is to be expected considering
the additional randomness (the simulations of $S_1$) and non-linearity (we're further removed from the target function).
The convergence is illustrated below.

![Learning a One-step Hedge](/img/bs_hedge_1.gif#center)

# Next Steps <a name="nextsteps"></a>
Roughly:
1. Implement a two-period hedge. This requires constructing a semi-recurrent network, for which I'll need additional PyTorch API knowledge.
2. Change the underlying model to Heston. For this, we'll need two-dimensional inputs, as we'll require a derivative trading instrument to hedge the volatility.

# Resources <a name="resources"></a>
The simple neural network was visualized using
 - [Netron](https://lutzroeder.github.io/ai) and
 - [NN SVG](http://alexlenail.me/NN-SVG/index.html)
 
 Both have browser-based implementations available. 
 I edit the resulting SVGs using [Inkscape](https://inkspace.org).