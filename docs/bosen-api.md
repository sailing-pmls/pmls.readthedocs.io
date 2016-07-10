# Bosen API

## Build your first Petuum Bosen App
After you have successfully completed the compilation process, you can start building you first Bosen App.

### Step 0. Include the Bosen header file and the Bosen App template file.

```cpp
#include <petuum_ps_common/include/petuum_ps.hpp>
#include <petuum_ps_common/include/ps_application.hpp>
```

The Bosen App template file contains an abstract class ```PsApplication```, which mainly takes care of worker thread spawning, registering, and de-registering processes. All you have to do is to create a subclass (your own application class) and implement the pure virtual functions.

### Step 1. Implement the initialization function.

In this step, you need to implement 

```cpp
void initialize(petuum::TableGroupConfig &table_group_config)
```

Initialization tasks like data loading and table specification should be done here. After data are prepared, follow the steps below.

### Step 1.0. Register Row types.

The Bosen allows applications to use diﬀerent row types. In order for the Bosen to create rows of the correct type, the rows have to be registered with the Bosen before they are created. In this example, we register a row type ```petuum::DenseRow<int>``` with ID ```0``` as:

```cpp
petuum::PSTableGroup::RegisterRow<petuum::DenseRow<int> >(0);
```

### Step 1.1. Initialize Table Group.

Here you need to set the configurations for the Table Group (i.e. the Parameter Server).

```cpp
table_group_config.host_map.insert(std::make_pair(0, HostInfo(0, "127.0.0.1", "10000")));
petuum::PSTableGroup::Init(table_group_config, false);
```

### Step 1.2. Create Tables.

You can create several tables, each with the same data type (i.e. the row types registered before).

```cpp
petuum::ClientTableConfig table_config;

table_config.table_info.row_type = 0;
table_config.table_info.row_capacity = 100;
table_config.process_cache_capacity = 1000;
table_config.oplog_capacity = 1000;

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

Congratulations! You have now finished all the required steps towards an amazing Petuum Bosen application. To run the application, instantiate an app object and call its ```run``` function in your main function.

## Detailed programming instructions

For instructions on how to program with the Petuum v1.1 Bösen Bounded-Async Key-Value store, please consult the following pdf: [Bosen Reference Manual](_downloads/bosen_refman.pdf).
