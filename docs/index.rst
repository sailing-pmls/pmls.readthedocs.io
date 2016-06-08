.. Petuum documentation master file, created by
   sphinx-quickstart on Mon Jun  6 16:01:54 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

.. image:: _images/full_logo.png
    :align: center
|
Overview
========

Petuum is a distributed machine learning (ML) platform. It takes care of the difficult system "plumbing work", allowing users to focus on the ML. Petuum runs efficiently at scale in data-centers and cloud compute environments like Amazon EC2 and Google GCE.

Petuum provides the essential distributed programming tools to tackle the challenges of running ML at scale: big data (many data samples) and big models (large parameter and intermediate variable spaces). Unlike general-purpose data processing platforms like Hadoop and Spark, Petuum is designed specifically for ML algorithms, which means that it is able to take advantage of data correlation, error tolerance, and other statistical properties to maximize the performance of ML algorithms. This is all realized through the following three core frameworks:

* Bosen [`doc <https://github.com/petuum/bosen>`_] [`github <https://github.com/petuum/bosen>`_], for network-optimized iterative ML.
* Strads [`doc <https://github.com/petuum/strads>`_] [`github <https://github.com/petuum/strads>`_], for model-parallel scheduled ML.
* Poseidon [`doc <https://github.com/petuum/poseidon>`_] [`github <https://github.com/petuum/poseidon>`_], for distributed GPU deep learning.

The name Petuum comes from "perpetuum mobile," which is a musical style characterized by a continuous steady stream of notes. Paganini's Moto Perpetuo is an excellent example. It is our goal to build a system that runs efficiently and reliably -- in perpetual motion.

Applications
------------

In addition to programming frameworks, Petuum includes many built-in distributed ML algorithms, each implemented for speed and scalability:

1. Topic Models
  a) Latent Dirichlet allocation (topic modeling)
  b) MedLDA (supervised topic modeling)
2. Deep Learning
  a) General-purpose deep neural network
  b) DNN for speech recognition
3. Matrix factorization and Sparse coding
  a) Matrix factorization
  b) Non-negative matrix factorization
  c) Sparse coding
4. Regression
  a) Lasso regression
5. Metric learning
  a) Distance metric learning
6. Clustering
  a) K-means clustering
7. Classification
  a) Random forest
  b) Logistic regression
  c) Support-vector machine
  d) Multi-class logistic regression

.. Welcome to Petuum's documentation!
.. ==================================

.. Contents:

.. toctree::
   :maxdepth: 2

.. Indices and tables
.. ==================

.. * :ref:`genindex`
.. * :ref:`modindex`
.. * :ref:`search`
