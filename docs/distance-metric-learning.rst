# Distance Metric Learning

This app implements Distance Metric Learning (DML) as proposed in [1], on **Bösen**. DML takes data pairs labeled either as similar or dissimilar to learn a Mahalanobis distance matrix such that similar data pairs will have small distances while dissimilar pairs are separated apart. 

[1] Eric P. Xing, Michael I. Jordan, Stuart Russell, and Andrew Y. Ng. "Distance metric learning with application to clustering with side-information." In Advances in neural information processing systems, pp. 505-512. 2002.

### Performance

On a dataset with 1M data instances; 22K feature dimension; 22M parameters, the Bösen DML implementation converges in approximately 15 minutes, using 4 machines (64 cores each).

[[ http://www.cs.cmu.edu/~pengtaox/petuum_results/dml_cvg.png | height = 200px ]]

Going from 1 to 4 machines results in a speedup of roughly 3.75x.

# Quick start

The DML app can be found in `bosen/app/dml/`. **From this point on, all instructions will assume you are in `bosen/app/dml/`.** After building Petuum (as explained earlier in this manual), you can build the DML app from `bosen/app/dml` by running

```
make
```

This will put the DML binary in the subdirectory `bin/`. 

Next, download the preprocessed MNIST dataset from [http://www.cs.cmu.edu/~pengtaox/dataset/mnist_petuum.zip](http://www.cs.cmu.edu/~pengtaox/dataset/mnist_petuum.zip). Unzip the files and copy them to the `datasets` folder. You can use the following commands:

```
cd datasets
wget http://www.cs.cmu.edu/%7Epengtaox/dataset/mnist_petuum.zip
unzip mnist_petuum.zip
cd ..
```

Change `app_dir` in `script/run_local.py`; see example below. You need to provide the full path to the DML directory. This will allow for correct YARN operation (if required).
```
app_dir = "/home/user/bosen/app/dml"
```
Using the MNIST dataset, you can test that the app works on the local machine (with 4 worker threads). From the `app/dml/` directory, run:

```
./script/launch.py
```

The DML app runs in the background (progress is output to stdout). After the app terminates, you should get one output file:

```
datasets/dismat.txt
```

`dismat.txt` saves the distance matrix in row major order and each line corresponds to a row.


# Running the Distance Metric Learning Application

Parameters of the DML app are specified in `script/run_local.py`, including:
* `<num_worker_threads>`: how many worker threads to use in each machine
* `<staleness>`: staleness value
* `<parafile>`: configuration file of DML parameters
* `<feature_file>`: file storing features of data samples
* `<simi_pairs_file>`: file storing similar pairs
* `<diff_pairs_file> `: file storing dissimilar pairs
* `<model_weight_file>`: the path where the learned distance matrix will be stored

The machine file is specified in `script/launch.py`:
```
hostfile_name = "machinefiles/localserver"
```

After configuring these parameters, launch the app by:
```
./script/launch.py
```

## Format of DML Configuration File
The DML configurations are stored in `<parameter_file>`. Each line corresponds to a parameter and its format is
```
<parameter_name>: <parameter_value>
```
`<parameter_name>` is the name of the parameter. It is followed by a `:` (there is no blank between `<parameter_name>` and `:`). `<parameter_value>` is the value of this parameter. Note that `:` and `<parameter_value>` must be separated by a blank.

The list of parameters and their meanings are:
* src_feat_dim: dimension of the observed feature space
* dst_feat_dim: dimension of the latent space
* learn_rate: learning rate of Stochastic Gradient Descent (SGD)
* epoch: number of epochs in SGD
* num_total_pts: number of data samples
* num_simi_pairs: number of similar pairs
* num_diff_pairs: number of dissimilar pairs
* mini_batch_size: size of mini-batch
* num_iters_evaluate: every `<num_iters_evaluate>` iterations, we do an objective function evaluation
* num_smp_evaluate: when evaluating the objective function, we randomly sample `<num_smp_evaluate>` points to compute the objective 

Note that, the order of the parameters cannot be switched.
Here is an example:
```
src_feat_dim: 780
dst_feat_dim: 600
lambda: 1
thre: 1
learn_rate: 0.001
epoch: 50
num_total_pts: 60000
num_simi_pairs: 1000
num_diff_pairs: 1000
mini_batch_size: 100
num_iters_evaluate: 1000
num_smps_evaluate: 1000
```
## Input data format
The input data is stored in three files
```
<feat file>
<simi pairs file>
<diff pairs file> 
```
`<feat file>` stores the features of data samples in sparse format. Each lines corresponds to one data sample. The format is 

`<class label> <num of nonzero features> <feature id>:<feature value> <feature id>:<feature value> ...`

`<simi pairs file>` stores the data pairs labeled as similar. Each line contains a similar data pair separated with tab.
`<diff pairs file>` stores the data pairs labeled as dissimilar. Each line contains a dissimilar data pair separated with tab.


## Output model format
The app outputs the learned model to one file:
```
<dismat_file>
```

`<dismat_file>` saves the distance matrix in row major order and each line corresponds to a row. Elements in each row are separated with blank.

## Terminate DML app

The DML app runs in the background, and outputs its progress to stdout. If you need to terminate the app before it finishes, for distributed version, run

```
./script/kill.py <hostfile>
```


## File IO from HDFS

Put datasets to HDFS
```
hadoop fs -mkdir -p /user/bosen/dataset/dml
hadoop fs -put datasets /user/bosen/dataset/dml
```

Change the corresponding file paths in `script/run_local.py` to the right HDFS path. Comment out the local path.
```
# , "parafile": join(app_dir, "datasets/dml_para.txt")
, "parafile": "hdfs://hdfs-domain/user/bosen/dataset/dml/datasets/dml_para.txt"
# , "feature_file": join(app_dir, "datasets/mnist_petuum/minist_reformatted.txt")
, "feature_file": "hdfs://hdfs-domain/user/bosen/dataset/dml/datasets/mnist_petuum/minist_reformatted.txt"
# , "simi_pairs_file": join(app_dir, "datasets/mnist_petuum/mnist_simi_pairs.txt")
, "simi_pairs_file": "hdfs://hdfs-domain/user/bosen/dataset/dml/datasets/mnist_petuum/mnist_simi_pairs.txt"
# , "diff_pairs_file": join(app_dir, "datasets/mnist_petuum/mnist_diff_pairs.txt")
, "diff_pairs_file": "hdfs://hdfs-domain/user/bosen/dataset/dml/datasets/mnist_petuum/mnist_diff_pairs.txt"
# , "model_weight_file": join(app_dir, "datasets/dismat.txt")
, "model_weight_file": "hdfs://hdfs-domain/user/bosen/dataset/dml/datasets/dismat.txt"
```
Launch it over ssh
```
./script/launch.py
```

Check the output
```
hadoop fs -cat /user/bosen/dataset/dml/datasets/dismat.txt
```

## Use Yarn to launch DML app
```
./script/launch_on_yarn.py
```
