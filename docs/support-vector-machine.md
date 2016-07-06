# Quick start for Support Vector Machine
Petuum provides a SVM solver on distributed system. SVM application can be found in `strads/apps/svm_release/`. From this point on, all instructions will assume you are in `strads/apps/svm_release/`.

After building the Strads system (as explained in the installation page), you may build the the SVM solver from `strads/apps/svm_release/` by running 

`make`

Test the app (on your local machine) by running 

`python svm.py`

This will perform SVM on [rcv1.binary sample data](https://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/binary.html#rcv1.binary) in `./input`. The estimated model weights can be found in `./output`.

## Performance
Coming soon 

## Input data format

The SVM use the [LIBSVM format](https://www.csie.ntu.edu.tw/~cjlin/libsvm/):

```
y col:value  col:value  col:value  col:value  col:value 
y col:value  col:value  col:value  col:value 
y col:value  col:value  col:value
```

A single line represents a sample that consists of y response values and non-zero entries with column indexes. `col` is 1-indexed as like Matlab. 

## Output format

The output file of SVM looks something like this:

```
col value
col value
col value
col value
col value
...
```
Each row with column id and value represents a non-zero model-parameter.

## Machine configuration 
See [Strads configuration files](https://github.com/petuum/bosen/wiki/Configuration-Files-for-Petuum-Apps#strads-configuration-files)

## Program Options 
The SVM is launched using a python script, e.g. svm.py.

```
machfile = ['./singlemach.vm']

# data setting                                                                                                         
input = ['./input/rcv']

# degree of parallelism                                                                                                
set_size = [' 1 ']

prog = ['./bin/svm-dual ']

os.system(" mpirun -machinefile "+machfile[0]+" "+prog[0]+" --machfile "+machfile[0]+" -input "+inputfile[0]+" -max_iter 200 -C 1.0 "+" -parallels "+dparallel[0]+" ");

```
The basic options are:

* `inputfile`: Path to the design matrix file, which must be present/visible to all machines. We strongly recommend providing the full path name to the data file. 

* `max_iter`: maximum number of iterations  
The following options are available for advanced users, who wish to control the dynamic scheduling algorithm used in the linear solver:

* `dparallel`: the number of parameters to schedule per iteration. Increasing this can improve performance, but only up to a point.

* `C`: is a SVM penalty parameter, which should be larger than 0.
