# Implementation of A Simple Self-supervised Algorithm for Contrastive Learning in Visual Representations 

## A. Motivations

Getting AI to learn like a new-born baby is the goal of self-supervised learning. Scientists are working on creating better AI that learns through self-supervision, with the pinnacle being AI that could learn like a baby, based on observation of its environment and interaction with people. This would be an important advance because AI has limitations based on the volume of data required to train machine learning algorithms (**scalability**), and the brittleness of the algorithms when it comes to adjusting to changing circumstances (**Improved AI capabilities**). Some early success with self-supervised learning has been seen in the natural language processing used in mobile phones, smart speakers, and customer service bots. Training AI today is time-consuming and expensive. The promise of self-supervised learning is for AI to train itself without the need of external labels attached to the data in which human intervention and supervision are both involved. In other words, the self-supervised learning aims to further enable machines to come up with a solution automatically - completely relying on their own capabilities without any interference (**Understanding how the human mind works**). As we understand this self-learning process better, we will be able to get closer to create models that think more similar to humans. 

   Fig. 1                     |   Fig. 2
:----------------------------:|:------------------------------:
![Fig1](./imgs/demo_simclr_1.png) | ![Fig2](./imgs/demo_simclr_2.gif)

## B. Differences among Supervised, Unsupervised, Semi-supervised and Self-supervised Learnings

  Supervised | Unsupervised | Semi-supervised | Self-supervised 
:-----------:|:------------:|:---------------:|:---------------:
pure (type-1) | pure (type-2) | hybrid of type-1 and type-2 | subset of type-2
train labeled input | train unlabeled input | train labeled input | train unlabeled input
predict labeled input | no predictions | predict unlabeled input | predict unlabeled input
test labeled input | no testing | test labeled input | test unlabeled input
regression | clustering, market segmentation | look-alike segmentation | regression, image segmentation
binary or multiple classification | grouping, association, dim. reduction | binary classification | binary or multiple classification

## C. Comparison of Advantages and Disadvantages in Self-supervised Learning 

Merits           | Shortages
:---------------:|:----------------------:
scalability | intensity 
capability | inaccuracy
human-alike | irreproducity

## D. Applications in Self-supervised Learning

* Healthcare
* Autonomous driving
* Chatbots
* Injury and illness prevention
* Climate change
* Vaccine development
* Behavior science

![Fig10](./imgs/demo_simclr_10.png)

## E. What's the contrastive learning? 

Contrastive learning is a self-supervised, task-independent deep learning technique that allows a model to learn about data, even without labels. Contrastive learning attemps to teach machines how to differentiate similar objusts from dissimilar ones without the need of manual annotation (aka: human supervision as required in supervised learning). In brief, it's an approach toward learning data without labels. SimCLR is an example of a contrastive learning approach that learns how to represent images such that similar images have similar representations, thereby allowing the model to learn how to distinguish between images. The pre-trained model with a general understanding of the data can be fine-tuned for a specific task such as image classification when labels are scarce to significantly improve label efficiency, and potentially surpass supervised methods. 

  Fig. 3                     |   Fig. 4
:----------------------------:|:------------------------------:
![Fig3](./imgs/demo_simclr_4.png) | ![Fig4](./imgs/demo_simclr_3.png)


First thing, our machine turns each input into a representation. Afterward, the similarity between a pair of representations is computed. 

 Fig. 5                     |   Fig. 6
:----------------------------:|:------------------------------:
![Fig5](./imgs/demo_simclr_5.png) | ![Fig6](./imgs/demo_simclr_6.png)

## F. What's the SIMple framework for Contrastive Learning of visual Representation (SimCLR)? 

With the rules of contrast learning, SimCLR provides a model that learns representations by maximizing the agreement between randomly transformed views of the same data sample through minimizing the contrast loss in the latent space.

![Fig8](./imgs/demo_simclr_8.gif)

## G. Principles of SimCLR 
[paper (Chen et. al)](https://arxiv.org/abs/2002.05709)
[github](https://github.com/google-research/simclr)

![Fig7](./imgs/demo_simclr_7.png)

Suppose we have a training corpus of millions of unlabeled images.

### a. Data Augmentation

First, we generate batches of 8,192 raw images. We perform any combination of the following augmentations randomly: `crop (or resize), flip, color distortion (or jitter), grayscale`. We do this twice per image in our batch. Thus, for a batch size of 2, we get 2 * N (N=2) = 2 * 2 = 4 total images.

![Fig11](./imgs/demo_simclr_11.png)

### b. Base Encoder

We then use our DNN as baseline behaving as an encoder, h = f(x), to extract vector representations from augmented images (`x`). The encoder used is generic and replaceable with other architectures such as [ResNet-50](https://arxiv.org/abs/1512.03385), VGG-16, VGG-19 and so on. ResNet-50 architecture is selected as the ConvNet encoder. The output is a 2048-dimensional vector h, which is commonly shared by each positive pair of two augmented images. By compressing our images into a latent space representation, the model is able to learn the high-level features of the images.

![Fig12](./imgs/demo_simclr_12.png)

### c. Projection Head

The output of the CNN is then inputted to a set of Dense Layers called the projection head, z = g(h) to transform the data into another space. The representations h_{i} and h_{j} of the two augmented images are then passed through a series of non-linear `Dense -> Relu -> Dense` layers to apply non-linear transformation and project it into a representation z_{i} and z_{j}. This is denoted by g(.) in the paper and called projection head. This extra step is empirically shown to improve performance. This component maps representations of the space in which contrastive loss is applied.

![Fig13](./imgs/demo_simclr_13.png)

### d. Contrastive Loss Function

Thus, for each augmented image in the batch, we get embedding vectors z_{i}, z_{j}, z{k}, z{l}. From these embedding, we calculate the loss in following steps:

#### (1) Calculation of Cosine Similarity

Since we are comparing two vectors, a natural choice is cosine similarity, which is based on the angle between the two vectors in space. The similarity between two augmented versions of an image is calculated using cosine similarity. For two augmented images x_{i} and x_{j}, the cosine similarity is calculated on its projected representations z_{i} and z_{j}. It is logical that when vectors are closer (smaller angle between them) together in space, they are more similar. Thus, if we take the cosine(angle between the two vectors) as a metric, we will get a high similarity when the angle is close to 0, and a low similarity otherwise, which is exactly what we want.

![Fig14](./imgs/demo_simclr_14.png)

, where:

* τ is the adjustable temperature parameter. It can scale the inputs and widen the range [-1, 1] of cosine similarity
* ∥z_{i}∥ is the norm of the vector.

#### (2) Loss Calculation

We also need a loss function that we can minimize. SimCLR uses a contrastive loss called “NT-Xent loss” (Normalized Temperature-Scaled Cross-Entropy Loss). We first apply the softmax function to compute the probability that the two augmented images are similar. Notice that the denominator is the sum of e^similarity(all pairs, including negative pairs). Negative pairs are obtained by creating pairs between our augmented images, and all of the other images in our batch. 

![Fig9](./imgs/demo_simclr_9.png)

This softmax calculation is equivalent to getting the probability of the second augmented cat image being the most similar to the first cat image in the pair. Here, all remaining images in the batch are sampled as a dissimilar image (negative pair). 

![Fig16](./imgs/demo_simclr_16.png)

Lastly, we wrap this value around in a -log() so that minimizing this loss function corresponds to maximizing the probability that the two augmented images are similar.

![Fig15](./imgs/demo_simclr_15.png)

We calculate the loss for the same pair a second time as well where the positions of the images are interchanged.

![Fig17](./imgs/demo_simclr_17.png)

Finally, we compute loss over all the pairs in the batch of size N=2 and take an average.

![Fig18](./imgs/demo_simclr_18.png)

Based on the loss, the encoder and projection head representations improves over time and the representations obtained place similar images closer in the space.


### e. Improving performance


## H. Requirements:
  
  * jupyterlab==2.0.1
  * Keras==2.3.1
  * tensorflow==2.1
  * pandas==0.22.0
  * opencv-python==4.2.0.32
  * scikit-learn==0.23.1
  * scipy==1.4.1
  * numpy==1.18.1
  * DateTime==4.3
