---
layout: post
title: Running PredictionIO using its Docker Image
tags: [recommendation, prediction-io, python, machine-learning]
bigimg: /img/posts/2018-06-14-running-docker-image-of-predictionio/predictionio.png
---

### What is PredictionIO?

*Apache PredictionIO is an open source Machine Learning Server built on top of a state-of-the-art open source stack for developers and data scientists to create predictive engines for any machine learning task.*

PredictionIO machine learning templates can be used for creating and deploying machine learning models with ease. After Deploying it provides an API to interact with the deployed ML engine(which containing the trained model), which can be used to fetch the desired results.

There are different types of templates available which can be used as per requirement and it also provides an option to customize the templates as per needs.

PredictionIO is open source and is a project of Apache foundation. Currently it supports MLLib and OpenNLP libraries.

### Running PredictionIO 
Running PredictionIO through its docker image is one of the best options and all necessary dependencies are already installed.

We will be using a docker image for the latest version of PredictionIO created by [steveny2k](https://github.com/steveny2k). This image on [Dockerhub](https://hub.docker.com/r/steveny/predictionio/) 

Following are the steps to start and run a recommendation engine on the docker container -

<script src="https://gist.github.com/amandeep511997/87ff0a0d3800ce88f9b66e2487316035.js"></script>

We can install PredictionIO using other options also, see [here](http://predictionio.apache.org/install/).



