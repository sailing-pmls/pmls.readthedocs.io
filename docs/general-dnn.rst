# Deep Neural Network
This app implements a fully-connected Deep Neural Network (DNN) for multi-class classification, on **Bösen**. The DNN consists of an input layer, arbitrary number of hidden layers and an output layer. Each layer contains a certain amount of neuron units. Each unit in the input layer corresponds to an element in the feature vector. We represent the class label using 1-of-K coding and each unit in the output layer corresponds to a class label. The number of hidden layers and the number of units in each hidden layer are configured by the users. Units between adjacent layers are fully connected. In terms of DNN learning, we use the cross entropy loss and stochastic gradient descent where the gradient is computed using backpropagation method.

### Performance

On a dataset with 1M data instances; 360 feature dimension; 24M parameters, the Bösen DNN implementation converges in approximately 45 minutes, using 6 machines (16 cores each).

[[ http://www.cs.cmu.edu/~pengtaox/petuum_results/dnn_cvg.png | height = 200px ]]

Going from 1 to 6 machines results in a speedup of roughly 4.1x.

# Quick start
The DNN app can be found in `bosen/app/dnn`. From this point on, all instructions will assume you are in `bosen/app/dnn`. After building Petuum (as explained earlier in this manual), you can build the DNN app from `bosen/app/dnn` by running
```
make
```
This will put the DNN binary in the subdirectory `bin/`.

Create a simulated dataset:
```
script/gen_data.sh 10000 360 2001 1 datasets
```
Change `app_dir` in `script/run_local.py`; see example below. You need to provide the full path to the DNN directory. This will allow for correct YARN operation (if required).
```
app_dir = "/home/user/bosen/app/dnn"
```
then you can test that the app works on the local machine (with 4 worker threads). From the `bosen/app/dml` directory, run:
```
./script/launch.py
```
The DNN app runs in the background (progress is output to stdout). After the app terminates, you should get 2 output files:
```
datasets/weights.txt
datasets/biases.txt
```
`weights.txt` saves the weight matrices. The order is: weight matrix between layer 1 (input layer) and layer 2 (the first hidden layer), weight matrix between layer 2 and layer 3, etc. All matrices are saved in row major order and each line corresponds to a row.

### Making predictions
Change `app_dir` in `script/predict.py`; see example below. You need to provide the full path to the DNN directory.

```
app_dir = "/home/user/bosen/app/dnn"
```

You can now make predictions with the learned model using
```
./script/launch_pred.py
```
After the app terminates, you should get a prediction result file for each of the input data file. Please check the directory where you put the data files. Each line of the prediction result file contains the prediction for the corresponding data.

# Input data format
We assume users have partitioned the data into M pieces, where M is the total number of clients (machines). Each client will be in charge of one piece. User needs to provide a file recording the data partition information. In this file, each line corresponds to one data partition. The format of each line is
```
<data_file> \t <num_data_in_partition>
```

`<num_data_in_partition>` is the number of data points in this partition. `<data_file>` stores the class label and feature vector of a data sample in each line. `<data_file>` must be an absolute path. The format of each line in `<data_file>` is:
```
<class_label> \t <feature vector>
```
Elements in the feature vector are separated with single blank. Note that class label starts from 0. If there are K classes, the range of class labels are `[0,1,...,K-1]`.

# Creating synthetic data

We provide a synthetic data generator for testing purposes. The generator generates random data and automatically partitions the data into different files. To see detailed instructions, run
```
scripts/gen_data.sh
```
The basic syntax is
```
scripts/gen_data.sh   <num_train_data> <dim_feature> <num_classes> <num_partitions> <save_dir>
```

- `<num_train_data>`: number of training data
- `<dim_feature>`: dimension of features
- `<num_classes>`: number of classes
- `<num_partitions>`: how many pieces the data is going to be partitioned into
- `<save_dir>`: the directory where the generated data will be saved

The generator will generate an amount of `<num_partitions>` files storing class labels and feature vectors. The data files are automatically named as 0_data.txt, 1_data.txt, etc. The generator will also generate a file `data_ptt_file.txt` where each line stores the data file of one data partition and the number of points in this partition.

For example, you can create a dataset by running the following command:
```
scripts/gen_data.sh 10000 360 2001 3 datasets
```
It will generate a dataset where the number of training examples is 10000, feature dimension is 360 and number of classes are 2001. Moreover, it will partition the dataset into 3 pieces and save the generated files to `datasets/`: 0_data.txt, 1_data.txt, 2_data.txt and a data_ptt_file.txt file which will have three lines. Example:
```
/home/user/bosen/app/dnn/datasets/0_data.txt    3333
/home/user/bosen/app/dnn/datasets/1_data.txt    3333
/home/user/bosen/app/dnn/datasets/2_data.txt    3334
```
Each line corresponds to one data partition and contains the data file of this partition and number of data points in this partition. Note that the path of data file must be an absolute path.

# Running the Deep Neural Network Application
Parameters of the DNN training are specified in `script/run_local.py`, including:
- `<num_worker_threads>`: how many worker threads to use in each machine
- `<staleness>`: staleness value
- `<parafile>`: configuration file on DNN parameters
- `<data_ptt_file>`: a file containing the data file path and the number of training points in each data partition
- `<model_weight_file>`: the path where the output weight matrices will be stored
- `<model_bias_file>`: the path there the output bias vectors will be stored

The machine file is specified in `script/launch.py`:
```
hostfile_name = "machinefiles/localserver"
```

After configuring these parameters, perform DNN training by:
```
./script/launch.py
```

Parameters of the DNN prediction are specified in `script/predict.py`, including:
- `<parafile>`: configuration file on DNN parameters
- `<data_ptt_file>`: a file containing the data file path and the number of training points in each data partition
- `<model_weight_file>`: the file storing the weight matrices
- `<model_bias_file>`: the file storing the bias vectors

The machine file is specified in `script/launch_pred.py`:
```
hostfile_name = "machinefiles/localserver"
```
After configuring these parameters, perform prediction by:
```
./script/launch_pred.py
```

# Format of DNN Configuration File

The DNN configurations are stored in <parameter_file>. Each line corresponds to a parameter and its format is
```
<parameter_name>: <parameter_value>
```

`<parameter_name>` is the name of the parameter. It is followed by a `:` (there is no blank between `<parameter_name>` and `:`). `<parameter_value>` is the value of this parameter. Note that `:` and `<parameter_value>` must be separated by a blank.

The list of parameters and their meanings are:
- `num_layers`: number of layers, including input layer, hidden layers, and output layer
- `num_units_in_each_layer`: number of units in each layer
- `num_epochs`: number of epochs in stochastic gradient descent training
- `stepsize`: learn rate of stochastic gradient descent
- `mini_batch_size`: mini batch size in each iteration
- `num_smp_evaluate`: when evaluating the objective function, we randomly sample <num_smp_evaluate> points to compute the objective
-`num_iters_evaluate`: every <num_iters_evaluate> iterations, we do an objective function evaluation
Note that, the order of the parameters cannot be switched. 

Here is an example:
```
num_layers: 6
num_units_in_each_layer: 360 512 512 512 512 2001
num_epochs: 2
stepsize: 0.05
mini_batch_size: 10
num_smp_evaluate: 2000
num_iters_evaluate: 100
```

# Terminating the DNN app
The DNN app runs in the background, and outputs its progress to stdout. If you need to terminate the app before it finishes, run
```
./script/kill.py <petuum_ps_hostfile>
```


## File IO from HDFS

Put datasets to HDFS
```
hadoop fs -mkdir -p /user/bosen/dataset/dnn
hadoop fs -put datasets /user/bosen/dataset/dnn
```

Change the corresponding file paths in `script/run_local.py` to the right HDFS path. Comment out the local path.
```
# , "parafile": join(app_dir, "datasets/para_imnet.txt")
, "parafile": "hdfs://hdfs-domain/user/bosen/dataset/dnn/datasets/para_imnet.txt"
# , "data_ptt_file": join(app_dir, "datasets/data_ptt_file.txt")
, "data_ptt_file": "hdfs://hdfs-domain/user/bosen/dataset/dnn/datasets/data_ptt_file.txt"
# , "model_weight_file": join(app_dir, "datasets/weights.txt")
, "model_weight_file": "hdfs://hdfs-domain/user/bosen/dataset/dnn/datasets/weights.txt"
# , "model_bias_file": join(app_dir, "datasets/biases.txt")
, "model_bias_file": "hdfs://hdfs-domain/user/bosen/dataset/dnn/datasets/biases.txt"
```
Launch it over ssh
```
./script/launch.py
```

Check the output
```
hadoop fs -cat /user/bosen/dataset/dnn/datasets/biases.txt
```

Similar configurations apply to DNN prediction.

## Use Yarn to launch DNN app
```
./script/launch_on_yarn.py
```
