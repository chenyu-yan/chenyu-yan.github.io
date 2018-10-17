---
layout: post
title:  "A Survey on DNN Compression Methods"
date:   2018-04-19 21:00:00 +0800
categories: Machine_Learning
tags: Deep_Learning
---

My final project in the course *Fundamentals of Machine Learning* relates to deep neural network (DNN) compression, so I read some research papers in this area and sum them up as a brief survey here.

## **Why Compress DNN**

* **Energy**: RAM access dominates the power consumption in DNN inference
* **Performance**: Large models might fail to run on entry-level hardwares
* **Mobile Computing**: Mobile devices have limited computing resources

## **Paper Summaries**

### *[Y Gong, et al. Compressing Deep Convolutional Networks Using Vector Quantization, arXiv 2014][gong2014compressing]*

There are mainly two classes of methods for compressing dense layers in DNN, **matrix factorization** and **vector quantization**. This paper discusses the later one.

Four vector quantization methods are discussed in the paper.

***Binarization***: for all parameter matrices, apply sign function to them.

***Scalar Quantization Using K Means***: Use K means to find some cluster centers of parameter matrix values. Then for each matrix element, use its cluster identifier instead of its origin values.

***Product Quantization***: This method divide a parameter matrix into several sub-matrices, and apply K means method to them respectively.

***Residual Quantization***: This method tries to recursively quantize residual value between origin value and quantized value, thus approximate each vector by representing it by a series of quantized vectors.

### *[S Han, et al. Learning both Weights and Connections for Efficient Neural Networks, NIPS 2015][han2015learning]*

This paper compresses the model by pruning connections among DNN nodes, including synapses and neurons. The whole workflow is as the below:

1. Train Connectivity
2. Prune Connections
3. Train Weights, go to 2 if better performance needed

Actually this workflow (**iterative pruning**) is a common one in DNN compression methods.

The intuition is that connections with trivial weights contributes little when the DNN makes decisions. The paper sets a weight threshold to throw unimportant connections. After pruning, the network has to be re-trained, otherwise its performance would be poor.

There are some other implementation details including:

* Use L2 regularization
* Dropout ratio should be lower in re-training phase since pruning already reduces model capacity (image here)
* Use origin weights as initialization when re-training
* Fix other layers when re-training certain layer to avoid gradient vanishing & errors

### *[H Hu, et al. Network Trimming: A Data-Driven Neuron Pruning Approach towards Efficient Deep Architectures, arXiv 2016][hu2016network]*

Instead of iteratively pruning connections, this paper iteratively prunes nodes. The intuition is that some nodes in a DNN may output zero in most of the times, indicating that it's not important.

The paper finds **APoZ** (Average Percentage of Zeros) for each node in a DNN, then throws nodes with APoZ below a certain threshold. The APoZ is estimated on a large dataset.

The threshold is empirically set as **one standard derivation from the mean APoZ**
of the target trimming layer.

### *[H Li, et al. Pruning Filters for Efficient ConvNets, arXiv 2016][li2016pruning]*

In Deep CNN, there are usually many filters. Some of them may be redundant, for they might learn redundant features, or no feature at all. These filters can be pruned, for they don't count much in DNN computation.

The paper uses the L1 norm for each CNN filter as the estimation of their importance, thus, the larger L1 norm is, the more important the filter is. The paper prunes least important filters globally.

After pruning, the whole network is re-trained. **Prune once and retrain** strategy is proved to be best in practice.

*In residual layers, the identical feature map (short path) determines which filters are pruned actually.*

### *[S Han, et al. Deep Compression: Compressing Deep Neural Networks with Pruning, Trained Quantization and Huffman Coding, ICLR 2016][han2015deep]*

As the title is, this paper presents a 3-stage method to compress DNN models, including pruning, quantization and huffman coding.

***Pruning***: Much like the author's previous work in NIPS 2015, pruning stage iteratively throw connections with small weights. Pruned parameters are stored as CSR or CSC sparse matrices.

***Quantization***: K-Means based quantization is performed locally in layer matrices. Centroids are initialized linearly in order to keep important weights (with large abs value) as many as possible. BP algorithm is performed on centroids as following: for each centroid, gradients of origin weights in its cluster are summed up and subtracted from centroid value.

***Huffman Coding***: A common method for compression.

Note that training is implemented on Caffe and predicting is implemented on Intel MLK and NVidia CUDA.

### *[Y He, et al. Channel Pruning for Accelerating Very Deep Neural Networks, ICCV 2017][he2017channel]*

This paper aims to prune CNN by removing redundant channels.

Channel pruning can be formalized as an optimization problem:

* Given channel weights of a convolution layer W
* Given dataset X
* Given origin layer output Y
* Given maximum channel number c'

Solve

* A 0-1 backpack problem solution among channels, i.e. choose a subset of channels
* Weights of remaining channels

such that minimize the difference between Y and output produced by pruned layer.

During the optimization procedure, values of W and channel subset will both change. The paper proposes to alternatively update them. Channel subset optimization reduces to **(i) LASSO regression** (with regularization terms, for the origin problem is NP-hard), and weight optimization reduces to **(ii) least squares**. In practice, the paper applies (i) multiple times and then (ii) just once for efficiency.

For residual layers, the paper

* Add a filter before the whole residual layer for channel pruning for the first layer
* Shortcut itself cannot be pruned, passing difference caused by first-layer pruning. The optimization target of the last layer is fixed by subtracting the difference.

### *[J Luo, et al. ThiNet: A Filter Level Pruning Method for Deep Neural Network Compression, ICCV 2017][luo2017thinet]*

This paper prunes CNN channels with the insight that, *if a subset of input of layer `i+1` is enough to recover the layer's output, then this subset is safe to be pruned as well as filters generating it*.

Pruning is data-driven, i.e. the subset of a layer to prune is selected by applying data to the layer and see which filters are less important.

For a single layer, the problem is formalized as a optimization problem minimizing the subset's contribution to layer output, which is NP-hard. The paper used greedy policy followed by fine-tuning to solve the problem. The whole network is pruned layer by layer.

[han2015learning]: http://papers.nips.cc/paper/5784-learning-both-weights-and-connections-for-efficient-neural-network.pdf
[hu2016network]: http://arxiv.org/pdf/1607.03250.pdf
[li2016pruning]: https://arxiv.org/pdf/1608.08710.pdf
[gong2014compressing]:https://arxiv.org/pdf/1412.6115
[han2015deep]:https://arxiv.org/pdf/1510.00149
[he2017channel]:http://openaccess.thecvf.com/content_ICCV_2017/papers/He_Channel_Pruning_for_ICCV_2017_paper.pdf
[luo2017thinet]:http://openaccess.thecvf.com/content_ICCV_2017/papers/Luo_ThiNet_A_Filter_ICCV_2017_paper.pdf
