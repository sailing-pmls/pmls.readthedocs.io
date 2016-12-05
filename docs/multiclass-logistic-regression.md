# Multi-class Logistic Regression

Multi-class logistic regression, or Multinomial Logistic Regression (MLR) generalizes binary logistic regression to handle settings with multiple classes. It can be applied directly to multiclass classification problem, or used within other models (e.g. the last layer of a deep neural network). Our MLR app is implemented on the Bösen system.

## Performance
Using 8 machines (16 cores each), the MLR application converges in approximately 20 iterations or 6 minutes, on the [MNIST dataset](http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/multiclass.html#mnist8m) (8M samples, 784 feature dimensions, taking 19GB of hard disk space as libsvm format).

## Preliminaries

MLR uses the `ml` module in Bösen. If you followed the installation instructions, this should already be built, but if not, go to `bosen/` and run

```
make ml_lib
```

## Quick Start

PMLS MLR can be found in `bosen/app/mlr`. From this point on, all instructions will assume you are in `bosen/app/mlr`.  After building the main PMLS libraries (as explained earlier in this manual), you can build the MLR app from `bosen/app/mlr` by running

```
make -j2
```

This will put the MLR binaries in the subdirectory `bin`.

We now turn to the run scripts. A template is provided for you:

```
cp script/launch.py.template script/launch.py

# Make it executable
chmod +x script/launch.py

# Launch it through ssh.
./script/launch.py
```

The last command launches 4 threads on local node (single node) using a subset of the [Covertype dataset](https://archive.ics.uci.edu/ml/datasets/Covertype). You should see something like

```
37 370 0.257692 0.618548 520 0.180000 50 6.85976
38 380 0.261538 0.61826 520 0.180000 50 7.03964
39 390 0.257692 0.616067 520 0.180000 50 7.22944
40 400 0.253846 0.61287 520 0.180000 50 7.41488
40 400 0.253846 0.61287 520 0.180000 50 7.43618
I0701 00:35:00.550900  9086 mlr_engine.cpp:298] Final eval: 40 400 train-0-1: 0.253846 train-entropy: 0.61287 num-train-used: 520 test-0-1: 0.180000 num-test-used: 50 time: 7.43618
I0701 00:35:00.551867  9086 mlr_engine.cpp:425] Loss up to 40 (exclusive) is saved to /home/wdai/petuum-yarn/app/mlr/out.loss in 0.000955387
I0701 00:35:00.552652  9086 mlr_sgd_solver.cpp:160] Saved weight to /home/wdai/petuum-yarn/app/mlr/out.weight
I0701 00:35:00.553907  9031 mlr_main.cpp:150] MLR finished and shut down!
```

The numbers will be slightly different as it's executed indeterministically with multi-threads. The evaluations are saved to `output/out.loss` and model weights saved to `output/out.weight`.

## Use HDFS

MLR supports HDFS read and output. You need to build Bösen with `HAS_HDFS = -DHAS_HADOOP` in `bosen/defns.mk`. See the [YARN/HDFS page](yarn-hdfs.md) for detailed instructions.

Rebuild the binary if you rebuilt the library with Hadoop enabled (under `bosen/app/mlr`):

```
make clean all
```

Let's copy the demo data to HDFS, where `/path/to/mlr_data/` should be replaced:

```
hadoop fs -mkdir -p /path/to/mlr_data/
hadoop fs -put datasets/covtype.scale.t* /path/to/mlr_data/

# quickly verify
hadoop fs -ls /path/to/mlr_data/
```

Change a few paths in `script/launch.py`, where `<ip>:<port>` points to HDFS (You can find them on the HDFS web UI):

```
"train_file": "hdfs://<ip>:<port>/path/to/mlr_data/covtype.scale.train.small"
"trest_file": "hdfs://<ip>:<port>/path/to/mlr_data/covtype.scale.test.small"
"output_file_prefix": "hdfs://<ip>:<port>/path/to/mlr_data/out"
```

Also uncomment this line in `script/launch.py`:
```
cmd += "export CLASSPATH=`hadoop classpath --glob`:$CLASSPATH; "
```

to add hadoop path. Older hadoop might not have `hadoop classpath --glob`. You need to make sure the class path is set appropriately.

Then launch it as before:

```
./script/launch.py

# Check the result
hadoop fs -cat /path/to/mlr_data/out.loss
```

## Use Yarn

We will launch job through Yarn and read/output to HDFS. Make sure you've built Yarn by running `gradle build` under `bosen/src/yarn` and have HDFS enabled in `bosen/defns.mk` like before.

Remove the outputs from previous runs:

```
hadoop fs -rm /path/to/mlr_data/out.*
```

Create run script from template

```
cp script/run_local.py.template script/run_local.py
```

In `scripts/run_local.py`, set `train_file`, `test_file`, `output_file_prefix` as previously described in the Use HDFS section. Also set the `app_dir` to the absolute path, e.g., `/path/to/bosen/app/mlr`. Then launch it:

```
chmod +x script/launch_on_yarn.py

# script/launch_on_yarn.py will call script/run_local.py
./script/launch_on_yarn.py
```

You can monitor the job progress in Yarn's WebUI. There you can also find the application ID (e.g., `application_1431548686685_0240`). You can then get the stderr/stdout outputs:

```
yarn logs -applicationId application_1431548686685_0240
```

There you should see similar output as before. As before, you can check the results by `hadoop fs -cat /path/to/mlr_data/out.loss`.

## Data Format

MLR accepts both libsvm format (good for data with sparse features) and dense binary format (for dense features). Here we focus on libsvm. The covtype data set (`datasets/covtype.scale.train.small`) uses libsvm and looks like:

```
2 1:0.341596 2:0.637566 3:3.05741 4:1.32943 5:1.22779 6:0.909315 7:-2.65783 8:0.995448 9:1.89365 10:-0.596055 13:1 47:1 
1 1:0.195157 2:0.771602 3:-0.680908 4:2.20452 5:1.22779 6:2.26125 7:-0.379156 8:1.24836 9:1.05749 10:2.58174 11:1 43:1
```

where the first column is the class label and the rest are `feature_id:feature_value` pairs.  Each data file is associated with a meta data. For example, the covtype training data has meta file `datasets/covtype.scale.train.small.meta` with the following fields:

- num_train_total: Number of training data.
- num_train_this_partition: Number of training data in this partition (different from num_train_total if partitioned)
- feature_dim: Number of features.
- num_labels: Number of classes.
- format: Data format: libsvm or bin.
- feature_one_based: 1 if feature id starts at 1 instead of 0.
- label_one_based: 1 if class label starts at 1 instead of 0.
- snappy_compressed: 1 if the file is compressed by [Snappy](https://code.google.com/p/snappy/) which often leads to 2~4x reduction in size.

Similarly, the covtype test data has meta file `datasets/covtype.scale.test.small.meta` with the following fields:

- num_data: Number of test data
- num_test: Number of data to use in test
- feature_dim: Number of features
- num_labels: Number of classes.
- format: Data format: libsvm or bin.
- feature_one_based: 1 if feature id starts at 1 instead of 0.
- label_one_based: 1 if class label starts at 1 instead of 0.
- snappy_compressed: 1 if the file is compressed by [Snappy](https://code.google.com/p/snappy/) which often leads to 2~4x reduction in size.

## Synthetic Data

While we are on the topic of data, let's look at the synthetic data generator which was built in previous `make`. An example script is provided for generating sparse synthetic data in `scripts/run_gen_data.py`. The parameters in the scripts are explained in the source code's flag definitions `bosen/app/mlr/src/tools/gen_data_sparse.cpp`. In particular, `num_train`, `feature_dim`, `nnz_per_col` will primarily determine the size of your data set. The generation mechanism is well documented in the header comment in the source code if you are interested in how multi-class sparse data is generated. The generator will automatically output both the data file and the associated meta data required by MLR to make it easy for MLR to consume.

You can run it with default parameters:

```
python script/run_gen_data.py
```

and find the data at `datasets/lr2_dim10_s100_nnz10.x1.libsvm.X.0`.

## MLR Details

With the data in place, let's look at the input parameters for MLR in `script/launch.py` (similar set of parameters are also in `script/run_local.py` which is for Yarn.):

- Input Files:
    * `train_file="covtype.scale.train.small"`: Training file. This assumes the existence of two files:    `datasets/covtype.scale.train.small` and the meta file `datasets/covtype.scale.train.small.meta`.
    * `test_file="covtype.scale.test.small"`: Test file, analogous to `train_file`. Optional if `perfor_test=false`.
    * `global_data=true`: `true` to let all client machines read the same file. `false` to use partitioned data set where each machine reads from a partition named `train_file.X`, X being the machine ID (0 to num_machines - 1). 
    * `perform_test=true`: `true` to perform test. `test_file` is ignored / not required if `perform_test=false`.

- Initialization:
    * `use_weight_file=false`: true to continue from a previous run.
    * `weight_file=`: When `use_weight_file=true`, `weight_file` is the file output by previous run, e.g. `output/mlr.covtype.scale.train.small.S0.E40.M1.T4/mlr_out.weight` generated from Quick Start above. `weight_File` is ignored if `use_weight_file=false`.

- Execution Parameteres:
    * `num_epochs=40`: Number of passes over the entire data set.
    * `num_batches_per_epoch=300`: Number of mini-batches in each epoch. We clock the parameter server at the end of each mini-batch.
    * `learning_rate=0.01`: Learning rate for gradient in stochastic gradient descent.
    * `decay_rate=0.95`: We use multiplicative decay, i.e., learning rate at epoch `t` is `learning_rate`*`decay_rate^t`.
    * `num_batches_per_eval=300`: Evaluate (approximately) the training (and test error if `perform_test=true`) every `num_batches_per_eval`. Usually set to `num_batches_per_epoch` to evaluate after each epoch.  
    * `num_train_eval=10000`: Number of training examples used to evaluate the intermediate training error. The final evaluation will use all training example.
    * `num_test_eval=20`: Number of test examples used to evaluate the approximate training error.
    * `lambda=0`: L2 regularization parameter
    * `init_lr` and `lr_decay_rate`: Learning rate is `init_lr*lr_decay_rate^T` where `T` is the epoch number.

- System Parameters:
    * `hostfile="scripts/localserver"`: Machine file. See [Configuration Files for PMLS Apps](configuration.md)
    * `num_app_threads=4`: Number of application worker threads.
    * `staleness=0`: Staleness for the weight table (the main table).
    * `num_comm_channels_per_client=1`: The number of threads running server and back ground communication. Usually 1~2 is good enough.

## Terminating the MLR app

The MLR app runs in the background, and outputs its progress to standard error. If you need to terminate the app before it finishes, just run

```
python scripts/kill_mlr.py <petuum_ps_hostfile>
```
