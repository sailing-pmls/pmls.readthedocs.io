# PMLS Bösen and Strads Installation

## Foreword and Supported Operating Systems

PMLS Bösen is a communication-efficient distributed key-value store (parameter server) for data-parallel Machine Learning, and PMLS Strads is a dynamic scheduler for model-parallel Machine Learning. Both Bösen and Strads have been officially tested on **64-bit Ubuntu Desktop 14.04** (available at: [http://www.ubuntu.com/download/desktop](http://www.ubuntu.com/download/desktop)). The instructions in this tutorial are meant for Ubuntu 14.04.

We have also successfully tested PMLS on some versions of RedHat and CentOS. However, the commands for installing dependencies in this manual are specific to 64-bit Ubuntu Desktop 14.04. They do not apply to RedHat/CentOS; you will need to know the corresponding packages in `yum`.

**Note:** Server versions of Ubuntu may require additional packages above those listed here, depending on your configuration.

## Obtaining PMLS

The best way to download PMLS is via the `git` command. Install `git` by running

```
sudo apt-get -y update
sudo apt-get -y install git
```

Then, run the following commands to download PMLS Bösen and Strads:

```
git clone -b stable https://github.com/sailing-pmls/bosen.git
git clone https://github.com/sailing-pmls/strads.git
cd bosen
git clone https://github.com/sailing-pmls/third_party.git third_party
cd ..
```

Next, **for each machine that PMLS will be running on**, execute the following commands to install dependencies:

```
sudo apt-get -y update
sudo apt-get -y install g++ make autoconf git libtool uuid-dev openssh-server cmake libopenmpi-dev openmpi-bin libssl-dev libnuma-dev python-dev python-numpy python-scipy python-yaml protobuf-compiler subversion libxml2-dev libxslt-dev zlibc zlib1g zlib1g-dev libbz2-1.0 libbz2-dev
```

**Warning:** Some parts of PMLS require openmpi, but are incompatible with mpich2 (e.g. in the Anaconda scientific toolkit for Python). If you have both openmpi and mpich2 installed, make sure `mpirun` points to openmpi's executable.

## Compiling PMLS

You’re now ready to compile PMLS. From the directory in which you started, run

```
cd strads
make
cd ../bosen/third_party
make
cd ../../bosen
cp defns.mk.template defns.mk
make
cd ..
```

If you are installing PMLS to a shared filesystem, **the above steps only need to be done from one machine**.

The first make builds Strads, and the second and third makes build Bösen and its dependencies. All commands will take between 5-30 minutes each, depending on your machine. We’ll explain how to compile and run PMLS’s built-in apps later in this manual.

## Compiling PMLS Bösen with cmake

Run the following commands to download PMLS Bösen.
```
git clone https://github.com/sailing-pmls/bosen.git
```

**For each machine that PMLS will be running on**, execute the following commands to install dependencies and libraries.
```
sudo apt-get -y install libgoogle-glog-dev libzmq3-dev libyaml-cpp-dev \
  libgoogle-perftools-dev libsnappy-dev libsparsehash-dev libgflags-dev \
  libboost-thread1.55-dev libboost-system1.55-dev libleveldb-dev \
  libconfig++-dev libeigen3-dev libevent-pthreads-2.0-5
```
You’re now ready to compile PMLS. Run
```
cd bosen
mkdir build
cd build && cmake .. && make -j
```
If you are installing PMLS to a shared filesystem, **the above steps only need to be done from one machine**.
The process takes about 5 minutes.


## Very important: Setting up password-less SSH authentication

PMLS uses `ssh` (and `mpirun`, which invokes `ssh`) to coordinate tasks on different machines, **even if you are only using a single machine**. This requires password-less key-based authentication on all machines you are going to use (PMLS will fail if a password prompt appears).

If you don't already have an SSH key, generate one via

```
ssh-keygen
```

You'll then need to add your public key to each machine, by appending your public key file `~/.ssh/id_rsa.pub` to `~/.ssh/authorized_keys` on each machine. If your home directory is on a shared filesystem visible to all machines, then simply run

```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

If the machines do not have a shared filesystem, you need to upload your public key to each machine, and the append it as described above.

**Note:** Password-less authentication can fail if `~/.ssh/authorized_keys` does not have the correct permissions. To fix this, run `chmod 600 ~/.ssh/authorized_keys`.

## Shared directories

**We highly recommend using PMLS in an cluster environment with a shared filesystem, such as NFS**. To set up NFS on Ubuntu machines, you may refer to [here](https://help.ubuntu.com/14.04/serverguide/network-file-system.html).

Provided all machines are identically configured and have the necessary packages/libraries from `apt-get`, **you only need to compile PMLS and its applications once, from one machine**. The PMLS ML applications are all designed to work in this environment, as long as the input data and configuration files are also available through the shared filesystem.

If your cluster doesn't have shared directories, some PMLS applications can still work, but we do not officially support this. You'll need to take the following extra steps:

1. **Ensure all machines are identically configured:** they must be running the same Linux distro (with the same machine architecture, e.g. x86_64), and the packages described earlier must be installed on every machine. The machines need not have exactly identical hardware, but the Linux software environment must be the same. **PMLS will fail if you have different versions of `gcc` or the needed software packages across different machines.**
2. You need to copy or `git clone` PMLS onto every machine, **at exactly the same path** (e.g. /home/username/pmls). **You must compile PMLS and the PMLS apps separately on each machine.**
3. When running PMLS apps, the input data and configuration files must be present on every machine, **at exactly the same path.**

## Network ports to open
If you have a firewall, you must open these ports on all machines:
* SSH port: 22
* Bösen apps: port range 9999-10998 (you can change these)
* Strads apps: port ranges 47000-47999 and 38000-38999

## Cloud compute support
PMLS can run in any Linux-based cloud environment that supports SSH; we recommend using 64-bit Ubuntu 14.04. If you wish to run PMLS on Amazon EC2, we recommend using the official 64-bit Ubuntu 14.04 Amazon Machine Images provided by Canonical: [http://cloud-images.ubuntu.com/releases/14.04/release/](http://cloud-images.ubuntu.com/releases/14.04/release/).

If you're using Red Hat Enterprise Linux or CentOS on Google Compute Engine, you need to turn off the `iptables` firewall (which is on by default), or configure it to allow traffic through ports 9999-10998 (or whatever ports you intend to use). See [https://developers.google.com/compute/docs/troubleshooting#knownissues](https://developers.google.com/compute/docs/troubleshooting#knownissues) for more info.

## Getting started with applications

Now that you have successfully set up PMLS on one or more machines, you can try out some applications. We recommend getting started with:
* [Bosen: Non-negative Matrix Factorization](nonneg-matrix-fact.md)
* [Strads: Latent Dirichlet Allocation](latent-dirichlet-allocation.md)
