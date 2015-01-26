---
layout: post
title: "Neural Network to Predict DotA Game Results"
modified: 2015-01-25 20:34:36 -0800
comments: true
excerpt: 
tags: [Machine Learning]
image:
  feature: 
  credit: 
  creditlink: 
date: 2015-01-25 20:34:36 -0800
---
## Background
DotA (Defense of the Ancients) is a quite popular MOBA game, where 10 players fight 5 vs. 5 with 10 different heroes. The two sides are called the Radiant and the Dire.

Some heroes have abilities that work well together. And some heroes' abilities can be countered if the opposing side has certain heroes. Thus, result of a game is influenced by the lineups of each side to an extent.

## Goal
My goal was quite simple. I wanted to predict the outcome of a game given its lineups. 

But there is more to say about this. In DotA, a patch is released every few months, which changes the game a lot. Some heroes are buffed and regain popularity. Some heroes are nerfed and the win rate plummets. Moreover, a new play style for a certain hero might be discovered and the win rate rises like a rocket. So the relation between used heroes and result changes as the time goes, which limits our selection of data a lot.

## Data collection
Fortunately, DotA has an API for match results. However, it is a pain to use. So I used the database from a previous MIT licensed project by [Kevin](http://kevintechnology.com/post/71621133663/using-machine-learning-to-recommend-heroes-for), a collection of high-skill games in late 2013.

## Features
In DotA, there are 108 heroes, each of which may appear at either side. So for each match I use $$108 \times 2 = 216$$ bit features. The first 108 features describes which five heroes are on the Radiant. And the rest describes which five are on the Dire. In this way, whether a hero is strong or weak in the game can be easily represented in the trained network.

The predicted value is represented in a bit: 1 represents one side (the Radiant) wins; 0 represents the other side (the Dire) wins.

## Model
I chose a network with sigmoid activation functions and 6 layers. The numbers of neurons of the layers are 216, 316, 100, 20, 10, 1.

The idea behind this is that there are (in my opinion) roughly 100 pairs of heroes that work well together or counter each other. Classifiers such as logistic classifier cannot represent this non-linear relation. So I used a neural network with $$216 + 100$$ neurons at the second layer in hopes of capturing this fact.

## Data manipulation
There are 56635 valid matches. Because we are predicting results solely based on heroes, each matches can be used as two training examples, one being the Radiant wins, the other being the Dire wins after switching the heroes' sides. So we have $$56635 \times 2 = 113270$$ data.

Of all these data, 10% are used for validation, 10% are used for testing, and 80% are used for training.

## Training and validation
Due to limited computation power of my machine, the max epoch is set to be 100. In addition, after each epoch of training, the network is tested on the validation set. If the error rate improves, save the network. Otherwise, continue training for 20 more epochs to escape possible local optimum and stop afterwards.

## Implementation
All code is written in Python. 

Data are stored in a MongoDB. For neural nets, I use the PyBrain library. Python's `atexit` module is also used in case that training is interrupted.

Previously, I tried [Fast Artificial Neural Network Library (FANN)](http://leenissen.dk/fann/wp/) with C++. However, a bug at the library side forced my to leave FANN, which appeared to be a faster approach.

## Result
The training didn't take long on a Macbook Pro Retina 15-inch 2013 running OS X 10.10. After about 30~40 minutes, I obtained a network with 70.4% correct rate on test set, which I think is not impressive at all since previous attempt with logistic regression achieves 69.8%.

My speculation is that the interactions of heroes might not matter as much as I thought. The rest 29.6% goes to player skills, luck, strategies, communications, play styles, etc.

## Future work
Future works include 

1. update the database each day and only preserve the recent matches. In this way, the predictor can also be updated as the metagame changes;
2. force the network's weights to be symmetric (or shared) because in the end there are two features describing every hero;
3. try more complex network structures and activation functions.

I would love to include match making ratings (MMR, a measure of skill) of the players as part of the feature vector. However, Unfortunately, this information is not exposed in API. 

## GitHub repository
The code and saved network are [here](https://github.com/SsnL/dotaPredict). Feel free to comment with any question suggestion.

## A few unrelated words
It's been a while since my last post. It's quite difficult to code in China given the slow access to GitHub, Google, StackOverFlow, pretty much everything I needed. Nevertheless, it was really exciting to see my parents, my brother and my girlfriend again. 

Hope you had a great holiday as well.