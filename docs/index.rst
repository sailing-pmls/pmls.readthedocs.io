.. Petuum documentation master file, created by
   sphinx-quickstart on Mon Jun  6 16:01:54 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

.. image:: _images/full_logo.png
    :align: center
|
Overview
========

Petuum is a distributed machine learning (ML) framework. It takes care of the difficult system "plumbing work", allowing users to focus on the ML. Petuum runs efficiently at scale in data-centers and cloud compute environments like Amazon EC2 and Google GCE.

Petuum provides the essential distributed programming tools to tackle the challenges of running ML at scale: big data (many data samples) and big models (large parameter and intermediate variable spaces). Unlike general-purpose data processing platforms like Hadoop and Spark, Petuum is designed specifically for ML algorithms, which means that it is able to take advantage of data correlation, error tolerance, and other statistical properties to maximize the performance of ML algorithms. This is all realized through core features such as Bosen (a bounded-asynchronous key-value store) and Strads (a scheduler for model-parallel ML computations).

In addition to distributed ML programming tools, Petuum comes with many distributed ML algorithms, each implemented for speed and scalability. Please refer to the Petuum wiki for a full listing: `<https://github.com/petuum/bosen/wiki>`_.

The Petuum project is organized into three open-source components:

* `Bosen (Bounded-async key-value store) <https://github.com/petuum/bosen>`_
* `Strads (Model-parallel scheduler) <https://github.com/petuum/strads>`_
* `Poseidon (Deep Learning framework) <https://github.com/petuum/poseidon>`_

The name Petuum comes from "perpetuum mobile," which is a musical style characterized by a continuous steady stream of notes. Paganini's Moto Perpetuo is an excellent example. It is our goal to build a system that runs efficiently and reliably -- in perpetual motion.

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
