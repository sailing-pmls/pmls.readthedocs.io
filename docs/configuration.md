# Configuration

## Make sure password-less SSH is set up correctly

This is the most common issue people have when running PMLS. Please read!

**You must be able to `ssh` into all machines without a password - even if you are only using your local machine.** The PMLS apps will fail in unexpected ways if password-less `ssh` is not working. When this happens, you will not see any error output stating that this is the problem!

**Hence, you will save yourself a lot of trouble by taking the time to ensure password-less `ssh` actually works, before you attempt to run the PMLS apps.** For example, if you are going to run PMLS on your local machine, make sure that `ssh 127.0.0.1` logs you in without asking for a password. See [Installing PMLS](installation.md) for instructions on how to do this.

## Bösen and Strads

PMLS includes two platforms for writing and running ML applications: Bösen for data-parallel execution, and Strads for model-parallel execution. Each PMLS ready-to-run application is either a Bösen application, or a Strads application. The two systems use different machine configuration files; please see the following guides.

**Note: This page explains machine configuration for non-YARN, stand-alone operation. If you are looking to run PMLS on YARN, please see [this page](https://github.com/petuum/bosen/wiki/Running-on-YARN-HDFS).**

## Bösen configuration files

Some PMLS ML applications require Bösen configuration files, in the following format:

```
0 ip_address_0 9999
1 ip_address_1 9999
2 ip_address_2 9999
3 ip_address_3 9999
...
```

**IMPORTANT: There cannot be any trailing blank lines in the configuration file. If there are trailing blank lines, the applications will fail.**

You can give your configuration file any filename. The placeholders `ip_address_0`, `ip_address_1`, etc. are the IP addresses of each machine you want to use. If you only know the hostname of the machine, for example `work-machine-1.mycluster.com`, use `host` to get the actual IP (the hostname will not work):

```
host work-machine-1.mycluster.com
```

Each line in the server configuration file format specifies an ID (0, 1, 1000, 2000, etc.), the IP address of the machine assigned to that ID, and a starting port number (9999). Every machine is assigned to one ID and one starting port number. We say "starting port number" because some applications may use additional consecutive ports, up to a maximum of 100 ports (i.e. the port range 9999-10998).

## I want to run on my local machine

Simply create this localhost configuration file:

```
0 127.0.0.1 9999
```

## My cluster does not have shared directories

PMLS is meant to be used from a shared filesystem, such as NFS - you can find instructions for how to set up NFS on Ubuntu machines [here](https://help.ubuntu.com/14.04/serverguide/network-file-system.html).

If you don't have NFS, you can still run PMLS, but you need to pay attention to the following points:

Make sure that PMLS is compiled on every machine, and the machine configuration files are present on every machine. The paths to PMLS and machine configuration files must be identical across machines.

Different apps have different requirements for input data; if no requirements are explicitly stated, you should make sure the input data is present on every machine.

Some apps store their arguments and parameters in script files. If you modify these script files, you should copy them to every machine, to be safe.

## Caution - Please Read!

**You cannot `crtl-c` to terminate a Bösen app**, because they are invoked in the background via `ssh`. Each Bösen app comes with a kill script for this purpose; please see the individual app sections in this manual for instructions.

If you want to simultaneously run two PMLS apps on the same machines, make sure you give them separate Bösen configuration files with different starting ports, separated by at least 1000 (e.g. 9999 and 10999). **The apps cannot share the same port ranges!**

If you cannot run an app - especially if you see error messages with “Check failure stack trace” - the cause is probably another running (or hung) PMLS app using the same ports. In that case, you should use the offending app's kill script to terminate it.

## Strads configuration files

Some PMLS ML applications require Strads machine configuration files, which are simply a list of machine IP addresses (like an MPI hostfile). Unlike Bösen configuration files, Strads configuration files may repeat IP addresses.

Strads creates three types of processes: Workers, the Scheduler, and the Coordinator.
- The Coordinator is the master process that oversees the Scheduler and Workers, and is spawned on the last IP in the machine file.
- The Scheduler creates and dispatches model variables for the Workers to update. Unless otherwise stated, there is only one Scheduler, spawned on the 2nd last IP in the machine file.
- The Workers perform the actual variable updates to the ML program. They are spawned on the remaining IPs in the machine file. **At least 2 workers are required.**

**You may repeat IP addresses to put multiple processes on the same machine. However, the repeat IPs must be contiguous.**

Example 1: If you want 4 workers, then the machine file looks like this:

```
worker1-ip-address
worker2-ip-address
worker3-ip-address
worker4-ip-address
scheduler-ip-address
coordinator-ip-address
```

Example 2: Let's say you only have 2 machines, and you want to spawn 2 Workers on the 1st machine, and the Scheduler and the Coordinator on the 2nd machine. The machine file looks like this:

```
ip-address-1
ip-address-1
ip-address-2
ip-address-2
```

Because repeat IPs must be contiguous, the following configuration is invalid:

```
ip-address-1   (this configuration WILL NOT WORK)
ip-address-2
ip-address-1
ip-address-2
```

## I want to run on my local machine

If you only have 1 machine, use this Strads machine file with 2 Workers (plus the Scheduler and Coordinator):

```
127.0.0.1
127.0.0.1
127.0.0.1
127.0.0.1
```

## My cluster does not have shared directories

PMLS is meant to be used from a shared filesystem, such as NFS - you can find instructions for how to set up NFS on Ubuntu machines [here](https://help.ubuntu.com/14.04/serverguide/network-file-system.html).

If you don't have NFS, you can still run PMLS, but you need to pay attention to the following points:

Make sure that PMLS is compiled on every machine, and the machine configuration files are present on every machine. The paths to PMLS and machine configuration files must be identical across machines.

Different apps have different requirements for input data; if no requirements are explicitly stated, you should make sure the input data is present on every machine.

Some apps store their arguments and parameters in script files. If you modify these script files, you should copy them to every machine, to be safe.
