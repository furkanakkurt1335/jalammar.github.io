---
layout: prediction_post
published: True
title: Visualizing A Neural Machine Translation Model (Mechanics of Seq2seq Models With Attention)
---

Sequence-to-sequence models are deep learning models that have achieved a lot of success in tasks like machine translation, text summarization, and image captioning. Google Translate started [using](https://blog.google/products/translate/found-translation-more-accurate-fluent-sentences-google-translate/) such a model in production in late 2016. These models are explained in the two pioneering papers ([Sutskever et al., 2014](https://papers.nips.cc/paper/5346-sequence-to-sequence-learning-with-neural-networks.pdf), [Cho et al., 2014](http://emnlp2014.org/papers/pdf/EMNLP2014179.pdf)).

I found, however, that understanding the model well enough to implement it requires unraveling a series of concepts that build on top of each other. I thought that a bunch of these ideas would be more accessible if expressed visually. That's what I aim to do in this post. You'll need some previous understanding of deep learning to get through this post. I hope it can be a useful companion to reading the papers mentioned above (and the attention papers linked later in the post).


A sequence-to-sequence model is a model that takes a sequence of items (words, letters, features of an images...etc) and outputs another sequence of items. A trained model would work like this:
<video width="100%" height="auto" loop autoplay controls>
  <source src="/images/seq2seq_1.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>



<!--more-->

<br />

In neural machine translation, a sequence is a series of words, processed one after another. The output is, likewise, a series of words:

<video width="100%" height="auto" loop autoplay controls>
  <source src="/images/seq2seq_2.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>


## Looking under the hood

Under the hood, the model is composed of an encoder and a decoder.

The encoder processes each item in the input sequence, it compiles the information it captures into a vector (called the context). After processing the entire input sequence, the encoder send the context over to the decoder, which begins producing the output sequence item by item.

<video width="100%" height="auto" loop autoplay  controls>
  <source src="/images/seq2seq_3.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>



<br />

The same applies in the case of machine translation.

<video width="100%" height="auto" loop autoplay controls>
  <source src="/images/seq2seq_4.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>


The context is a vector (an array of numbers, basically) in the case of machine translation. The encoder and decoder tend to both be [recurrent neural networks](https://www.youtube.com/watch?v=UNmqTiOnRfg).

<div class="img-div" markdown="0">
    <img src="/images/context.png" />
    The context is a vector of floats. Later in this post we will visualize vectors in color by assigning brighter colors to the cells with higher values.
</div>

You can set the size of the context vector when you set up your model. It is basically the number of hidden units in the encoder RNN. These visualizations show a vector of size 4, but in real world applications the context vector would be of a size like 256, 512, or 1024.

<br />

Each pulse for the encoder or decoder is that RNN processing its inputs and generating an output for that time step. Since the encoder and decoder are both RNNs, each time step one of the RNNs does some processing, it updates its hidden state based on its inputs and previous inputs it has seen.

Let's look at the hidden states for the encoder. Notice how the last hidden state is actually the context we pass along to the decoder.


<video width="100%" height="auto" loop autoplay controls>
  <source src="/images/seq2seq_5.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>


<br />

The decoder also maintains a hidden state that it passes from one time step to the next. We just didn't visualize it in this graphic because we're concerned with the major parts of the model for now.


Let's now look at another way to visualze a sequence-to-sequence model. This animation will make it easier to understand the static graphics that describe these models. This is called an "unrolled" view where instead of showing the one decoder, we show a copy of it for each time step. This way we can look at the inputs and outputs of each time step.

<video width="100%" height="auto" loop autoplay controls>
  <source src="/images/seq2seq_6.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

<br />

# Let's Pay Attention Now
The context vector turned out to be a bottleneck for these types of models. It made it challenging for the models to deal with long sentences. A solution was proposed in [Bahdanau et al., 2014](https://arxiv.org/abs/1409.0473) and [Luong et al., 2015](https://arxiv.org/abs/1508.04025). These papers introduced and refined a technique called "Attention", which highly improved the quality of machine translation systems. Attention allows the model to focus on the relevant parts of the input sequence as needed.


<img src="/images/attention.png" />

<div class="img-div" markdown="0">
    At time step 7, the attention mechanism enables the decoder to focus on the word "étudiant" ("student" in french) before it generates the English translation. This ability to amplify the signal from the relevant part of the input sequence makes attention models produce better results than models without attention.
</div>

<br />

Let's continue looking at attention models at this high level of abstraction. An attention model differs from a classic sequence-to-sequence model in two main ways:

First, the encoder passes a lot more data to the decoder. Instead of passing the last hidden state of the encoding stage, the encoder passes _all_ the hidden states to the decoder:

<video width="100%" height="auto" loop autoplay controls>
  <source src="/images/seq2seq_7.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>


<br />

Second, an attention decoder does an extra step before processing its inputs. In order to focus on the parts of the input that are relevant to this decoding time step, the decoder does the following:

 1. look at the set of encoder hidden states it received -- each encoder hidden state is most associated with a certain word in the input sentence
 1. give each hidden state a score (let's ignore how the scoring is done for now)
 1. multiply each hidden state by its softmaxed score, thus amplifying hidden states with high scores, and drowning out hidden states with low scores


<video width="100%" height="auto" loop autoplay controls>
   <source src="/images/attention_process.mp4" type="video/mp4">
   Your browser does not support the video tag.
</video>

<br />
<br />

This scoring exercise is done at each time step on the decoder side:

<video width="100%" height="auto" loop autoplay controls>
   <source src="/images/attention_tensor_dance.mp4" type="video/mp4">
   Your browser does not support the video tag.
</video>

<br />
<br />

This is another way to look at which part of the input sentence we're paying attention to at each decoding step:


<video width="100%" height="auto" loop autoplay controls>
  <source src="/images/seq2seq_9.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

Note that the model isn't just mindless aligning the first word at the output with the first word from the input. It actually learned from the training phase how to align words in that language pair (French and English in our example). An example for how precise this mechanism can be comes from the attention papers listed above:


<div class="img-div" markdown="0">
<img src="/images/attention_sentence.png" />
    You can see how the model paid attention correctly when outputing "European Economic Area". In French, the order of these words is reversed ("européenne économique zone") as compared to English. Every other word in the sentence is in similar order.
</div>


<br />

---

<br />

I hope you've found this useful. These visuals are early iterations of a lesson on attention that is part of the Udacity [Natural Language Processing Nanodegree Program](https://www.udacity.com/course/natural-language-processing-nanodegree--nd892). We go into more details in the lesson, including discussing applications and touching on more recent attention methods like the Transformer model from  [Attention Is All You Need](https://arxiv.org/abs/1706.03762).

Check out the trailer of the [NLP Nanodegree Program](https://www.udacity.com/course/natural-language-processing-nanodegree--nd892):

<iframe width="560" height="315" src="https://www.youtube.com/embed/FggN0VtWYTk" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

I've also created a few lessons as a part of Udacity's [Machine Learning Nanodegree Program](https://www.udacity.com/course/machine-learning-engineer-nanodegree--nd009t). The lessons I've created cover Unsupervised Learning, as well as a jupyter notebook on movie recommendations using collaborative filtering.

I'd love any feedback you may have. Please reach me at [@jalammmar](https://twitter.com/jalammar).