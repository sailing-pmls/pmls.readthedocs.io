# Petuum YARN+HDFS support

As of version 1.1, Petuum includes support for running apps on YARN and HDFS (Hadoop version 2.4 or later). To enable support for YARN and HDFS, please follow the instructions below.

## Preliminaries

**Before you begin, please ensure the following:**
* **`hadoop` command is working on every machine.**
* **Java 7 or higher is installed.**

First, install Petuum on every machine in the YARN cluster, as described in [Installation](installation.md). You may install Petuum to the local filesystem; NFS is not required for YARN operation.

Second, you will need to install Gradle to build the Petuum Java components required for YARN. Follow the instructions on the [Gradle webpage](http://gradle.org) to install Gradle, and ensure that the command `gradle` (from the `bin` subdirectory) is in your PATH.

**We do not recommend installing Gradle via `apt-get` on Ubuntu 14.04, as the repository version may be out of date.**

Finally, on each machine, open `bosen/defns.mk`, and note the following lines:

```
JAVA_HOME = /usr/lib/jvm/java-7-openjdk-amd64
HADOOP_HOME = /usr/local/hadoop/hadoop-2.6.0
HAS_HDFS = # Leave empty to build without hadoop.
#HAS_HDFS = -DHAS_HADOOP # Uncomment this line to enable hadoop
```

Change these lines to:

```
JAVA_HOME = <path to Java 7 or Java 8 on your machines>
HADOOP_HOME = <path to Hadoop on your machines>
HAS_HDFS = -DHAS_HADOOP # Uncomment this line to enable hadoop
```

## Recompiling Bösen

You must now recompile Bösen and the supported Bösen apps on each machine. Recompile Bösen using

```
cd bosen
make clean
make
```

Now build the YARN support libraries:

```
cd bosen/src/yarn
gradle build
```

Finally, recompile the apps you want to use via

```
cd bosen/app/path_to_app
make clean
make
```

The list of supported apps, as well as instructions on how to run them, can be found below.

## Which applications are supported?

The following Bösen apps have YARN+HDFS support:

  * [General-purpose Deep Neural Network (DNN)](dnn-general.md)
  * [Non-negative Matrix Factorization (NMF)](nonneg-matrix-fact.md)
  * [Sparse Coding](sparse-coding.md)
  * [Distance Metric Learning](distance-metric-learning.md)
  * [K-means Clustering](k-means.md)
  * [Random Forest](random-forest.md)
  * [Multi-class Logistic Regression](multiclass-logistic-regression.md)

Please refer to the respective wiki pages for running instructions. **Note that the YARN launch scripts are different from the regular SSH launch scripts.**

YARN/HDFS support for Strads will be coming in a future update.

## Troubleshooting

### Running out of virtual memory when using YARN

If you are running out of virtual memory when launching Petuum apps via YARN, you may need to edit `$HADOOP_CONF_DIR/yarn-site.xml` in order to increase the maximum amount of virtual memory that can be allocated to containers. Search for the lines

```
<property> 
<name>yarn.nodemanager.vmem-pmem-ratio</name>  
<value>100</value>
</property>
```

and increase `value` to a ratio higher than 100.
