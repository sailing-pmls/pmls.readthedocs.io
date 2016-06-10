# Matrix Factorization

Given an input matrix `A` (with some missing entries), MF learns two matrices `W` and `H` such that `W*H` approximately equals `A` (except where elements of `A` are missing). If `A` is N-by-M, then `W` will be N-by-K and `H` will be K-by-M. Here, K is a user-supplied parameter (the “rank”) that controls the accuracy of the factorization. Higher values of K usually yield a more accurate factorization, but require more computation.

MF is commonly used to perform Collaborative Filtering, where `A` represents the known relationships between two categories of things - for example, `A(i,j) = v` might mean that “person i gave product j rating v”. If some relationships `A(i,j)` are missing, we can use the learnt matrices `W` and `H` to predict them:

`A(i,j) = W(i,1)*H(1,i) + W(i,2)*H(2,i) + ... + W(i,K)*H(K,i)`

The Petuum MF app uses a model-parallel coordinate descent scheme, implemented on the Strads scheduler. If you would like to use the older Bösen-based Petuum MF app, you may obtain it from the [[Petuum v0.93 release|https://github.com/petuum/public/tree/release_0.93]].

### Performance 

The Strads MF app finishes training on the Netflix dataset (480k by 20k matrix) with rank=40 in 2 minutes, using 25 machines (16 cores each).

# Quick start

Petuum MF uses the Strads scheduler, and can be found in `src/strads/apps/cdmf_release/`. **From this point on, all instructions will assume you are in `src/strads/apps/cdmf_release/`.** After building the main Petuum libraries (as explained under Installation), you may build the MF app from `src/strads/apps/cdmf_release/` by running

```
make
```

Test the app (on your local machine) by running

```
./run.py
```

This will perform a rank K=40 decomposition on a synthetic 10k-by-10k matrix, and output the factors `W` and `H` to `tmplog/wfile-mach-*` and `tmplog/hfile-mach-*` respectively.

# Input data format

The MF app uses the [[MatrixMarket format|http://math.nist.gov/MatrixMarket/formats.html]]:

```
%%MatrixMarket matrix coordinate real general
num_rows num_cols num_nonzeros
row col value 
row col value 
row col value 
```

The first line is the MatrixMarket header, and should be copied as-is. The second line gives the number of rows N, columns M, and non-zero entries in the matrix. This is followed by `num_nonzeros` lines, each representing a single matrix entry `A(row,col) = value` (where `row` and `col` are 0-indexed).

# Output format

The MF app outputs `W` and `H` to `tmplog/wfile-mach-*` and `tmplog/hfile-mach-*` respectively. The `W` files have the following format:

```
row-id: value-0 value-1 ... value-(K-1)
row-id: value-0 value-1 ... value-(K-1)
...
```

Each line represents one row in `W`, beginning with the row index `row-id`, and followed by all K values that make up the row.

The `H` files follow a similar format:

```
col-id: value-0 value-1 ... value-(K-1)
col-id: value-0 value-1 ... value-(K-1)
...
```

The number of files `wfile-mach-*` and `hfile-mach-*` depends on the number of worker processes used --- see the next section for more information.

# Program options

The MF app is launched using a python script, e.g. `run.py` used earlier:

```
#!/usr/bin/python
import os
import sys

datafile = ['./sampledata/mftest.mmt ']
threads = [' 16 ']
rank = [' 40 ']
iterations = [' 10 ']
lambda_param = [' 0.05 ']

machfile = ['./singlemach.vm']

prog = ['./bin/lccdmf ']
os.system("mpirun -machinefile "+machfile[0]+" "+prog[0]+" --machfile "+machfile[0]+" -threads "+threads[0]+" -num_rank "+rank[0]+" -num_iter "+iterations[0]+" -lambda "+lambda_param[0]+" -data_file "+datafile[0]+"  -wfile_pre tmplog/wfile -hfile_pre tmplog/hfile");
```

The basic options are:
* `datafile`: Path to the data file, which must be present/visible to all machines. We strongly recommend providing the full path name to the data file.
* `threads`: How many threads to use for each worker.
* `rank`: The desired decomposition rank K.
* `iterations`: How many iterations to run.
* `lambda_param`: Regularization parameter (Petuum MF uses an L2 regularizer)
* `machfile`: Strads machine file; see below for details.

Strads requires a machine file - `singlemach.vm` in the above example. Strads machine files control which machines house Workers, the Scheduler, and the Coordinator (the 3 architectural elements of Strads). In `singlemach.vm`, we spawn all element processes on the local machine `127.0.0.1`, so the file simply looks like this:

```
127.0.0.1
127.0.0.1
127.0.0.1
127.0.0.1
```

To prepare a multi-machine file, please refer to the Strads section under [[Configuration Files for Petuum Apps|Configuration-files-for-Petuum-apps]].
