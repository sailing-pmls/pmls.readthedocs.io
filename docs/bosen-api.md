# Bosen API

## Bosen API Documentation

Click [here](http://pmls-bosen.readthedocs.io/en/latest/) for Bosen API Documentations.

## Write your first PMLS Bosen App
After you have successfully completed the compilation process, you can follow this tutorial and start writing you first Bosen Application. The code fragments of this tutorial come from our [demo program](https://github.com/sailing-pmls/bosen/tree/master/app/app_demo). Our demo implements logistic regression and tests it on the [UCI Breast Cancer dataset](https://archive.ics.uci.edu/ml/datasets/Breast+Cancer+Wisconsin+%28Diagnostic%29). Enter the [app directory](https://github.com/sailing-pmls/bosen/tree/master/app/app_demo) and run the launching script with

```python
python script/launch.py
```

You can obtain a result of roughly 92%-93% test accuracy.

### Step 0. Include the Bosen header file and the Bosen App template file.

```cpp
#include <petuum_ps_common/include/petuum_ps.hpp>
#include <petuum_ps_common/include/ps_app.hpp>
```

The Bosen App template file contains an abstract class ```PsApp```, which mainly takes care of worker thread spawning, registering, and de-registering processes. All you have to do is to create a subclass (a c-plus-plus class of your bosen application) and implement the pure virtual functions within that class.

### Step 1. Implement the initialization function.

In this step, you need to implement 

```cpp
virtual void InitApp() = 0;
```

Initialization tasks such as data loading should be done here. This function will only be executed in the init thread of each process. After data are prepared, follow the steps below.

### Step 2. Create Tables.

Implement the following function, which will return a set of configs of the tables you need.

```cpp
// Return a set of table configuration. See TableConfig struct.
virtual std::vector<TableConfig> ConfigTables() = 0;
```

You can create several tables, each with the same data type. Table configs are specified below:

```cpp
struct TableConfig {
  // Required config for table.
  std::string name;   // name is used to fetch the table
  RowType row_type{kUndefinedRow};
  int64_t num_cols{-1};
  int64_t num_rows{-1};
  int staleness{0};

  // Optional configs
  // Number of rows to cache in process cache. Default -1 means all rows.
  int64_t num_rows_to_cache{-1};
  // Estimated upper bound # of pending oplogs in terms of # of rows. Default
  // -1 means all rows.
  int64_t num_oplog_rows{-1};
  // kDenseRowOpLog or kSparseRowOpLog
  int oplog_type{RowOpLogType::kDenseRowOpLog};
};
```

Data types are listed below:

```cpp
enum RowType {
  kUndefinedRow = 0
  , kDenseFloatRow = 1
  , kDenseIntRow = 2
  , kSparseFloatRow = 3
  , kSparseIntRow = 4
};
```

### Implement the worker thread function.

This worker function will be executed on each thread when the Bosen is initialized.

```cpp
virtual void WorkerThread(int client_id, int thread_id) = 0;
```

### Step 3. Gain Table Access.

In order for the worker thread to handle any parameters stored in Bosen, you first need to get access to the corresponding tables via table name:

```cpp
petuum::Table<float> W = GetTable<float>("w");
```

Then you can read or update parameters via the table interface.

### Step 4. Initialize parameters (Optional).

We use temporary variable ```update_batch``` to store parameter updates before we send it into the ```BatchInc``` update interface. Make sure that you only initialize parameters once.

```cpp
if (thread_id == 0) {
  petuum::DenseUpdateBatch<float> update_batch(0, feat_dim_);
  for (int i = 0; i < feat_dim_; ++i) {
    update_batch[i] = (rand() % 1001 - 500) / 500.0 * 1000.0;
  }
  W.DenseBatchInc(0, update_batch);
}
```

### Step 5. Sync after initialization using process_barrier_ from the parent class.

Usually we sync after initialization:

```cpp
process_barrier_->wait();
```

### Step 6-7. Get/Update parameters.

At the begining of each training epoch, we wish to read the latest parameters from tables, while at the end, we wish to update them. Reading parameters requires you use the ```Get``` interface:

```cpp
petuum::RowAccessor row_acc;
const petuum::DenseRow<float>& r = W.Get<petuum::DenseRow<float>>(
    0, &row_acc);
for (int i = 0; i < feat_dim_; ++i) {
  paras_[i] = r[i];
}
```

Updating them requires you use the ```BatchInc``` interface, as in step 4.

### Step 8. Don't forget the Clock Tick.

At the end of each training epoch, you need to signal Bosen that this epoch is over by calling:

```cpp
petuum::PSTableGroup::Clock();
```

And the rest will be taken care of by Bosen.

### Step 9. Instantiate and Run.

Congratulations! You have now finished all the required steps towards an amazing PMLS Bosen application. To run the application, instantiate an app object and call its ```Run(int32_t num_worker_threads)``` function in your main function, as in our [demo main function](https://github.com/sailing-pmls/bosen/blob/master/app/app_demo/src/lr_main.cpp):

```cpp
lrapp.Run(FLAGS_num_app_threads);
```

You can specify the number of worker threads in your application. The default is ```1```.

## Build your PMLS Bosen App

To build the application with Make, you need to do the following:

1. Create file ```defns.mk``` by copying ```defns.mk.template```. You should have done this in the compilation process.

2. Include ```defns.mk``` in your makefile. We need defns.mk because it provides:

  a. compile flags that are needed to compile the Bosen header files (PETUUM_CXXFLAGS);
  
  b. external libraries that the Bosen library depends on (PETUUM_LDFLAGS);
  
  c. As well as paths to search for the external header files (PETUUM_INCFLAGS) and libraries (PETUUM_LDFLAGS). 

3. With the above maros properly defined, you can write a Makefile of your own to compile this application.

We provide a sample [Makefile](https://github.com/sailing-pmls/bosen/blob/master/app/app_demo/Makefile), which should work on most of the Bosen Apps. Remember to change the directories when you use it.

## Detailed programming instructions

For instructions on how to program with the PMLS v1.1 Bösen Bounded-Async Key-Value store, please consult the following pdf: [Bosen Reference Manual](_downloads/bosen_refman.pdf).

