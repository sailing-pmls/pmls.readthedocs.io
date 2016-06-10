Maximum entropy discrimination latent Dirichlet allocation (MedLDA)
===================================================================

MedLDA (`JMLR'12 paper <http://bigml.cs.tsinghua.edu.cn/~jun/pub/MedLDA_jmlr.pdf>`_) is a supervised topic model that jointly learns classifier and latent topic representation from the text, by integrating max-margin principle to hierarchical Bayesian topic model. Here we provide multi-class MedLDA using Gibbs sampling algorithm described in the `KDD'13 paper <http://bigml.cs.tsinghua.edu.cn/~jun/large-scale-gibbs-medlda.pdf>`_.

MedLDA is built on top of the **Strads** scheduler system, but uses a data-parallel style. A similar (though more complicated) implementation can be found `here <http://bigml.cs.tsinghua.edu.cn/~jun/gibbs-medlda.shtml>`_.

Performance
-----------

Using 20 machines (12 cores each), the Strads MedLDA app can solve for 1.1 million documents, 20 labels, 1000 topics in 5000 seconds.

Installation
------------

The application can be found at :code:`strads/apps/medlda_release/`. All subsequent operations are done under this directory:
::
  cd strads/apps/medlda_release/

Assuming you have completed the Strads installation instructions,
::
  make

will do the right job. The generated binary executable is located under :code:`bin/`. 

Data Preparation
----------------

This application takes inputs in LIBSVM format:
::
  <label>...<word>:<count>...

For instance, :code:`21 5 cogito:1 sum:1 ergo:1` represents a short document :code:`Cogito ergo sum` labeled as 21 and 5. Note that :code:`<word>` is considered as a string while :code:`<label>` should be an integer in :code:`[0, num_label)`.

We included a toy data set `20newsgroups <http://qwone.com/~jason/20Newsgroups/>` under the app directory for demo purpose. If you type
::
  wc -l 20news.{train,test}

you'll see the training file contains 11269 documents and the test file contains 7505 documents. 

The data needs to be partitioned according to the number of workers: e.g. if you use 3 workers, you will need 3 partitions. To partition the data, run
::
  python split.py <filename> <num_worker>

For instance,
::
  python split.py 20news.train 3
  python split.py 20news.test 3

randomly partitions the training and testing corpus into three subsets: :code:`20news.train.0`, :code:`20news.train.1`, :code:`20news.train.2`, and :code:`20news.test.0`, :code:`20news.test.1`, :code:`20news.test.2`.

If your cluster doesn't support Network File System (NFS), don't forget to :code:`scp` or :code:`rsync` files to the right host. The workers will determine which file to read according to the file suffix, e.g., :code:`worker_0` reads :code:`<filename>.0`.

If your cluster supports NFS or you only intend to do a single machine experiment, you're ready to go.

Quick example on a single machine
---------------------------------

We will train MedLDA on 20newsgroups dataset on a single machine. The dataset and machine configuration files are provided in the package. It will spawn 3 servers and 3 workers, each worker running 2 threads.
You can then execute:
::
  python single.py

It will run the :code:`medlda` binary using default flag settings. You will see outputs like:
::
  ......
  I1222 20:36:42.225085  2688 trainer.cpp:78] (rank:1) num train doc: 3756
  I1222 20:36:42.225152  2688 trainer.cpp:79] (rank:1) num train word: 37723
  I1222 20:36:42.225159  2688 trainer.cpp:80] (rank:1) num train token: 446496
  I1222 20:36:42.225164  2688 trainer.cpp:81] (rank:1) ---------------------------------------------------------------------
  I1222 20:36:42.236769  2689 trainer.cpp:78] (rank:2) num train doc: 3756
  I1222 20:36:42.236814  2689 trainer.cpp:79] (rank:2) num train word: 37111
  I1222 20:36:42.236821  2689 trainer.cpp:80] (rank:2) num train token: 426452
  I1222 20:36:42.236827  2689 trainer.cpp:81] (rank:2) ---------------------------------------------------------------------
  I1222 20:36:42.238376  2687 trainer.cpp:78] (rank:0) num train doc: 3757
  I1222 20:36:42.238426  2687 trainer.cpp:79] (rank:0) num train word: 37572
  I1222 20:36:42.238435  2687 trainer.cpp:80] (rank:0) num train token: 445351
  I1222 20:36:42.238440  2687 trainer.cpp:81] (rank:0) ---------------------------------------------------------------------
  ......
  I1222 20:38:18.517712  2689 trainer.cpp:139] (rank:2) Burn-in Iteration 39  1.99362 sec
  I1222 20:38:18.517719  2688 trainer.cpp:139] (rank:1) Burn-in Iteration 39  1.99363 sec
  I1222 20:38:18.517725  2687 trainer.cpp:139] (rank:0) Burn-in Iteration 39  1.99362 sec
  I1222 20:38:20.451874  2687 trainer.cpp:139] (rank:0) Burn-in Iteration 40  1.93407 sec
  I1222 20:38:20.451865  2689 trainer.cpp:139] (rank:2) Burn-in Iteration 40  1.93407 sec
  I1222 20:38:20.451872  2688 trainer.cpp:139] (rank:1) Burn-in Iteration 40  1.93408 sec
  I1222 20:38:20.456595  2689 trainer.cpp:374] (rank:2) ---------------------------------------------------------------------
  I1222 20:38:20.456612  2689 trainer.cpp:375] (rank:2)     Elapsed time: 98.1856 sec   Train Accuracy: 0.999734 (3755/3756)
  I1222 20:38:20.456607  2687 trainer.cpp:374] (rank:0) ---------------------------------------------------------------------
  I1222 20:38:20.456634  2689 trainer.cpp:378] (rank:2) ---------------------------------------------------------------------
  I1222 20:38:20.456622  2687 trainer.cpp:375] (rank:0)     Elapsed time: 98.1783 sec   Train Accuracy: 0.999468 (3755/3757)
  I1222 20:38:20.456643  2687 trainer.cpp:378] (rank:0) ---------------------------------------------------------------------
  I1222 20:38:20.457993  2688 trainer.cpp:374] (rank:1) ---------------------------------------------------------------------
  I1222 20:38:20.458014  2688 trainer.cpp:375] (rank:1)     Elapsed time: 98.1953 sec   Train Accuracy: 0.999734 (3755/3756)
  I1222 20:38:20.458036  2688 trainer.cpp:378] (rank:1) ---------------------------------------------------------------------
  ......
  I1222 20:38:30.521900  2688 trainer.cpp:398] (rank:1) Train prediction written into /tmp/dump_train_pred.1
  I1222 20:38:30.526638  2689 trainer.cpp:398] (rank:2) Train prediction written into /tmp/dump_train_pred.2
  I1222 20:38:30.592419  2687 trainer.cpp:398] (rank:0) Train prediction written into /tmp/dump_train_pred.0
  I1222 20:38:31.044430  2687 trainer.cpp:403] (rank:0) Train doc stats written into /tmp/dump_train_doc.0
  I1222 20:38:31.076773  2689 trainer.cpp:403] (rank:2) Train doc stats written into /tmp/dump_train_doc.2
  I1222 20:38:31.213727  2688 trainer.cpp:403] (rank:1) Train doc stats written into /tmp/dump_train_doc.1
  Rank (1) Ready to exit program from main function in ldall.cpp
  I1222 20:38:31.256194  2687 trainer.cpp:449] (rank:0) Hyperparams written into /tmp/dump_param
  I1222 20:38:31.259068  2687 trainer.cpp:454] (rank:0) Classifier written into /tmp/dump_classifier
  Rank (2) Ready to exit program from main function in ldall.cpp
  I1222 20:38:31.271615  2687 trainer.cpp:464] (rank:0) Dict written into /tmp/dump_dict
  I1222 20:38:31.271632  2687 trainer.cpp:465] (rank:0) Total num of words: 53485
  I1222 20:38:46.930896  2687 trainer.cpp:487] (rank:0) Model written into /tmp/dump_model
  Rank (0) Ready to exit program from main function in ldall.cpp

Once all workers have reported :code:`Ready to exit program`, you may :code:`Ctrl-c` to terminate the program.

As the last few lines suggest, the training results will be stored at :code:`/tmp/dump_*` by default. Specifically, (let D = num of docs in a partition, L = num of labels, and K = num of topics)
* :code:`_train_pred.x` stores the predicted label of partition :code:`x`. (D x 1 integer vector)
* :code:`_train_doc.x` stores the doc-topic distribution in log scale. (D x K matrix)
* :code:`_param` stores the value of :code:`alpha`, :code:`beta`, :code:`num_topic`, and :code:`num_label`. 
* :code:`_classifier` stores the classifier weights. (K x L matrix, each column is a binary classifier)
* :code:`_dict` stores the aggregated distinct words appeared in the train corpus.
* :code:`_model` stores the topic-word distribution in log scale. (K x V matrix)

Now we're ready for test. You can run
::
  python single_test.py

It will load the model files generated at the training phase and perform inference on test documents. You will see outputs like:
::
  ......
  I1222 20:39:30.173037  3258 tester.cpp:24] (rank:1) Hyperparams loaded from /tmp/dump_param
  I1222 20:39:30.173049  3259 tester.cpp:24] (rank:2) Hyperparams loaded from /tmp/dump_param
  I1222 20:39:30.173061  3258 tester.cpp:25] (rank:1) Alpha: 0.16 Beta: 0.01 Num Topic: 40 Num Label: 20
  I1222 20:39:30.173069  3259 tester.cpp:25] (rank:2) Alpha: 0.16 Beta: 0.01 Num Topic: 40 Num Label: 20
  I1222 20:39:30.173780  3257 tester.cpp:31] (rank:0) Classifier loaded from /tmp/dump_classifier
  I1222 20:39:30.176692  3258 tester.cpp:31] (rank:1) Classifier loaded from /tmp/dump_classifier
  I1222 20:39:30.176772  3259 tester.cpp:31] (rank:2) Classifier loaded from /tmp/dump_classifier
  I1222 20:39:30.213495  3257 tester.cpp:44] (rank:0) Dict loaded from /tmp/dump_dict
  I1222 20:39:30.213523  3257 tester.cpp:45] (rank:0) Total num of words: 53485
  I1222 20:39:30.228713  3259 tester.cpp:44] (rank:2) Dict loaded from /tmp/dump_dict
  I1222 20:39:30.228745  3259 tester.cpp:45] (rank:2) Total num of words: 53485
  I1222 20:39:30.235925  3258 tester.cpp:44] (rank:1) Dict loaded from /tmp/dump_dict
  I1222 20:39:30.235959  3258 tester.cpp:45] (rank:1) Total num of words: 53485
  I1222 20:39:31.196068  3258 tester.cpp:51] (rank:1) Model loaded into /tmp/dump_model
  I1222 20:39:31.259271  3259 tester.cpp:51] (rank:2) Model loaded into /tmp/dump_model
  I1222 20:39:31.306517  3258 tester.cpp:95] (rank:1) num test doc: 2502
  I1222 20:39:31.306558  3258 tester.cpp:96] (rank:1) num test oov: 8212
  I1222 20:39:31.306565  3258 tester.cpp:97] (rank:1) num test token: 300103
  I1222 20:39:31.306571  3258 tester.cpp:98] (rank:1) ---------------------------------------------------------------------
  I1222 20:39:31.348415  3259 tester.cpp:95] (rank:2) num test doc: 2501
  I1222 20:39:31.348460  3259 tester.cpp:96] (rank:2) num test oov: 8227
  I1222 20:39:31.348469  3259 tester.cpp:97] (rank:2) num test token: 292610
  I1222 20:39:31.348476  3259 tester.cpp:98] (rank:2) ---------------------------------------------------------------------
  I1222 20:39:31.350323  3257 tester.cpp:51] (rank:0) Model loaded into /tmp/dump_model
  I1222 20:39:31.420966  3257 tester.cpp:95] (rank:0) num test doc: 2502
  I1222 20:39:31.421005  3257 tester.cpp:96] (rank:0) num test oov: 7172
  I1222 20:39:31.421013  3257 tester.cpp:97] (rank:0) num test token: 267530
  I1222 20:39:31.421018  3257 tester.cpp:98] (rank:0) ---------------------------------------------------------------------
  I1222 20:39:35.194511  3257 tester.cpp:118] (rank:0)     Elapsed time: 3.77297 sec   Test Accuracy: 0.797362 (1995/2502)
  I1222 20:39:35.206197  3257 tester.cpp:212] (rank:0) Test prediction written into /tmp/dump_test_pred.0
  I1222 20:39:35.307680  3259 tester.cpp:118] (rank:2)     Elapsed time: 3.95875 sec   Test Accuracy: 0.822471 (2057/2501)
  I1222 20:39:35.315475  3259 tester.cpp:212] (rank:2) Test prediction written into /tmp/dump_test_pred.2
  I1222 20:39:35.392273  3258 tester.cpp:118] (rank:1)     Elapsed time: 4.08543 sec   Test Accuracy: 0.804956 (2014/2502)
  I1222 20:39:35.404650  3258 tester.cpp:212] (rank:1) Test prediction written into /tmp/dump_test_pred.1
  I1222 20:39:35.549335  3257 tester.cpp:217] (rank:0) Test doc stats written into /tmp/dump_test_doc.0
  Rank (0) Ready for exit program from main function in ldall.cpp
  I1222 20:39:35.647891  3259 tester.cpp:217] (rank:2) Test doc stats written into /tmp/dump_test_doc.2
  Rank (2) Ready for exit program from main function in ldall.cpp
  I1222 20:39:35.724900  3258 tester.cpp:217] (rank:1) Test doc stats written into /tmp/dump_test_doc.1
  Rank (1) Ready for exit program from main function in ldall.cpp

Once all workers have reported :code:`Ready to exit program`, you may :code:`Ctrl-c` to terminate the program.

We're done! Similar to the training phase, prediction results for test documents are stored at :code:`/tmp/dump_test_doc.x` and :code:`/tmp/dump_test_pred.x`.

Configuration and using multiple machines
-----------------------------------------

Let us inspect the training script :code:`single.py`:
::
  #!/usr/bin/python
  import os
  import sys
  
  machfile = ['./singlemach.vm']
  traindata = ['./20news.train']
  numservers = ['3']
  
  prog=['./bin/medlda ']
  os.system("mpirun -machinefile "+machfile[0]+" "+prog[0]+" -machfile "+machfile[0]+" -schedulers "+numservers[0]+" -train_prefix "+traindata[0]);

Things to note:

* The last line :code:`os.system` executes the MedLDA app; you may insert advanced command line flags here. See the end of this article for a full list of flags.
* :code:`machfile` gives the machine configuration file.
* :code:`traindata` gives the dataset prefix (:code:`20news.train` in this case).
* :code:`numservers` is the number of servers (key-value stores) to use.

MedLDA is built upon the Strads scheduler architecture, and uses a similar machine configuration file. This machine file is a simple list of IP addresses, corresponding to workers, followed by servers, and finally the Strads coordinator. For example, the :code:`singlemach.vm` machine file used in :code:`single.py` looks like this:
::
  127.0.0.1       <--- this is worker 0
  127.0.0.1       <--- this is worker 1
  127.0.0.1       <--- this is worker 2
  127.0.0.1       <--- this is server 0
  127.0.0.1       <--- this is server 1
  127.0.0.1       <--- this is server 2
  127.0.0.1       <--- this is the coordinator

The last IP is always the coordinator, and the servers come immediately before it (:code:`numservers` controls the number of servers). The workers make up the remaining IPs (you must use at least 2 workers).

**Note: remember to partition your data for the correct number of workers.**

To use multiple machines, simply change the machine file IPs to point to the desired machines. **You may repeat IP addresses to assign multiple processes to the same machine, but the repeat IPs must be contiguous - :code:`ip1` followed by :code:`ip2` followed by :code:`ip1` is invalid.**

**Important: do not forget to prepare the test script (e.g. :code:`single_test.py`) in the same fashion. You must use the same machine configuration as the training script.**

Command line flags
------------------

Flags for training:
* :code:`train_prefix`: Prefix to the LIBSVM format training corpus. 
* :code:`dump_prefix`: Prefix to the dump results. 
* :code:`num_thread`: Number of worker threads. 
* :code:`alpha`: Parameter of Dirichlet prior on doc-topic distribution. 
* :code:`beta`: Parameter of Dirichlet prior on topic-word distribution. 
* :code:`cost`: Cost parameter on hinge loss, usually called "C".
* :code:`ell`: Margin parameter in SVM, usually set to 1. Hinge loss = max(0, ell - y \<w, x\>).
* :code:`num_burnin`: Number of burn-in iterations.
* :code:`num_topic`: Number of topics, usually called "K".
* :code:`num_label`: Total number of labels.
* :code:`eval_interval`: Print out information every N iterations

Flags for testing:
* :code:`test_prefix`: Prefix to LIBSVM format test corpus.
* :code:`dump_prefix`: Prefix to the dump results. 
* :code:`num_thread`: Number of worker threads.
