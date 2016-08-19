Petuum Tutorial and Quick Start
========
The fastest way to get started with Petuum is to use this [script](https://gist.github.com/holyglenn/dc3a2b8a5d496735a0a297b0d5ec3479/raw/47442c52181545f40b4302c6ebdb19c25c75d433/petuum.py), which will setup Bösen and Strads systems on a single machine with just 1 command. 
After setting it up, you can run two demo applications to verify that they are working.
To get the script:
```
wget https://gist.githubusercontent.com/holyglenn/dc3a2b8a5d496735a0a297b0d5ec3479/raw/2b21c2cf23d0360d2b4760e92fdb308ab263dd49/petuum.py
```

Before start, run the following command to prepare compilation environment.
This is the only setup where sudo is required.
If you do not have sudo privilege, please contact your administrator for help.
```
sudo apt-get -y update && sudo apt-get -y install g++ make autoconf git libtool uuid-dev openssh-server cmake libopenmpi-dev openmpi-bin libssl-dev libnuma-dev python-dev python-numpy python-scipy python-yaml protobuf-compiler subversion libxml2-dev libxslt-dev zlibc zlib1g zlib1g-dev libbz2-1.0 libbz2-dev
```

After getting the compilation environment ready, you are good to run Petuum with or without sudo.

If you have sudo privilege, you can install part of Petuum's dependencies to save compilation time.
```
sudo apt-get -y install libgoogle-glog-dev libzmq3-dev libyaml-cpp-dev \
  libgoogle-perftools-dev libsnappy-dev libsparsehash-dev libgflags-dev libeigen3-dev
```
Then run the following command to setup Petuum, which takes approximately 10 minutes on a 2-core machine.
The script will first enable passwordless ssh connection to localhost using default id_rsa.pub key or generate one if without.
Then it will download and compile Bösen and Strads systems and their customized dependencies.
By default Petuum is under `~/petuum_test`. 
```
python petuum.py setup
```



If you don't have sudo, run the setup command with `--no-sudo` argument. 
In addition to the setup process above, the script will compile all Petuum's dependencies and install them in its local folder.
This process takes about 20 minutes.
```
python petuum.py setup --no-sudo
```

Now Petuum is ready to go. To run the Multi-class Logistic Regression demo (in Bösen system), 
```
python petuum.py run_mlr
```
The app launches locally and trains multi-class logistic regression model using a subset of the [Covertype dataset](https://archive.ics.uci.edu/ml/datasets/Covertype). You should see something like below. The numbers will be slightly different as it's executed indeterministically with multi-threads. 
```
40 400 0.253846 0.61287 520 0.180000 50 7.43618
I0701 00:35:00.550900  9086 mlr_engine.cpp:298] Final eval: 40 400 train-0-1: 0.253846 train-entropy: 0.61287 num-train-used: 520 test-0-1: 0.180000 num-test-used: 50 time: 7.43618
I0701 00:35:00.551867  9086 mlr_engine.cpp:425] Loss up to 40 (exclusive) is saved to /home/ubuntu/petuum/app/mlr/out.loss in 0.000955387
I0701 00:35:00.552652  9086 mlr_sgd_solver.cpp:160] Saved weight to /home/ubuntu/petuum/app/mlr/out.weight
I0701 00:35:00.553907  9031 mlr_main.cpp:150] MLR finished and shut down!
```

To run the MedLDA supervised topic model (in STRADS system), run
```
python petuum.py run_lda
```
The app launches 3 workers locally and trains with [20 newsgroup](http://qwone.com/~jason/20Newsgroups/) dataset. You will see outputs like below. Once all workers have reported "Ready to exit program", you may Ctrl-C to terminate the program.
```
......
Rank (2) Ready to exit program from main function in ldall.cpp
I1222 20:38:31.271615  2687 trainer.cpp:464] (rank:0) Dict written into /tmp/dump_dict
I1222 20:38:31.271632  2687 trainer.cpp:465] (rank:0) Total num of words: 53485
I1222 20:38:46.930896  2687 trainer.cpp:487] (rank:0) Model written into /tmp/dump_model
Rank (0) Ready to exit program from main function in ldall.cpp
```
Use the following command to display top 10 words in each of the topics that's just generated.
```
python petuum.py display_topics
```

If you seek further deployment or prefer a more detailed hands-on experience, please refer to the [full installation guide](installation.md) or the [manual](index.md).
Also check out [Poseiden](https://github.com/petuum/poseidon/wiki#quick-start), the multi-GPU distributed deep learning framework of Petuum.
