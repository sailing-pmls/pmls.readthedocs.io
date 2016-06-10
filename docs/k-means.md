# K-Means Clustering
K-means is a clustering algorithm, which identifies cluster centers based on Euclidean distances. Our K-Means app on **Bösen** uses the Mini-Batch K-means algorithm [1].

# Quick Start
The app can be found at `bosen/app/kmeans`. From this point on, all instructions will assume you are at `bosen/app/kmeans`. After building the main Petuum libraries, you can build kmeans:

```
make -j2
```

Then

```
# Create run script from template
cp script/launch.py.template script/launch.py

chmod +x script/launch.py
./script/launch.py
```

The last command runs kmeans using the provided sample dataset `dataset/sample.txt` and output the found centers in `output/out.centers` and the cluster assignments in `output/out.assignmentX.txt` where `X` is the worker ID (each worker outputs cluster assignments in its partition).

# Use HDFS

Kmeans supports HDFS read and output. You need to build Bösen with `HAS_HDFS = -DHAS_HADOOP` in `bosen/defns.mk`. See the [[YARN/HDFS page|Running-on-YARN-HDFS]] for detailed instructions.

Rebuild the binary if you rebuilt the library with Hadoop enabled (under `bosen/app/kmeans`):

```
make clean all
```

Let's copy the demo data to HDFS, where `/path/to/data/` should be replaced:

```
hadoop fs -mkdir -p /path/to/data/
hadoop fs -put dataset/sample.txt /path/to/data/

# quickly verify
hadoop fs -ls /path/to/data/
```

Change a few paths in `script/launch.py`, where `<ip>:<port>` points to HDFS (You can find them on the HDFS web UI):

```
"train_file": "hdfs://<ip>:<port>/path/to/data/sample.txt"
"output_file_prefix": "hdfs://<ip>:<port>/path/to/data/out"
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
hadoop fs -ls /path/to/data/out*
```

# Use Yarn

We will launch job through Yarn and read/output to HDFS. Make sure you've built Yarn by running `gradle build` under `bosen/src/yarn` and have HDFS enabled in `bosen/defns.mk` like before.

Remove the outputs from previous runs:

```
hadoop fs -rm /path/to/data/out.*
```

Create run script from template

```
cp script/run_local.py.template script/run_local.py
```

In `scripts/run_local.py`, set `train_file`, `output_file_prefix` as previously described in the Use HDFS section. Also set the `app_dir` to the absolute path, e.g., `/path/to/bosen/app/kmeans`. Then launch it:

```
chmod +x script/launch_on_yarn.py

# script/launch_on_yarn.py will call script/run_local.py
./script/launch_on_yarn.py
```

You can monitor the job progress in Yarn's WebUI. There you can also find the application ID (e.g., `application_1431548686685_0240`). You can then get the stderr/stdout outputs:

```
yarn logs -applicationId application_1431548686685_0240
```

There you should see similar output as before. As before, you can check the results by `hadoop fs -cat /path/to/data/out.centers`.

## Input Format
The input data needs to be in the libsvm format. A data point would look like:

```
+1 1:0.504134425797 2:-0.259344641268 3:1.74953783689 4:2.62475223767 5:-3.12607240823 6:8.9244514355 7:5.69376634865 8:8.41921260536 9:0.0805904064359 10:4.36430063362
``` 

where +1 is ignored and feature index starts at 1. We provide a script to generate synthetic dataset:

```
cd dataset
# Generates 100 10-dim points drawn from 10 centers
python data_generate.py 10 10 100 libsvm
```


2 files are generated. `dataset/synthetic.txt` contains the datapoints and `dataset/centers.txt` contains the centers for these data points.


## Setting up machines
Put the desired machine IP addresses in the Parameter Server machine file. See this page for more information: [[Configuration-Files-for-Petuum-apps]].

## Common Parameters
In `script/launch.py.template' and `script/run_local.py.template`:

* host_filename = **Parameter Server machine file.** Contains a list of the ip addresses and port numbers for machines in the cluster.
* train_file = Name of the training file present under the dataset folder. It is assumed the entire file is present on a shared file system.
* total_num_of_training_samples = Number of data points that need to be considered.
* num_epochs = Number of mini batch Iterations.
* mini_batch_size = Size of the mini batch.
* num_centers = The number of cluster centers.
* dimensionality = The number of dimensions in each vector.
* num_app_threads = The number of application threads to run minibatch iterations in parallel.
* load_clusters_from_disk = true/talse indicating whether to do initialization of centers from external source. If set False, a random initialization is performed.
* cluster_centers_input_location = Location to the file containing the cluster centers. This file is read if the above argument is set to true.
* output_dir = The directory location where the cluster centers information is written after optimization. The assignments for the data points is written in the assignments sub-directory of the same folder.

## Center Initialization
As of now, we are only providing support for a random initialization for centers. But you could also provide any choice of clusters centers as input to the algorithm. This option can be enabled in the script provided for running the application. The data format for the input centers is same as the one specified for the training dataset. 

## Output
The centers obtained by running the application can be found under the `output/` subdirectory. The indices of the centers run from 0 to k-1 , where k is the number of cluster centers. 
The assignment of the nearest cluster center to the training data points can be found in the assignments  subdirectory. The files in this subdirectory contain the line number of the training data point in the training file and its correspoding cluster center index.


## References
[1]: Sculley, D. "Web-scale k-means clustering." Proceedings of the 19th international conference on World wide web. ACM, 2010.




