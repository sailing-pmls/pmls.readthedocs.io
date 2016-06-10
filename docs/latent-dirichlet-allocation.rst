Latent Dirichlet Allocation (LDA)
=================================

After compiling Strads, we can test that it is working correctly with Latent Dirichlet Allocation, a popular algorithm for topic modeling of documents.

Introduction to LDA
-------------------

Topic modeling, a.k.a Latent Dirichlet Allocation (LDA), is an algorithm that discovers latent semantic structure from documents. LDA finds global topics, which are weighted vocabularies, and the topical composition of each document in the collection.

In Petuum v1.0, our LDA app uses a new model-parallel Gibbs sampling scheme described in this 2014 `NIPS paper <http://www.cs.cmu.edu/~epxing/papers/2014/STRADS_NIPS14.pdf>`_, and implemented on top of the Strads scheduler. The documents are partitioned onto different machines, which take turns to sample disjoint subsets of words. By keeping the word subsets disjoint, our model-parallel implementation exhibits improved convergence times and memory utilization over data-parallel strategies. We use the sparse Gibbs sampling procedure in Yao et al (2009).

Performance
-----------

The Strads LDA app can train an LDA model with 1K topics, from a corpus with 8M documents and vocabulary size 140K, in 1000 seconds (17 minutes) using 25 machines (16 cores each).

Quickstart
-----------

Petuum LDA is implemented on Strads, and can be found in :code:`strads/apps/lda_release/`. From this point on, all instructions will assume you are in :code:`strads/apps/lda_release/`. After building Strads (as explained under Installation), you may build the LDA app from :code`strads/apps/lda_release/` by running
::
  make

Test the app (on your local machine) by running
::
  ./run.py

This will learn 1000 topics from a small subset of the NYtimes dataset, and output the word-topic and doc-topic tables to :code:`tmplog/wt-mach-*` and `tmplog/dt-mach-*` respectively.

Input data format
-----------------

The LDA app takes a single file as input, with the following format:
::
  0 dummyword word-id word-id ...
  1 dummyword word-id word-id ...
  2 dummyword word-id word-id ...
  ...

**Caution: the input file must use UNIX line endings. Windows or Mac line endings will cause a crash.**

Each line represents a single document: the first item is the document ID (0-indexed), followed by any character string (represented by :code:`dummyword`), and finally a list of tokens in the document, each represented by its word ID.

# Output format

The LDA app outputs two types of files: word-topic tables :code:`tmplog/wt-mach-*` and doc-topic tables :code:`tmplog/dt-mach-*`. The word-topic tables use this format:
::
  word-id, topic-id count, topic-id count, topic-id count ...
  word-id, topic-id count, topic-id count, topic-id count ...
  ...

Each line contains, for a particular word :code:`word-id`, all the topics `topic-id` that word is seen in, and the number of times `count` that word is seen in each topic.

The doc-topic tables follow an identical format:

```
doc-id, topic-id count, topic-id count, topic-id count ...
doc-id, topic-id count, topic-id count, topic-id count ...
...
```

The number of files `wt-mach-*` and `dt-mach-*` depends on the number of worker processes used --- see the next section for more information.

Program options
---------------

The LDA app is launched using a python script, e.g. `run.py` used earlier:

```
#!/usr/bin/python
import os
import sys

datafile = ['./sampledata/nytimes_subset.id']
topics = [' 1000 ']
iterations = [' 3 ']
threads = [' 16 ']

machfile = ['./singlemach.vm']

prog = ['./bin/ldall ']
os.system("mpirun -machinefile "+machfile[0]+" "+prog[0]+" --machfile "+machfile[0]+" -threads "+threads[0]+" -num_topic "+topics[0]+" -num_iter "+iterations[0]+" -data_file "+datafile[0]+" -logfile tmplog/1 -wtfile_pre tmplog/wt -dtfile_pre tmplog/dt ");
```

The basic options are:
* `datafile`: Path to the data file, which must be visible to all machines. If using multiple machines, provide the full path to the data file.
* `topics`: How many topics to find.
* `iterations`: How many iterations to run.
* `threads`: How many threads to use for each Worker process.
* `machfile`: Strads machine file; see below for details.

Strads requires a machine file - `singlemach.vm` in the above example. Strads machine files control which machines house Workers, the Scheduler, and the Coordinator (the 3 architectural elements of Strads). In `singlemach.vm`, we spawn all element processes on the local machine `127.0.0.1`, so the file simply looks like this:

```
127.0.0.1
127.0.0.1
127.0.0.1
127.0.0.1
```

To prepare a multi-machine file, please refer to the Strads section under [[Configuration Files for Petuum Apps|Configuration-files-for-Petuum-apps]].
