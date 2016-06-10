# Frequently asked questions

**Q: What does "Petuum" mean?**

A: Petuum comes from "perpetuum mobile," which is a musical style characterized by a continuous steady stream of notes - as if in perpetual motion. [Paganini's Moto Perpetuo](https://www.youtube.com/watch?feature=player_detailpage&v=dPRWshWq9E4#t=3) is an excellent example. In turn, Bösen and Strads embody the well-known Bösendorfer piano and Stradivarius violin - fine instruments, played by Liszt and Paganini respectively. The goal of the Petuum project is to build an ML platform that runs efficiently and reliably, like its namesake.

**Q: How does Petuum differ from Spark and Hadoop?**

A: Petuum uses bounded-asynchronous communication and fine-grained update scheduling to speed up ML algorithms, over bulk-synchronous execution (used in Spark and Hadoop).

These techniques take advantage of ML-specific properties such as error tolerance and dependency structures, in order to speed up ML algorithms while still maintaining correct execution guarantees.

Petuum is specifically designed for algorithms used in ML, such as optimization algorithms and sampling algorithms. It is not intended as a general platform for all types of Big Data algorithms.

**Q: How does Petuum differ from other Key-Value stores or Parameter Servers?**

A: Petuum has two major components: a dynamic scheduler system (Strads) for model-parallel ML algorithms, and a bounded-asynchronous key-value store (Bösen) for data-parallel ML algorithms.

The Bösen key-value store supports tunable ML-specific consistency models that guarantee correct execution even under worst-case conditions. Eventually-consistent key-value stores may not have ML execution guarantees.

**Q: What kind of clusters is Petuum targeted at?**

A: Petuum is designed to improve the speed and maximum problem size of ML algorithms, on moderate cluster or cloud compute sizes (1-100 machines). Petuum can also be run on desktops or laptops.
