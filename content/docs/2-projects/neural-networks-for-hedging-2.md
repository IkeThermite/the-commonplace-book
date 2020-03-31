---
title: "Neural Networks for Hedging II"
date: 2019-11-27
bookToC: 3
---

# Neural Networks for Hedging II

## Introduction 
This is Part 2, Part 1 can be found [here](https://www.ralphrudd.com/docs/2-projects/neural-networks-for-hedging-1/). All the code for both parts is available on [Github](https://github.com/IkeThermite/Buehler-et-al-Deep-Hedging).
Our goal is to implement the 2018 (published in 2019) paper by Beuhler et al., ["Deep Hedging"](https://www.tandfonline.com/doi/full/10.1080/14697688.2019.1571683), using [PyTorch](https://pytorch.org).

In [Part 1](http://www.ralphrudd.com/blog/2019/10/22/neural-networks-for-hedging-1), we built a very simple neural net with two hidden layers. As a first test-case, we used it to approximate the Black-Scholes delta function, when the true analytical delta is used as the loss function.
Then, we attempted to learn the appropriate delta when the loss function was the profit and loss that arises from selling a vanilla call option and hedging with the underlying asset. Both of these experiments were done in a _single timestep model_, such that we sell the option today, implement our hedge, and the option expires one timestep from now, where we realize the profit or loss on our implemented hedge.

Here, we attempt to learn the two timestep hedge, but without making the network any deeper. Instead, we build a _recurrent neural net_.

## A Recurrent Neural Network
We will use the same neural network structure as in [Part 1](http://www.ralphrudd.com/blog/2019/10/22/neural-networks-for-hedging-1), but with the addition of a _hidden state_. 

This neural network, with two hidden layers as before, is traversed at each timestep and receives as input the price of the underlying asset for that time, as well as the delta from the previous timestep, representing the hidden state. At each timestep, the network outputs the delta for that time.

![A simple neural network.](/img/simple_recursive_network_nn_svg.png#center)

The network is _recurrent_, as the output at the current time (the hidden state), feeds into the network as an input at the next timestep. An alternative in our case would be to have a separate neural network for each timestep, with two hidden layers each. This would work, but would greatly increase the number of parameters that need to be trained.

In PyTorch, the network is defined as follows:

```python
class Net(nn.Module):
    def __init__(self, num_neurons):
        super(Net, self).__init__()
        self.lin1 = nn.Linear(2, num_neurons)
        self.sig1 = nn.Sigmoid()
        self.bn1 = nn.BatchNorm1d(num_neurons)
        self.lin2 = nn.Linear(num_neurons, 1)
        self.sig2 = nn.Sigmoid()
        

    def forward(self, X):
        batch_size, steps, _ = X.shape
        output = []
        self.y = torch.rand(batch_size, 1)
        
        for i in range(steps):
            self.y = torch.cat((X[:,i,:], self.y), dim=1)
            self.y = self.bn1(self.sig1(self.lin1(self.y)))
            self.y = self.sig2(self.lin2(self.y));
            output.append(self.y)
        
        return output, self.y       
```

The recurrent nature of the net comes from the ``for`` loop inside the ``forward`` method. There are two further modifications to the structure from Part 1: an additional sigmoid function on the first linear layer and a batch normalization procedure applied before the second layer. Batch normalization was recommended in the original paper and greatly increased the learning rate of the network.

## Learning The Two-step Hedge
We attempt to learn the best hedge position at each timestep by minimizing the terminal profit and loss.

At $t_0$ we sell a call option for $C_0$ and buy $\delta_0$ units of the underlying stock, $S_0$.
At $t_1$ we rebalance our hedge by adjusting our holding in the stock (our final position must be $\delta_1$). Finally, at $T = t_2$ we have to pay out the payoff of the option (if positive) and close out our position. Thus, the function we want to minimize looks like

$$
\delta_1(S_2 - S_1) + \delta_0(S_1 - S_0) + C_0 - (S_2 - K)^+
$$

and our network must provide both $\delta_0$ and $\delta_1$. Again these should be close to the Black-Scholes deltas, with the difference accounting for the discrete-time nature of the hedge.

To create the training set, we need to price the call options at $t_0$ as well as simulate realizations for the underlying asset at $t_1$ and $t_2$. 

The training of the RNN is illustrated below and contrasted to the analytical Black-Scholes delta at each timestep.

![A simple neural network.](/img/rnn_bs_hedge.gif#center)

## Next Steps
Roughly:
1. What is happening in the tails at $t_1$? Are there simply too few samples there to learn the appropriate shape?
2. Expand the two-period hedge to a $n$-period hedge. I am dissatisfied that the PyTorch ``RNNCell`` couldn't match my requirements here and that's worth investigating. I'll also need to get significantly more skilled at manipulating arrays.
2. As before, I still intend to change the underlying model to Heston. For this, we'll need two-dimensional inputs, as we'll require a derivative trading instrument to hedge the volatility.
