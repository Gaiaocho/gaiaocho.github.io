# Implementing Perceptrons and Sigmoid Neurons. 

In normal programing we  break down real world problems into discrete steps that
we later instruct the computer on how to perform. In contrast with Neural Networks,
we let the computer learn from observational data and let it figure out it's own
solution. 

This has been known for long since before the 1950s however a break through occured in 
the early 2000s where a technique for learning in deep neural networks. Today almost 20 year
later Neural networks  excel and an astounding amount of tasks.

Let's build one to understand them. 


## Neurons
Let's say we wanted to tackle Audio recognition, that is to say we wanted to create a program 
that given an audio signal could classify it as one thing or another, what exactly, isn't relevant 
for now but will become so later. In traditional programming such a task would become quickly 
riddled with branches and exceptions and with more recognitions becomes even more unwieldly. 

A task that is instinctual to us, becomes really tough when we try to instruct a computer to do it. 
It's not that such a task is easy rather we are amazingly good at such tasks in this particular aspect
it is because of our [Auditory Cortex](https://en.wikipedia.org/wiki/Auditory_cortex) which is comprised of a bunch of Neural Networks, which are in turn composed of [Neurons](https://en.wikipedia.org/wiki/Neuron). 

In biology a Neuron is a cell that fires electrical signal across a neural network. It fires these 
signals based on signals that it itself may receive from other Neurons and usually it fires these 
signals as inputs to other  neurons.

We will go through two types of artificial Neurons. 

### Perceptrons
Developed [since the 1940s](https://scholar.google.ca/scholar?hl=en&as_sdt=0,5&cluster=4035975255085082870), there is more modern ones but this will be the base upon which we shall build. 

Consider a perceptron to be a function that takes several binary inputs and and produces a single binary output. To compute the output each binary input is attached to a `weight` this weight might be 
known or provided, we will assume provided. The neurons output value is based on weither the  weightedsum  is less than or greater  than some value known as a `threshold`.

```c
int fire_perceptron(struct neuron_input *inputs, int threshold, size_t  len) {
    int weighted_sum = sum_of_weights(inputs, len); 

    if(weighted_sum > threshold) {
        return 1; 
    } else {
        return 0; 
    }
}
```

Consider the firing function above, we have a function  that computes fireing for a Neuron, it takes 
some inputs and  computes whether to fire (1) or not (0)  based on the sum of weights

### Sigmoid Neurons 

Now as we explore Neurons further we will find that due to the their  nature once we are layering
them into networks, Small changes in input will cause large change in outputs and this is particularly
obvios when layered.  
To tackle this we introduce a sigmoid function as an alternative to a step function which effectively is what our perceptron is.

```c 
int fire_sigmoid(struct neuron_input *inputs, int threshold, size_t  len) {
    int weighted_sum = sum_of_weights(inputs, len); 

    return 1 / (1 + exp(-(weighted_sum - bias))); 
}
```

## In the Real World
The Sigmoid Neuron is more suited to deep learning due to its nature of basically being able to take 
a large number, and express it as a value between 0 and 1. And this gives us the ability to have a small change i.e in the weight of one of our inputs and  have that small change cause a small change in the output. 

Now we can learn by making small changes to inputs and observing their results! 




## References 
[Dot Product](https://en.wikipedia.org/wiki/Dot_product)
[Auditory Cortex](https://en.wikipedia.org/wiki/Auditory_cortex)
[NeoCortex](https://en.wikipedia.org/wiki/Neocortex)
[Neuron](https://en.wikipedia.org/wiki/Neuron)
[Neural Networks and Deep Learning](http://neuralnetworksanddeeplearning.com/about.html)
