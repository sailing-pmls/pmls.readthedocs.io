.. Petuum documentation master file, created by
   sphinx-quickstart on Mon Jun  6 16:01:54 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

.. image:: images/full_logo.png
    :align: center
|
Overview
========

Petuum is a distributed machine learning (ML) platform. It takes care of the difficult system "plumbing work", allowing users to focus on the ML. Petuum runs efficiently at scale in data-centers and cloud compute environments like Amazon EC2 and Google GCE. See our :doc:`quickstart guide <quickstart>` to get started running applications on Petuum.

Petuum provides the essential distributed programming tools to tackle the challenges of running ML at scale: big data (many data samples) and big models (large parameter and intermediate variable spaces). Unlike general-purpose data processing platforms like Hadoop and Spark, Petuum is designed specifically for ML algorithms, which means that it is able to take advantage of data correlation, error tolerance, and other statistical properties to maximize the performance of ML algorithms. All of this is realized through the following three core frameworks:

* Bosen [`github <https://github.com/petuum/bosen>`_] [`doc <http://docs.petuum.com/projects/petuum-bosen/>`_], for network-optimized iterative ML.
* Strads [`github <https://github.com/petuum/strads>`_] [`doc <http://docs.petuum.com/projects/petuum-strads>`_], for model-parallel scheduled ML.
* Poseidon [`github <https://github.com/petuum/poseidon>`_] [`doc <http://docs.petuum.com/projects/petuum-poseidon>`_], for distributed GPU deep learning.

Our name Petuum is inspired by "perpetuum mobile," a musical style characterized by a continuous steady stream of notes. Paganini's Moto Perpetuo is an excellent example. It is our goal to build a system that runs efficiently and reliably -- in perpetual motion.

Applications
------------

In addition to programming frameworks, Petuum includes many built-in distributed ML algorithms, each implemented for speed and scalability:

1. Topic Models
   
   * :doc:`Latent Dirichlet allocation <latent-dirichlet-allocation>`
   * :doc:`MedLDA <med-lda>`

2. Deep Learning
   
   * :doc:`General-purpose deep neural network <dnn-general>`
   * :doc:`DNN for speech recognition <dnn-speech>`

3. Matrix factorization and sparse coding

   * :doc:`Matrix factorization <matrix-fact>`
   * :doc:`Non-negative matrix factorization <nonneg-matrix-fact>`
   * :doc:`Sparse coding <sparse-coding>`

4. Regression

   * :doc:`Lasso regression <lasso-and-lr>`

5. Metric learning
 
   * :doc:`Distance metric learning <distance-metric-learning>`

6. Clustering
 
   * :doc:`K-means clustering <k-means>`

7. Classification
 
   * :doc:`Random forest <random-forest>`
   * :doc:`Logistic regression <lasso-and-lr>`
   * :doc:`Support vector machine <support-vector-machine>`
   * :doc:`Multi-class logistic regression <multiclass-logistic-regression>`

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
