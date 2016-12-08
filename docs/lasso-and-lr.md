# Lasso and Logistic Regression
PMLS provides a linear solver for Lasso and Logistic Regression, using the **Strads** scheduler system. These apps can be found in `strads/apps/linear-solver_release/`. From this point on, all instructions will assume you are in `strads/apps/linear-solver_release/`.

After building the Strads system (as explained in the installation page), you may build the the linear solver from `strads/apps/linear-solver_release/` by running 

`make`

Test the app (on your local machine) by running 

`python lasso.py`  for lasso 

`python logistic.py` for LR 

This will perform Lasso/LR on two separate synthetic data sets in `./input`. The estimated model weights can be found in `./output`.

**Note: on some configurations, MPI may report that the program "exited improperly". This is not an issue as long as it occurs after this line:**
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@ Congratulation ! Finishing task. output file  ./output/coeff.out
```
**If you see this line, the Lasso/LR program has finished successfully.**

## Performance

The logistic regression app on Strads can solve a 10M-dimensional sparse problem (30GB) in 20 minutes, using 8 machines (16 cores each). The Lasso app can solve a 100M-dimensional sparse problem (60GB) in 30 minutes, using 8 machines (16 cores each).

## Input data format

The Lasso/LR apps use the [MatrixMarket format](http://math.nist.gov/MatrixMarket/formats.html):

```
%%MatrixMarket matrix coordinate real general
num_rows num_cols num_nonzeros
row col value 
row col value 
row col value 
```

The first line is the MatrixMarket header, and should be copied as-is. The second line gives the number of rows N, columns M, and non-zero entries in the matrix. This is followed by `num_nonzeros` lines, each representing a single matrix entry `A(row,col) = value` (where `row` and `col` are 1-indexed as like Matlab).

## Output format

The output file of Lasso/LR also follows the MatrixMarket format, and looks something like this:

```
%MatrixMarket matrix coordinate real general
1 1000 108
1 6 -0.02388
1 9 -0.08180
1 40 -0.13945
1 63 -0.07788
...
```

This represents the model weights as a single row vector.

## Machine configuration 
See [Strads configuration files](configuration.md)

## Program Options 
The Lasso/LR is launched using a python script, e.g. lasso.py/logistic.py.

```
machfile = ['./singlemach.vm']

# data setting                                                                                                         
datafilex = ['./input/lasso500by1K.X.mmt']
datafiley = ['./input/lasso500by1K.Y.mmt']
csample = [' 500 ']
column = [' 1000 ']

# scheduler setting                                                                                                    
cscheduler = [' 1 ']
scheduler_threads = [' 1 ']

# worker thread per machine                                                                                            
worker_threads = [' 1 ']

# degree of parallelism                                                                                                
set_size = [' 1 ']

prog = ['./bin/cdsolver ']

os.system(" mpirun -machinefile "+machfile[0]+" "+prog[0]+" --machfile "+machfile[0]+" -threads "+worker_threads[0]+" \
-samples "+csample[0]+" -columns "+column[0]+" -data_xfile "+datafilex[0]+" -data_yfile "+datafiley[0]+"  -schedule_si\
ze "+set_size[0]+" -max_iter 30000 -lambda 0.001 -scheduler "+cscheduler[0]+" -threads_per_scheduler "+scheduler_threa\
ds[0]+" -weight_sampling=false -check_interference=false -algorithm lasso");


```
The basic options are:

* `datafilex`: Path to the design matrix file, which must be present/visible to all machines. We strongly recommend providing the full path name to the data file. 
* `datafiley`: Path to the observation file, which must be present/visible to all machines. We strongly recommend providing the full path name to the data file. 
* `samples`, `columns`: number of rows and columns of the design matrix
* `threads`: number of threads to create per worker machine
* `scheduler`: number of scheduler machines to use
* `threads_per_scheduler`: number of threads to create per scheduler machine
* `max_iter`: maximum number of iterations  
* `algorithm`: flag to specify algorithm to run (lasso/logistic)

The following options are available for advanced users, who wish to control the dynamic scheduling algorithm used in the linear solver:

* `weight_sampling`, `check_interference`: flags to disable/enable parameter-weight-based dynamic scheduling and parameter dependency checking. We strongly recommend to set both of them to the same value (either false or true).
* `schedule_size`: the number of parameters to schedule per iteration. Increasing this can improve performance, but only up to a point.
