# Bosen API

## Write your first Petuum Bosen App
After you have successfully completed the compilation process, you can follow this tutorial and start writing you first Bosen Application. The code fragments of this tutorial come from our [demo program](https://github.com/petuum/bosen/tree/master/app/app_demo). You can read it when you finish this tutorial.

### Step 0. Include the Bosen header file and the Bosen App template file.

```cpp
#include <petuum_ps_common/include/petuum_ps.hpp>
#include <petuum_ps_common/include/ps_application.hpp>
```

The Bosen App template file contains an abstract class ```PsApplication```, which mainly takes care of worker thread spawning, registering, and de-registering processes. All you have to do is to create a subclass (a c-plus-plus class of your bosen application) and implement the pure virtual functions within that class.

### Step 1. Implement the initialization function.

In this step, you need to implement 

```cpp
void initialize(petuum::TableGroupConfig &table_group_config)
```

Initialization tasks such as data loading and table specification should be done here. After data are prepared, follow the steps below.

### Step 1.0. Register Row types.

The Bosen allows applications to use diﬀerent row types. In order for the Bosen to create rows of the correct type, the row types have to be registered with the Bosen before they are created. In this example, we register a row type ```petuum::DenseRow<float>``` with ID ```kDenseRowFloatTypeID``` as:

```cpp
petuum::PSTableGroup::RegisterRow<petuum::DenseRow<float> >(kDenseRowFloatTypeID);
```

Then you can refer to the type of a non-sparse row of floating-point numbers as ```kDenseRowFloatTypeID```.

### Step 1.1. Initialize Table Group.

Here you need to set the configurations for the Table Group (i.e. the Parameter Server). Only one table is needed in our example. Refer to [configs.hpp](https://github.com/petuum/bosen/blob/master/src/petuum_ps_common/include/configs.hpp#L57) for the meanings of all table group parameters.

```cpp
table_group_config.num_tables = 1;
table_group_config.host_map.insert(std::make_pair(0, HostInfo(0, "127.0.0.1", "10000")));
petuum::PSTableGroup::Init(table_group_config, false);
```

### Step 1.2. Create Tables.

You can create several tables, each with the same data type (i.e. the row types registered before). ```table_info.row_type```(row type), ```table_info.row_capacity```(row capacity), ```process_cache_capacity```(maximal number of rows) and ```oplog_capacity```(maximal number of rows that can be written to) are four parameters which are required to be set for each table. Refer to [configs.hpp](https://github.com/petuum/bosen/blob/master/src/petuum_ps_common/include/configs.hpp#L152) for more parameters.

```cpp
petuum::ClientTableConfig table_config;

table_config.table_info.row_type = kDenseRowFloatTypeID;
table_config.table_info.row_capacity = feat_dim;
table_config.process_cache_capacity = 1;
table_config.oplog_capacity = 1;

// Here 0 is the table ID, which will be used later to get table.
bool suc = petuum::PSTableGroup::CreateTable(0, table_config);
```

After all tables are created, call

```cpp
petuum::PSTableGroup::CreateTableDone();
```

### Step 2. Implement the worker thread function.

In this step, you need to implement

```cpp
void runWorkerThread(int threadId)
```

This worker function will be executed on each thread when the Bosen is initialized.

### Step 2.0. Gain Table Access.

In order for the worker thread to handle any parameters stored in Bosen, you first need to get access to the corresponding tables:

```cpp
petuum::Table<float> W = petuum::PSTableGroup::GetTableOrDie<float>(kDenseRowFloatTypeID);
```

Then you can read or update parameters via the table interface.

### Step 2.1. Initialize parameters (Optional).

We use temporary variable ```update_batch``` to store parameter updates before we send it into the ```BatchInc``` update interface. Make sure that you only initialize parameters once.

```cpp
if (thread_id == 0) {
  petuum::DenseUpdateBatch<float> update_batch(0, feat_dim);
  for (int i = 0; i < feat_dim; ++i) update_batch[i] = (rand() % 1001 - 500) / 500.0;
  W.DenseBatchInc(0, update_batch);
}
```

Usually we sync after initialization:

```cpp
process_barrier->wait();
```

### Step 2.2. Read & Update parameters.

At the begining of each training epoch, we wish to read the latest parameters from tables, while at the end, we wish to update them. Reading parameters requires you use the ```Get``` interface:

```cpp
petuum::RowAccessor row_acc;
const petuum::DenseRow<float>& r = W.Get<petuum::DenseRow<float> >(0, &row_acc);
for (int i = 0; i < feat_dim; ++i) paras[i] = r[i];
```

Updating them requires you use the ```BatchInc``` interface, as in the previous step.

### Step 2.3. Don't forget the Clock Tick.

At the end of each training epoch, you need to signal Bosen that this epoch is over by calling:

```cpp
petuum::PSTableGroup::Clock();
```

And the rest will be taken care of by Bosen.

### Step 3. Instantiate and Run.

Congratulations! You have now finished all the required steps towards an amazing Petuum Bosen application. To run the application, instantiate an app object and call its ```run(int32_t num_worker_threads)``` function in your main function, as in our [demo main function](https://github.com/petuum/bosen/blob/master/app/app_demo/src/lr_main.cpp):

```cpp
#include "lr_app.hpp" 

int main(int argc, char *argv[]) { 
  LRApp lrapp; 
  lrapp.run(2); 

  return 0; 
} 
```

You can specify the number of worker threads in your application. The default is ```1```.

## Build your Petuum Bosen App

To build the application with Make, you need to do the following:

1. Create file ```defns.mk``` by copying ```defns.mk.template```. You should have done this in the compilation process.

2. Include ```defns.mk``` in your makefile. We need defns.mk because it provides:

  a. compile flags that are needed to compile the Bosen header files (PETUUM_CXXFLAGS);
  
  b. external libraries that the Bosen library depends on (PETUUM_LDFLAGS);
  
  c. As well as paths to search for the external header files (PETUUM_INCFLAGS) and libraries (PETUUM_LDFLAGS). 

3. With the above maros properly defined, you can write a Makefile of your own to compile this application.

We provide a sample [Makefile](https://github.com/petuum/bosen/blob/master/app/app_demo/Makefile), which should work on most of the Bosen Apps. Remember to change the directories when you use it.

## Detailed programming instructions

For instructions on how to program with the Petuum v1.1 Bösen Bounded-Async Key-Value store, please consult the following pdf: [Bosen Reference Manual](_downloads/bosen_refman.pdf).
