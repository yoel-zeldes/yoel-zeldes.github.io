Title: Preparing for the Unexpected
Slug: preparing-for-the-unexpected
Date: 2019-01-27 23:00
Tags: deep learning
Summary: How to apply your model to input it has never seen before.
header_cover: images/oov/cover.jpg

Some of the problems we tackle using machine learning involve categorical
features that represent real world objects, such as words, items and categories.
So what happens when at inference time we get new object values that have never
been seen before? How can we prepare ourselves in advance so we can still make
sense out of the input?

Unseen values, also called OOV (Out of Vocabulary) values, must be handled
properly. Different algorithms have different methods to deal with OOV values.
Different assumptions on the categorical features should be treated differently
as well.

In this post, I’ll focus on the case of deep learning applied to dynamic data,
where new values appear all the time. I’ll use Taboola’s recommender system as
an example. Some of the inputs the model gets at inference time contain unseen
values — this is common in recommender systems. Examples include:

* Item id: each recommendable item gets a unique identifier. Every day thousands
of new items get into the system.
* Advertiser id: sponsored content is created by advertisers. The number of new
daily advertisers is much smaller compared to the number of new items.
Nonetheless, it’s important to handle them correctly, especially since we want
to support new advertisers.

So what’s the challenge with OOV values?

# Learning to handle OOV values
An OOV value is associated with values not seen by the model at training time.
Hence, if we get an OOV value at inference time, the model won’t know what to do
with it.

One simple solution is to replace all the rare values with a special OOV token
before training. Since all OOV values are the same from the model’s point of
view, we’ll replace them with the OOV token at inference time. This solution has
two positive outcomes:

1.  The model will be exposed to the OOV token while training. In deep learning we
usually embed categorical features. After training, the model will learn a
meaningful embedding for all OOV values .
1.  The risk of overfitting to the rare values will decrease. These values appear in
a small number of examples. The model might learn to use their embeddings to
explain particularities or random noise found in these specific examples.<br> 
Another possible disaster is that these embeddings won’t get enough gradient
updates propagated to them. As a consequence, the random initialization will
dominate the result embeddings over the signal learned through training.

Problem solved... Or is it?

# Handling OOV values is hard!
The model uses the item id feature to memorize different information per item,
similarly to the pure [collaborative
filtering](https://en.wikipedia.org/wiki/Collaborative_filtering) approach. Rare
items that are injected with the OOV token can’t benefit from it, so the model
performs worse on them.

The interesting thing is that even if we don’t use the item id at all during
training, the model still performs worse on rare items! This is because they
come from a distribution different than that of the general population. They
have specific characteristics — maybe they performed poorly online, which caused
Taboola’s recommender system to recommend them less, and in turn — they became
rare in the dataset. So why does this distribution difference matter?

If we learn the OOV embedding using this special distribution, it won’t
generalize to the general population. Think about it this way — every item was a
new item at some point. At that point, it was injected with the OOV token. So
the OOV embedding should perform well for all possible items.

# Randomness is the data scientist’s best friend
In order to learn the OOV embedding using the general population, we can inject
the OOV token to a random set of examples from the dataset before we start the
training process. But how many examples will suffice?

The more we sample, the better the OOV embedding will be. But at the same time,
the model will be exposed to a fewer number of non-OOV values, so the
performance will degrade.

How can we use lots of examples to train the OOV embedding while at the same
time use the same examples to train the non-OOV embeddings? Instead of randomly
injecting the OOV token before starting to train, we chose the following
approach: in each epoch the model trains using all of the available values (the
OOV token isn’t injected). At the end of the epoch we sample a random set of
examples, inject the OOV token, and train the model once again. This way, we
enjoy both worlds!

To evaluate the new approach, we injected the OOV token to all of the examples
and evaluated our offline metric (MSE). It improved by 15% compared to randomly
injecting the OOV token before the model starts to train.

# Final thoughts
Our model had been used in production for a long time before we thought of the
new approach. It could have been easy to miss this potential performance gain,
since the model performed well overall. It just stresses the fact that you
always have to look for the unexpected!

---

*Originally published by me at
[engineering.taboola.com](https://engineering.taboola.com/preparing-for-the-unexpected).*
