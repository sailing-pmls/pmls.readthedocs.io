# Random Forest
A [Random Forest] is a classification algorithm that uses a large number of decision trees. Our Random Forest app is implemented on **Bösen**, using [C4.5] to learn the trees.


# Quick Start

Random Forest can be found in `bosen/app/rand_forest`. From this point on, all instructions will assume you are in `bosen/app/rand_forest`.  After building the main Petuum libraries (as explained earlier in this manual), you can build the Rand Forest app from `bosen/app/rand_forest` by running

```
make -j2
```

This will put the rand_forest binaries in the subdirectory `bin`.

We now turn to the run script. A template is provided for you:

```
cp script/launch.py.template script/launch.py

# Make it executable
chmod +x script/launch.py

# Launch it through ssh.
./script/launch.py
```

The last command launches 4 threads on local node (single node) using [Iris dataset](https://archive.ics.uci.edu/ml/datasets/Iris). You should see something like

```
I0702 03:30:21.282187 18509 rand_forest.cpp:31] Each thread trained 122/125 trees.
I0702 03:30:21.290820 18509 rand_forest.cpp:31] Each thread trained 123/125 trees.
I0702 03:30:21.308017 18509 rand_forest.cpp:31] Each thread trained 124/125 trees.
SaveTrees to hdfs://cogito.local:8020/user/wdai/dataset/rand_forest/out.part0
I0702 03:30:21.315143 18509 rand_forest.cpp:31] Each thread trained 125/125 trees.
SaveTrees to hdfs://cogito.local:8020/user/wdai/dataset/rand_forest/out.part0
SaveTrees to hdfs://cogito.local:8020/user/wdai/dataset/rand_forest/out.part0
SaveTrees to hdfs://cogito.local:8020/user/wdai/dataset/rand_forest/out.part0
I0702 03:30:22.283545 18509 rand_forest_engine.cpp:210] client 0 train error: 0.025 (evaluated on 120 training data)
I0702 03:30:22.283751 18509 rand_forest_engine.cpp:221] client 0 test error: 0.1 (evaluated on 30 test data)
I0702 03:30:22.310386 18509 rand_forest_engine.cpp:316] Test using 500 trees.
I0702 03:30:22.321123 18509 rand_forest_engine.cpp:228] Test error: 0.1 computed on 30 test instances.
I0702 03:30:22.321923 18392 rand_forest_main.cpp:158] Rand Forest finished and shut down!
```

The results are saved to `output/`.

# Use HDFS

Random Forest supports HDFS read and output. You need to build Bösen with `HAS_HDFS = -DHAS_HADOOP` in `bosen/defns.mk`. See the [YARN/HDFS page](yarn-hdfs.md) for detailed instructions.

Rebuild the binary if you rebuilt the library with Hadoop enabled (under `bosen/app/rand_forest`):

```
make clean all
```

Let's copy the demo data to HDFS, where `/path/to/data/` should be replaced:

```
hadoop fs -mkdir -p /path/to/data/
hadoop fs -put dataset/iris.* /path/to/data/

# quickly verify
hadoop fs -ls /path/to/data/
```

Change a few paths in `script/launch.py`, where `<ip>:<port>` points to HDFS (You can find them on the HDFS web UI):

```
"train_file": "hdfs://<ip>:<port>/path/to/data/iris.train"
"test_file": "hdfs://<ip>:<port>/path/to/data/iris.test"
"pred_file": "hdfs://<ip>:<port>/path/to/data/pred"
"output_file": "hdfs://<ip>:<port>/path/to/data/output"
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
hadoop fs -ls /path/to/data/
```

# Use Yarn

We will launch job through Yarn and read/output to HDFS. Make sure you've built Yarn by running `gradle build` under `bosen/src/yarn` and have HDFS enabled in `bosen/defns.mk` like before.

Remove the outputs from previous runs:

```
hadoop fs -rm /path/to/data/out*
hadoop fs -rm /path/to/data/pred
```

Create run script from template

```
cp script/run_local.py.template script/run_local.py
```

In `script/run_local.py`, set `train_file`, `test_file`, `output_file`, `pred_file` as previously described in the Use HDFS section. Also set the `app_dir` to the absolute path, e.g., `/path/to/bosen/app/rand_forest`. Then launch it:

```
chmod +x script/launch_on_yarn.py

# script/launch_on_yarn.py will call script/run_local.py
./script/launch_on_yarn.py
```

You can monitor the job progress in Yarn's WebUI. There you can also find the application ID (e.g., `application_1431548686685_0240`). You can then get the stderr/stdout outputs:

```
yarn logs -applicationId application_1431548686685_0240
```

There you should see similar output as before. As before, you can check the results by `hadoop fs -ls /path/to/data/`.


## Input format
The input data needs to be in the libsvm format or binary format.

## Setting up machines
The ip addresses and ports of all the machines used should be included in the hostfile. You also need to assign client id to each machine in the following format.

    # Client id, IP, Port (do not include this line)
    0 192.168.1.1 10000
    1 192.168.1.2 10000
    2 192.168.1.3 10000
    ...

See this [page](configuration.md) for more details.

## Common parameters
All of the parameters required for running the application can be seen and changed in the sample script `script/launch.py.template`.

Here are several important parameters that should be paid attention to.

- **train_file**: Path of training set file.
- **test_file**: Path of test set file.
- **num_trees**: Overall number of trees in the forest,
- **max_depth**: Maximum depth of each tree (ignore it  or set to 0 if you want it to grow freely).
- **num_data_subsample**: Number of samples used to train each tree (ignore it  or set to 0 if you want it to use all the data).
- **num_feature_subsample**: Number of features used to find best split of each node (ignore it  or set to 0 if you want it to use all the feature).
- **host_filename**: Path of hostfile.
- **num_app_threads**: Number of threads used by each client (machine).
- **perform_test**: Do the test if true.
- **save_pred && pred_file**: If save_pred is true, prediction on test set will be saved in pred_file.
- **save_trees && output_file**: If save_trees is true, trained trees will be saved in output_file.
- **load_trees && input_file**: If load_trees is set as true, training part will be ignored and the app only performs test on trees loaded from input_file.

## Save prediction to file
The app allows users to save prediction on test set into file for future use.

To save prediction, set `save_pred` as true and set `pred_file`. If using provided script `run_rf.sh` or `run_rf_load.sh`, setting `pred_filename` will be enough. 

You can find the output file of prediction results under `output/rf.dataset.Sa.Tb.Mx.Ty/` (*dataset* for the name of dataset, *a* for staleness, *b* for number of epoch, *x* for number of client used and *y* for number of thread in each client). Note that if your machines do not have a shared file system, you can find the file only on client 0 as only thread 0 on client 0 will do the test and save results.

Remember to move important output to somewhere safe since the output directory will be rewrite if rerun the app with parameters above remaining the same.

## Save trained trees to file

The app provides the user with methods to do the training and testing separately.

To save trained model, set `save_trees` as true and set `output_file`.

Note that each client will generate one output file with postfix `.partn` where `n` is the client id. Each file will only contain trees trained by that client. So if the machines have a shared file system, you can find all output files in the output path. But if your machines do not have a shared file system, you have to collect them manually from all your clients and combine them to generate a final output file.

The format of output file is shown as below. Each line is the pre-order deserialization of a trained tree. Every `a:b` token indicates a node in the tree. For non-leaf node, `a` is the fearture used to split this node and `b` is the split threshold. For leaf node, `a` is -1 and `b` is the label.

    # Output file format
    3:0.934434 -1:0 2:4.703931 -1:1 -1:2 
    3:1.609452 2:3.019952 -1:0 -1:1 -1:2 
    2:2.264446 -1:0 3:1.774834 -1:1 -1:2 
    ...


Besides, by setting `perform_test` to false at the same time, the app will do the training only and save the trained trees for further test. Otherwise, the app will still do the test and the trained model is also saved to be reused later.

## Load trained trees from file

To load trained model from file and do test with it, set `load_trees` as true and set `input_file`. If you want to load your trees after saving them with `save_trees` parameter, remember to merge all the output files generated by all the clients into one and let `input_file` point to that file.

Once `load_trees` is set to true, the app will then skip the training part. One thread will perform the test, ignoring parameters such as `max_depth`, `num_data_subsample`, etc. Set `save_pred=true` to store the test predictions.

Note that if your machines do not have a shared file system, make sure that input file at least exists on Client 0.

## Finish up

To kill unfinished process, use

    # Kill the app
    python script/kill.py <hostfile>
    
    
[Random Forest]: http://www.stat.berkeley.edu/~breiman/RandomForests/cc_home.htm
[C4.5]: http://en.wikipedia.org/wiki/C4.5_algorithm
[Iris]: https://archive.ics.uci.edu/ml/datasets/Iris
