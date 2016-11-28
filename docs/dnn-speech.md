# Deep Neural Network for Speech Recognition

This tutorial shows how the Deep Neural Network (DNN) application (implemented on **Bösen**) can be applied to speech recognition, using Kaldi (<http://kaldi.sourceforge.net/about.html>) as our tool for feature extraction and decoding. Kaldi is a toolkit for speech recognition written in C++ and licensed under the Apache License v2.0. It also provides two DNN applications (<http://kaldi.sourceforge.net/dnn.html>), and we follow Dan's setting in feature extraction, preprocessing and decoding.

Our DNN consists of an input layer, arbitrary number of hidden layers and an output layer. Each layer contains a certain amount of neuron units. Each unit in the input layer corresponds to an element in the feature vector. We represent the class label using 1-of-K coding and each unit in the output layer corresponds to a class label. The number of hidden layers and the number of units in each hidden layer are configured by the users. Units between adjacent layers are fully connected. In terms of DNN learning, we use the cross entropy loss and stochastic gradient descent where the gradient is computed using backpropagation method. 

## Installation
### PMLS Deep Neural Network Application
The DNN for Speech Recognition app can be found in `bosen/app/dnn_speech/`. From this point on, all instructions will assume you are in `bosen/app/dnn_speech/`. After building PMLS (as explained earlier in this manual), you can build the DNN from `bosen/app/dnn_speech/` by running

```
make
```

This will put the DNN binary in the subdirectory `bin/`.

### Kaldi

From `bosen/app/dnn_speech/`, extract Kaldi by running:
```
tar -xvf kaldi-trunk.tar.gz
cd kaldi-trunk/tools
```

Next, we must build the ATLAS libraries in a local directory. From `kaldi-trunk/tools`,
```
sudo apt-get install gfortran
./install_atlas.sh
```
This process will take some time, and will report some messages of the form `Error 1 (ignored)` - this is normal. More details can be found in `kaldi-trunk/INSTALL`, `kaldi-trunk/tools/INSTALL` and `kaldi-trunk/src/INSTALL`.

Once ATLAS has been set up,
```
make
cd ../src/
./configure
make depend
make
```
The first `make` may produce some "Error 1 (ignored)" messages, this is normal and can be ignored. The `./configure` may produce a warning about GCC 4.8.2, which can also be ignored for our purposes. Be advised that these steps will take a while (up to 1-2 hours).

## The Whole Pipeline

Currently, we only support TIMIT dataset (<https://catalog.ldc.upenn.edu/LDC93S1>), a well-known benchmark dataset for speech recognition. You can process this dataset through following steps.

### 1. Feature extraction

**WARNING: this stage will take several hours, and requires at least 16GB free RAM.**

Run 
```
sh scripts/PrepDNNFeature.sh <TIMIT_path>
```
where `<TIMIT_path>` is the **absolute** path to the TIMIT directory. This will extract features and do some preprocessing work to generate `Train.fea`, `Train.label`, `Train.para`, `head.txt` and `tail.txt` in `app/dnn_speech` directory and `exp/petuum_dnn` in `kaldi-trunk/egs/timit/s5` directory. The script will take 1-2 hours to complete.

- `<TIMIT_path>` is the absolute path of TIMIT dataset. You can get it through <https://catalog.ldc.upenn.edu/LDC93S1>.
- `Train.fea` and `Train.label` save the feature and label of training examples, one example per line.
- `Train.para` saves the information of training examples, including the feature dimension, the number of classes of labels and the number of examples.
- `head.txt` contains the transistion model and the preprocessing information of training features, including splicing and linear discriminant analysis (LDA), saved in the format of Dan's setup in Kaldi.
- `tail.txt` contains the empirical distribution of the classes of labels, saved in the format of Dan's setup in Kaldi.
- `kaldi-trunk/egs/timit/s5/exp/petuum_dnn` contains all the log file and intermediate results.

### 2. DNN Training
According to the information in `Train.para`, you can set the configuration file for DNN (more details can be found in `Input data format` and `Format of DNN Configuration file` section). For example, if `Train.para` is
```
360 2001 1031950
```
You can set `datasets/data_partition.txt` by
```
/home/user/bosen/app/dnn_speech/Train 1031950
```
and `datasets/para_imnet.txt` by
```
num_layers: 4
num_units_in_each_layer: 360 512 512 2001
num_epochs: 2
stepsize: 0.1
mini_batch_size: 256
num_smp_evaluate: 2000
num_iters_evaluate: 100
```
And run 
```
scripts/run_dnn.sh 4 5 machinefiles/localserver datasets/para_imnet.txt datasets/data_partition.txt DNN_para.txt
```

The DNN app runs in the background (progress is output to stdout). After the app terminates, you should get 1 output files:
```
DNN_para.txt
```
which stores weight matrices and bias vectors as Dan's setup in Kaldi. More details can be found in next section.

### 3. Decoding
Run
```
scripts/NetworkDecode.sh DNN_para.txt datasets/para_imnet.txt
```
Wait for several minutes and you will see the decode result over the core test dataset of TIMIT. 

## Running the Deep Neural Network application

Notice that the interface of DNN for speech recognition is slightly different from the general purpose DNN in `app/dnn`. To see the instructions for the DNN for speech app, run
```
scripts/run_dnn.sh
```
The basic syntax is
```
scripts/run_dnn.sh   <num_worker_threads> <staleness> <hostfile> <parameter_file> <data_partition_file> <model_para_file> "additional options"
```
- `<num_worker_threads>`: how many worker threads to use in each machine
- `<staleness>`: staleness value
- `<hostfile>`: machine configuration file
- `<parameter_file>`: configuration file on DNN parameters
- `<data_partition_file>`: a file containing the data file path and the number of training points in each data partition
- `<model_para_file>`: the path where the output weight matrices and bias vector will be stored

The final argument, "additional options", is an optional quote-enclosed string of the form `"--opt1 x --opt2 y ..."` (you may omit this if you wish). This is used to pass in the following optional arguments:

- `ps_snapshot_clock x`: take snapshots every x iterations
- `ps_snapshot_dir x`: save snapshots to directory x (please make sure x already exists!)
- `ps_resume_clock x`: if specified, resume from iteration x (note: if --staleness s is specified, then we resume from iteration x-s instead)
- `ps_resume_dir x`: resume from snapshots in directory x. You can continue to take snapshots by specifying 
- `ps_snapshot_dir y`, but do make sure directory y is not the same as x!

For example, to run the DNN app on local machine (one client) where the number of worker thread is 4, staleness is 5, machine file is `machinefiles/localserver`, DNN configuration file is `datasets/para_imnet.txt`, data partition file is `datasets/data_partition.txt`, model parameter file is `DNN_para.txt`, use the following command:
```
scripts/run_dnn.sh 4 5 machinefiles/localserver datasets/para_imnet.txt datasets/data_partition.txt DNN_para.txt
```

## Input data format
We assume users have partitioned the data into M pieces, where M is the total number of clients (machines). Each client will be in charge of one piece. User needs to provide a file recording the data partition information. In this file, each line corresponds to one data partition. The format of each line is
```
<data_file> \t <num_data_in_partition>
```

`<num_data_in_partition>` is the number of data points in this partition. `<data_file>` is the prefix of the class label file (`<data_file>.label`) and the feature file (`<data_file>.fea`). And `<data_file>` must be an absolute path.

For example, 
```
/home/user/bosen/app/dnn_speech/Train 1031950
```

means there are 1031950 training examples, the class label file is `/home/user/bosen/app/dnn_speech/Train.label` and the feature file is `/home/user/bosen/app/dnn_speech/Train.fea`.

The format of `<data_file>.fea` is:
```
<feature vector 1>
<feature vector 2>
...
```
Elements in the feature vector are separated with single blank. 

The format of `<data_file>.label` is
```
<label 1>
<label 2>
...
```

Note that class label starts from 0. If there are K classes, the range of class labels are [0,1,...,K-1].


## Format of DNN Configuration File

The DNN configurations are stored in `<parameter_file>`. Each line corresponds to a parameter and its format is
```
<parameter_name>: <parameter_value>
```

`<parameter_name>` is the name of the parameter. It is followed by a `:` (there is no blank between `<parameter_name>` and :). `<parameter_value>` is the value of this parameter. Note that `:` and `<parameter_value>` must be separated by a blank.

The list of parameters and their meanings are:
- `num_layers`: number of layers, including input layer, hidden layers, and output layer
- `num_units_in_each_layer`: number of units in each layer
- `num_epochs`: number of epochs in stochastic gradient descent training
- `stepsize`: learn rate of stochastic gradient descent
- `mini_batch_size`: mini batch size in each iteration
- `num_smp_evaluate`: when evaluating the objective function, we randomly sample `<num_smp_evaluate>` points to compute the objective
- `num_iters_evaluate`: every `<num_iters_evaluate>` iterations, we do an objective function evaluation
Note that, the order of the parameters cannot be switched. 

Here is an example:
```
num_layers: 4
num_units_in_each_layer: 360 512 512 2001
num_epochs: 2
stepsize: 0.1
mini_batch_size: 256
num_smp_evaluate: 2000
num_iters_evaluate: 100
```

## Output format
The DNN app outputs just one file:

```
<model_para_file>
```

`<model_para_file>` saves the weight matrices and bias vectors. The order is: weight matrix between layer 1 (input layer) and layer 2 (the first hidden layer), bias vector for layer 2, weight matrix between layer 2 and layer 3, bias vector for layer 3, etc. All matrices are saved in row major order and each line corresponds to a row. Elements in each row are separated with blank.

## Terminating the DNN app
The DNN app runs in the background, and outputs its progress to stdout. If you need to terminate the app before it finishes, for distributed version, run

```
scripts/kill_dnn.sh <hostfile>
```
