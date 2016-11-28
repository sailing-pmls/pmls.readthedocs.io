# Sparse Coding

Given an input matrix `X`, the Sparse Coding app (implemented on **Bösen**) learns the dictionary `B` and the coefficients matrix `S` such that `S*B` is approximately equal to `X` and the coefficients matrix `S` is sparse.

If `X` is N-by-M, then `B` will be K-by-M and `S` will be N-by-K, where N is the number of data points, M is the dimension of the data, K is a user-supplied parameter that controls the size of the dictionary. The objective of sparse coding app is to minimize the function `||X - S*B||^2 + lambda*||S||_1` subject to the constraint that `||B(i,:)||_2 <= c`, where `lambda` and `c` are hyperparameters and `lambda/c` controls the regulariztion strength.

Our Sparse Coding app uses the projected Stochastic Gradient Descent (SGD) algorithm to learn the dictionary `B` and the coefficients `S`.

## Performance

On a dataset with 1M data instances; 22K feature dimension; 22M parameters, the Bösen Sparse Coding implementation converges in approximately 12.5 hours, using 8 machines (16 cores each).

## Quick Start

The Sparse Coding app can be found in `bosen/app/sparsecoding/`. From this point on, all instructions will assume you are in `bosen/app/sparsecoding/`. After building PMLS (as explained earlier in this manual), you can build the Sparse Coding app from `bosen/app/sparsecoding` by running

```
make
```

This will put the Sparse Coding binary in the subdirectory `bin/`. 

Create directory
```
mkdir sample
mkdir sample/data
mkdir sample/output
```

Create a simulated dataset:
```
./script/make_synth_data.py 5 100 6 sample/data/sample.txt 1
```

Change `app_dir` in `script/run_local.py`; see example below. You need to provide the full path to the Sparse Coding directory. This will allow for correct YARN operation (if required).
```
app_dir = "/home/user/bosen/app/sparsecoding"
```
then you can test that the app works on the local machine (with 4 worker threads). Now, run:
```
./script/launch.py
```
The Sparse Coding app runs in the background (progress is output to stdout). After a few minutes, you should get the following output files in the subdirectory `sample/output/`:
```
B.txt
S.txt.0
loss.txt
time.txt
```
The file `B.txt` and the file `S.txt.0` are the optimization results of dictionary `B` and coefficients `S`, which looks like this:

```
0.187123    0.584781    0.716686    0.259561    -0.20495
0.378984    0.0456092   -0.667287   -0.218985   -0.600886
-0.863578   -0.162164   -0.244522   -0.171278   -0.372572
0.188104    0.558853    0.744098    0.252362    0.186899
-0.71181    -0.165808   -0.562194   0.00220546  -0.386997
0.075108    0.496782    0.0201851   0.692971    0.516672
```

```
0   -0.232664   0   0.273726    -0.123055   0.216856
0.342858    0   0   0.249487    0   0.121574
0.392845    0   -0.214571   0.486131    -0.486699   0
0.346318    0   0   0.258979    0   0.118132
0   -0.221088   0   0.297128    -0.113161   0.209509
0   0   -0.058158   0.0697259   -0.00593826 0.378869
0   -0.252994   0   0.286514    -0.102685   0.203422
0.156121    0   -0.0727321  0.183034    -0.0765836  0.0734904
0.359529    0   0   0.249562    0   0.120825
...
```

The file `loss.txt` contains the statistics of loss function evaluated at iterations specified by user. If there are `num_client` machines in the experiment, it will has `num_client` columns, and the i-th column is the loss function evaluated by the i-th machine. Note that loss function is evaluated by averaging loss function at randomly sampled data points. The file `time.txt` also has `num_client` columns, and contains the time (seconds) taken between evaluations on each client (Evaluating time is excluded). Don't be surprised if the first row of `time.txt` is nearly 0, that's because the loss function is evaluated once before any optimization is conducted. Also note that at the end of `time.txt` and `loss.txt` there might be some `N/A`s, that's invalid and is due to the fact that different clients may evaluate different times.


You won't see the exact same numeric values in different runs as there are multiple optimums of the optimization problem. All you need to confirm is that the optimized `S` shall be sparse and average objective function value of the final result shoud be lower than or close to 0.1.

## Data format

The Sparse Coding app takes a dense matrix `X` as input, where each row corresponds to a data sample and each column corresponds to a feature. The file containing `X` should be a binary or text file, and elements are sorted by rows then columns:

```
ele_0_0     ele_0_1     ... ele_0_n-1
ele_1_0     ele_1_1     ... ele_1_n-1
...
ele_n-1_0   ele_n-1_1   ... ele_n-1_n-1
```

You must specify the format of file (binary or text) by using the parameter `file_format`, which will be explained in [Running the Sparse Coding application](#running-the-sparse-coding-application) section.

## Creating synthetic data

We provide a synthetic data generator for testing the app (requires Python 2.7). To see detailed instructions, run

```
python script/make_synth_data.py
```


## Running the Sparse Coding application

There are many parameters involved for running Sparse Coding. We provide default values for all of them in `scripts/run_local.py`. Some important ones are:

- Input files
    * `data_filename="sample/data/sample.txt"`: The data file name. The data file, which represents input matrix `X`, can be text file or binary file. The format of file needs to be specified in `data_format`. Matrix elements shall be sorted by rows then columns. Matrix elements in text file needs to be separated by blank characters. Matrix elements in binary file needs to be single-precision floating-point format which occupies 4 bytes each. 
    * `is_partitioned=0`: Indicates whether or not the input file has been partitioned. For distributed setting, each machine needs to access their part of data. If `is_partitioned` is set to 0, then each machine needs to get access to the whole data file and Sparse Coding app will do the partitioning automatically. Otherwise, the whole data file needs to be partitioned by column id (See [Data partitioning](#data-partitioning) for details), and each machine will read file named data_filename.client_id, e.g. if data_filename is "sample.txt", then machine 0 will read file "sample.txt.0".
    * `data_format="text"`: Specify the format of input and output data. Can be "text" or "binary". Note that the format of `time.txt` and `loss.txt` does not depend on this parameter and is always text.
    * `load_cache=0`: Specify if whether or not load previous saved results. If `load_cache` is set to 1, the app will load the results in directory `cache_dirname`, which has to contain the matrix `B` and the partial matrix `Si`. The data format of files in `cache_dirname` need to be the same as specified in `data_format`.
    * `cache_dirname="N/A"`: Required if `load_cache` is set to 1. See `load_cache`.
    
- Output files
    * `output_dirname`: Directory where the results will be put into. The `output_dirname` is path relative to `bosen/app/sparsecoding`. Change `output_path` directly if you want to use absolute path.

- Objective function parameters
    * `m`: Dimension of input data. It is also the number of columns (for text format input file) in input data.
    * `n`: Size of input data. It is also the number of rows (for text file) in input data.
    * `dictionary_size`: Size of dictionary.
    * `c`: The l2-norm constraint on dictionary elements.
    * `lambda`: The regularization strength in objective function.

- Optimization parameters
    * `num_epochs=500`: Perform how many epochs during optimization. Each epoch approximately visit all data points once (not exactly because the data points are visited stochastically).
    * `minibatch_size=100`: Size of minibatch.
    * `init_step_size=0.01`: Base init step size. The step size at the t-th iter is in the form of `init_step_size` * (t + `step_size_offset`)^(-`step_size_pow`). As we are alternatively optimizing `B` and `S`, and there might be multiple threads or multiple clients running, the actual step size for `B` and `S` is rescaled by a factor related to number of threads and dimension of data. For most applications, it is enough to only tune this parameter, keeping `step_size_offset` and `step_size_pow` to 0. If you get nan or inf during optimization, then you should decrease `init_step_size`. However, too small a step size will cause slow convergence speed.
    * `step_size_offset=0.0`: See `init_step_size`.
    * `step_size_pow=0.0`: See `init_step_size`.
    * `init_S_low=0.0`: Elements in matrix `S` are initialized from uniform distribution with lower bound `init_S_low` and upper bound `init_S_high`.
    * `init_S_high=0.0`: Elements in matrix `S` are initialized from uniform distribution with lower bound `init_S_low` and upper bound `init_S_high`.
    * `init_B_low=0.0`: Elements in matrix `B` are initialized from uniform distribution with lower bound `init_B_low` and upper bound `init_B_high`.
    * `init_B_high=0.0`: Elements in matrix `B` are initialized from uniform distribution with lower bound `init_B_low` and upper bound `init_B_high`.
    * `num_iter_S_per_minibatch=10`: How many iterations to perform gradient on a randomly picked data point to update the corresponding coefficient. The default value is enough for most applications. A bigger value will result in better optimization results at a given iteration, but at the cost of more time.
    * `init_step_size_B=0.0`: Optional. Valid when it is set to nonzero values. For advanced users, the step size for `B` and `S` can be set directly by setting `init_step_size_B`, `step_size_offset_B`, `step_size_pow_B`, `init_step_size_S`, `step_size_offset_S`, `step_size_pow_S` directly. Note that `init_step_size_B` or `init_step_size_S` must be set to nonzero values if you want to set them directly instead of using step size determined by base step size. For example, if `init_step_size_B` is set to a nonzero value, the step size at the t-th iter will be `init_step_size_B` * (t + `step_size_offset_B`)^(-`step_size_pow_B`). The step size formula for `S` is analogous.
    * `step_size_offset_B=0.0`: Optional. See `init_step_size_B`.
    * `step_size_pow_B=0.0`: Optional. See `init_step_size_B`.
    * `init_step_size_S=0.0`: Optional. See `init_step_size_B`.
    * `step_size_offset_S=0.0`: Optional. See `init_step_size_B`.
    * `step_size_pow_S=0.0`: Optional. See `init_step_size_B`.

- Evaluation parameters
    * `num_eval_minibatch=100`: Evaluate objective function per how many minibatches.
    * `num_eval_samples=$n`: How many samples to pick for each worker thread to evaluate objective function. If the size of data is so large that evaluating objective function takes too much time, you can use either a smaller `num_eval_samples` or a larger `num_eval_minibatch` value.

- System parameters
    * `num_worker_threads=4`: Number of threads running Sparse Coding on each machine.
    * `table_staleness=0`: The staleness for tables.
    * `maximum_running_time=0.0`: The app will try to terminate after running `maximum_running_time` hours. Valid if the value is greater than 0.0.

After all parameters have been chosen appropriately, use the command `./script/run_local.py`, then Sparse Coding application will run in background. The results will be put as specified in `output_dirname`. Matrix `B` will be stored in `B.txt` in client 0 (whose ip appears in the first line in hostfile). Matrix `S` will be stored in a row-partitioned manner, i.e., client i will have a `S.txt.i` in `output_dirname`, and the whole matrix `S` can be obtained by putting all `S.txt.i` together, which will be explained in [Data partitioning](#data-partitioning).

## Terminating the Sparse Coding app

The Sparse Coding app runs in the background, and outputs its progress to log files in user-specified directory. If you need to terminate the app before it finishes, just run

```
./script/kill.py <petuum_ps_hostfile>
```

## Data partitioning

If there are multiple machines in host file, each machine will only take a part of input matrix. Concretely, if there are `num_client` clients, client i will read the j-th row of `X` if j mod `num_client`=i. For example, if the data matrix `X` is:

```
1.0 1.0 1.0 1.0
2.0 2.0 2.0 2.0
3.0 3.0 3.0 3.0
4.0 4.0 4.0 4.0
5.0 5.0 5.0 5.0
```

In the above `X`, the feature dimension is 4, and the size of data is 5. Suppose there are 3 machines in host file, then machine 0 will read the 1-st, 4-th row of `X`, machine 2 will read the 2-nd, 5-th row of `X` and machine 3 will read the 3-rd row of `X`.

Each machine only needs to read part of data, so we provide a parameter `is_partitioned`. In order to use partitioned data (for example, when each client's disk is not enough to hold all data), `is_partitioned` shall be set to 1, and data needs to be partitioned according to the above partitioning strategy. Note that the name of partitioned data on each client needs to be `data_filename`.client_id. We have provided a tool for partitioning data in `scripts/partition_data.py`. Run `python scripts/partition_data.py` for its usage.

When using multiple machines, the result coefficients `S` will be stored in a distributed fashion, corresponding to the part of input data that client reads. For example, In the above example, machine 0 will store `S.txt.0`, which is the 1-st, 4-th row of `X`. We have provided a tool for merging partitioned `S.txt.i` together in `scripts/merge_data.py`. Run `python scripts/merge_data.py` for its usage.


## File IO from HDFS

Put datasets to HDFS
```
hadoop fs -mkdir -p /user/bosen/dataset/sc
hadoop fs -put sample /user/bosen/dataset/sc
```

Change the corresponding file paths in `script/run_local.py` to the right HDFS path. Comment out the local path.
```
# , "output_path": join(app_dir, "sample/output")
, "output_path": "hdfs://hdfs-domain/user/bosen/dataset/sc/sample/output"
# , "data_file": join(app_dir, "sample/data/sample.txt")
, "data_file": "hdfs://hdfs-domain/user/bosen/dataset/sc/sample/data/sample.txt"
```
Launch it over ssh
```
./script/launch.py
```

Check the output
```
hadoop fs -cat /user/bosen/dataset/sc/sample/output/time.txt
```

## Use Yarn to launch Sparse Coding app
```
./script/launch_on_yarn.py
```
