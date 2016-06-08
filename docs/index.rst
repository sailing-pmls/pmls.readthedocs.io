.. Petuum documentation master file, created by
   sphinx-quickstart on Mon Jun  6 16:01:54 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Petuum Overview
===============

Petuum is a distributed machine learning framework. It takes care of the difficult system "plumbing work", allowing you to focus on the ML. Petuum runs efficiently at scale on research clusters and cloud compute like Amazon EC2 and Google GCE.

Petuum provides essential distributed programming tools to tackle the challenges of running ML at scale: Big Data (many data samples) and Big Models (very large parameter and intermediate variable spaces). Unlike general-purpose distributed programming platforms, Petuum is designed specifically for ML algorithms. This means that Petuum takes advantage of data correlation, staleness, and other statistical properties to maximize the performance for ML algorithms, realized through core features such as Bosen, a bounded-asynchronous key-value store, and Strads, a scheduler for iterative ML computations.

In addition to distributed ML programming tools, Petuum comes with many distributed ML algorithms, all implemented on top of the Petuum framework for speed and scalability. Please refer to the Petuum wiki for a full listing: `<https://github.com/petuum/bosen/wiki>`_.

The Petuum project is organized into 4 open-source (BSD 3-clause license) Github repositories:

* `Bosen (C++ bounded-async key-value store) <https://github.com/petuum/bosen>`_
* `Strads (C++ model-parallel scheduler) <https://github.com/petuum/strads>`_
* `JBosen (Java bounded-async key-value store) <https://github.com/petuum/jbosen>`_
* `Poseidon (Deep Learning framework) <https://github.com/petuum/poseidon>`_

The name Petuum comes from "perpetuum mobile," which is a musical style characterized by a continuous steady stream of notes. Paganini's Moto Perpetuo is an excellent example. It is our goal to build a system that runs efficiently and reliably -- in perpetual motion.

.. Welcome to Petuum's documentation!
.. ==================================

.. Contents:

.. toctree::
   :maxdepth: 2



Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

