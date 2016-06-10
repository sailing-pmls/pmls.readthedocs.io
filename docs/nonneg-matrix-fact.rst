# Non-negative Matrix Factorization (NMF)

After compiling Bösen, we can test that it is working correctly with Non-negative Matrix Factorization, a popular algorithm for recommender systems.

## Introduction to NMF

Given an input matrix `X`, the NMF app on **Bösen** learns two non-negative matrices `L` and `R` such that `L*R` is approximately equal to `X`.

If `X` is N-by-M, then `L` will be N-by-K and `R` will be K-by-M where N is the number of data points, M is the dimension of the data, K is a user-supplied parameter that controls the rank of the factorization. The objective of NMF app is to minimize the function `||X - L*R||^2` subject to the constraint that the elements in matrices are non-negative.

Our NMF app uses the projected Stochastic Gradient Descent (SGD) algorithm to learn the two matrices `L` and `R`.

# Quick Start

The NMF app can be found in `bosen/app/NMF/`. From this point on, all instructions will assume you are in `bosen/app/NMF/`. After building Petuum (as explained earlier in this manual), you can build the NMF app from `bosen/app/NMF` by running

```
make
```

This will put the NMF binary in the subdirectory `bin/`. 

Create directory
```
mkdir sample
mkdir sample/data
mkdir sample/output
```

Create a simulated dataset:
```
./script/make_synth_data.py 3 3 sample/data/sample.txt
```
Change `app_dir` in `script/run_local.py`; see example below. You need to provide the full path to the NMF directory. This will allow for correct YARN operation (if required).
```
app_dir = "/home/user/bosen/app/NMF"
```
then you can test that the app works on the local machine (with 4 worker threads). From the `bosen/app/NMF/` directory, run:
```
./script/launch.py
```
The NMF app runs in the background (progress is output to stdout). After a few minutes, you should get the following output files in the subdirectory `sample/output/`:

```
L.txt.0
R.txt
loss.txt
time.txt
```

The file `L.txt.0` and the file `R.txt` are the optimization results of two matrices `L` and `R`, which looks something like this (note that there exist many valid solutions; the NMF app produces one at random):

```
0.80641	0	0	
0.80641	0	0	
0.80641	0	0	
0	0	1.47865	
0	0	1.47865	
0	0	1.47865	
0	2.2296	0	
0	2.2296	0	
0	2.2296	0	
```

```
1.24006	1.24006	1.24006	0	0	0	0	0	0	
0	0	0	0	0	0	1.34553	1.34553	1.34553	
0	0	0	1.35258	1.35258	1.35258	0	0	0	
```

The file `loss.txt` contains the statistics of loss function evaluated at iterations specified by user. If there are `num_client` machines in the experiment, it will has `num_client` columns, and the i-th column is the loss function evaluated by the i-th machine. Note that loss function is evaluated by averaging loss function at randomly sampled data points. The file `time.txt` also has `num_client` columns, and contains the time (seconds) taken between evaluations on each client (Evaluating time is excluded). Don't be surprised if the first row of `time.txt` is nearly 0, that's because the loss function is evaluated once before any optimization is conducted. Also note that at the end of `time.txt` and `loss.txt` there might be some `N/A`s, that's invalid and is due to the fact that different clients may evaluate different times.

You won't see the exact same numeric values in different runs as there are multiple optimums of the optimization problem. All you need to confirm is that in optimized `L`, lines 1-3, lines 4-6 and line 7-9 shall be similar. Similarly, in optimized `R`, columns 1-3, 4-6, 7-9 shall be similar. 

# Data format

The NMF app takes a dense matrix `X` as input, where each row corresponds to a data sample and each column corresponds to a feature. The file containing `X` should be a binary or text file, and elements are sorted by rows then columns:

```
ele_0_0     ele_0_1     ... ele_0_n-1
ele_1_0     ele_1_1     ... ele_1_n-1
...
ele_n-1_0   ele_n-1_1   ... ele_n-1_n-1
```

You must specify the format of file (binary or text) by using the parameter `file_format`, which will be explained in [Running the NMF application](#running-the-nmf-application) section.

# Creating synthetic data

We provide a synthetic data generator for testing purposes (we have tested it on python 2.7). To see detailed instructions, run

```
python script/make_synth_data.py
```


# Running the NMF application

There are many parameters involved for running NMF. We provide default values for all of them in `script/run_local.py`. Some important ones are:

- Input files
    * `data_filename="sample/data/sample.txt"`: The data file name. The data file, which represents input matrix `X`, can be text file or binary file. The format of file needs to be specified in `data_format`. Matrix elements shall be sorted by rows then columns. Matrix elements in text file needs to be separated by blank characters. Matrix elements in binary file needs to be single-precision floating-point format which occupies 4 bytes each. 
    * `is_partitioned=0`: Indicates whether or not the input file has been partitioned. For distributed setting, each machine needs to access their part of data. If `is_partitioned` is set to 0, then each machine needs to get access to the whole data file and NMF app will do the partitioning automatically. Otherwise, the whole data file needs to be partitioned by column id (See [Data partitioning](#data-partitioning) for details), and each machine will read file named data_filename.client_id, e.g. if data_filename is "sample.txt", then machine 0 will read file "sample.txt.0".
    * `data_format="text"`: Specify the format of input and output data. Can be "text" or "binary". Note that the format of `time.txt` and `loss.txt` does not depend on this parameter and is always text.
    * `load_cache=0`: Specify if whether or not load previous saved results. If `load_cache` is set to 1, the app will load the results in directory `cache_dirname`, which has to contain the matrix `R` and the partial matrix `Li`. The data format of files in `cache_dirname` need to be the same as specified in `data_format`.
    * `cache_dirname="N/A"`: Required if `load_cache` is set to 1. See `load_cache`.

- Output files
    * `output_dirname="sample/output"`: Directory where the results will be put into. The `output_dirname` is path relative to `bosen/app/NMF`. Set `output_path` directly if you want to use absolute path.

- Objective function parameters
    * `m`: Dimension of input data. It is also the number of columns (for text format input file) in input data.
    * `n`: Size of input data. It is also the number of rows (for text file) in input data.
    * `rank`: Rank of matrix factorization.

- Optimization parameters
    * `num_epochs=500`: Perform how many epochs during optimization. Each epoch approximately visit all data points once (not exactly because the data points are visited stochastically).
    * `minibatch_size=100`: Size of minibatch.
    * `init_step_size=0.01`: Base init step size. The step size at the t-th iter is in the form of `init_step_size` * (t + `step_size_offset`)^(-`step_size_pow`). As we are alternatively optimizing `R` and `L`, and there might be multiple threads or multiple clients running, the actual step size for `R` and `L` is rescaled by a factor related to number of threads and dimension of data. For most applications, it is enough to only tune this parameter, keeping `step_size_offset` and `step_size_pow` to 0. If the output contains nan during optimization, then the `init_step_size` shall be decreased. But smaller step size may result in lower convergence speed.
    * `step_size_offset=0.0`: See `init_step_size`.
    * `step_size_pow=0.0`: See `init_step_size`.
    * `init_L_low=0.0`: Elements in matrix `L` are initialized from uniform distribution with lower bound `init_L_low` and upper bound `init_L_high`.
    * `init_L_high=0.0`: Elements in matrix `L` are initialized from uniform distribution with lower bound `init_L_low` and upper bound `init_L_high`.
    * `init_R_low=0.0`: Elements in matrix `R` are initialized from uniform distribution with lower bound `init_R_low` and upper bound `init_R_high`.
    * `init_R_high=0.0`: Elements in matrix `R` are initialized from uniform distribution with lower bound `init_R_low` and upper bound `init_R_high`.
    * `num_iter_L_per_minibatch=10`: How many iterations to perform gradient on a randomly picked data point to update the corresponding row in `L` matrix. The default value is enough for most applications. A bigger value will result in better optimization results at a given iteration, but at the cost of more time.
    * `init_step_size_R=0.0`: Optional. Valid when it is set to nonzero values. For advanced users, the step size for `L` and `R` can be set directly by setting `init_step_size_R`, `step_size_offset_R`, `step_size_pow_R`, `init_step_size_L`, `step_size_offset_L`, `step_size_pow_L` directly. Note that `init_step_size_R` or `init_step_size_L` must be set to nonzero values if you want to set them directly instead of using step size determined by base step size. For example, if `init_step_size_R` is set to a nonzero value, the step size at the t-th iter will be `init_step_size_R` * (t + `step_size_offset_R`)^(-`step_size_pow_R`). The step size formula for `L` is analogous.
    * `step_size_offset_R=0.0`: Optional. See `init_step_size_R`.
    * `step_size_pow_R=0.0`: Optional. See `init_step_size_R`.
    * `init_step_size_L=0.0`: Optional. See `init_step_size_R`.
    * `step_size_offset_L=0.0`: Optional. See `init_step_size_R`.
    * `step_size_pow_L=0.0`: Optional. See `init_step_size_R`.

- Evaluation parameters
    * `num_eval_minibatch=100`: Evaluate objective function per how many minibatches.
    * `num_eval_samples=$n`: How many samples to pick for each worker thread to evaluate objective function. If the size of data is so large that evaluating objective function takes too much time, you can use either a smaller `num_eval_samples` or a larger `num_eval_minibatch` value.

- System parameters
    * `num_worker_threads=4`: Number of threads running NMF on each machine.
    * `table_staleness=0`: The staleness for tables.
    * `maximum_running_time=0.0`: The app will try to terminate after running `maximum_running_time` hours. Valid if the value is greater than 0.0.

After all parameters have been chosen appropriately, use the command `./script/run_local.py`, then NMF application will run in background. The results will be put as specified in `output_dirname`. Matrix `R` will be stored in `B.txt` in client 0 (whose ip appears in the first line in hostfile). Matrix `L` will be stored in a row-partitioned manner, i.e., client i will have a `L.txt.i` in `output_dirname`, and the whole matrix `L` can be obtained by putting all `L.txt.i` together, which will be explained in [Data partitioning](#data-partitioning).

#### Terminating the NMF app

The NMF app runs in the background, and outputs its progress to log files in user-specified directory. If you need to terminate the app before it finishes, just run

```
./script/kill.py <petuum_ps_hostfile>
```

#### Data partitioning

If there are multiple machines in host file, each machine will only take a part of input matrix. Concretely, if there are `num_client` clients, client i will read the j-th row of `X` if j mod `num_client`=i. For example, if the data matrix `X` is:

```
1.0 1.0 1.0 1.0
2.0 2.0 2.0 2.0
3.0 3.0 3.0 3.0
4.0 4.0 4.0 4.0
5.0 5.0 5.0 5.0
```

In the above `X`, the feature dimension is 4, and the size of data is 5. Suppose there are 3 machines in host file, then machine 0 will read the 1-st, 4-th row of `X`, machine 2 will read the 2-nd, 5-th row of `X` and machine 3 will read the 3-rd row of `X`.

Each machine only needs to read part of data, so we provide a parameter `is_partitoned`. In order to use partitioned data (for example, when each client's disk is not enough to hold all data), `is_partitioned` shall be set to 1, and data needs to be partitioned according to the above partitioning strategy. Note that the name of partitioned data on each client needs to be `data_filename`.client_id. We have provided a tool for partitioning data in `script/partition_data.py`. Run `python script/partition_data.py` for its usage.

When using multiple machines, the result matrix `L` will be stored distributedly corresponding to the part of input data that client reads. For example, In the above example, machine 0 will store `L.txt.0`, which is the 1-st, 4-th row of `L`. We have provided a tool for merging partitioned `L.txt.i` together in `script/merge_data.py`. Run `python script/merge_data.py` for its usage.


## File IO from HDFS

Put datasets to HDFS
```
hadoop fs -mkdir -p /user/bosen/dataset/nmf
hadoop fs -put sample /user/bosen/dataset/nmf
```

Change the corresponding file paths in `script/run_local.py` to the right HDFS path. Comment out the local path.
```
# , "output_path": join(app_dir, "sample/output")
, "output_path": "hdfs://hdfs-domain/user/bosen/dataset/nmf/sample/output"
# , "data_file": join(app_dir, "sample/data/sample.txt")
, "data_file": "hdfs://hdfs-domain/user/bosen/dataset/nmf/sample/data/sample.txt"
```
Launch it over ssh
```
./script/launch.py
```

Check the output
```
hadoop fs -cat /user/bosen/dataset/nmf/sample/output/time.txt
```

## Use Yarn to launch NMF app
```
./script/launch_on_yarn.py
```
