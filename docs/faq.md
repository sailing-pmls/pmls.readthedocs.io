# Frequently asked questions

**Q: How does PMLS differ from Spark and Hadoop?**

A: PMLS uses bounded-asynchronous communication and fine-grained update scheduling to speed up ML algorithms, over bulk-synchronous execution (used in Spark and Hadoop).

These techniques take advantage of ML-specific properties such as error tolerance and dependency structures, in order to speed up ML algorithms while still maintaining correct execution guarantees.

PMLS is specifically designed for algorithms used in ML, such as optimization algorithms and sampling algorithms. It is not intended as a general platform for all types of Big Data algorithms.

**Q: How does PMLS differ from other Key-Value stores or Parameter Servers?**

A: PMLS has two major components: a dynamic scheduler system (Strads) for model-parallel ML algorithms, and a bounded-asynchronous key-value store (Bösen) for data-parallel ML algorithms.

The Bösen key-value store supports tunable ML-specific consistency models that guarantee correct execution even under worst-case conditions. Eventually-consistent key-value stores may not have ML execution guarantees.

**Q: What kind of clusters is PMLS targeted at?**

A: PMLS is designed to improve the speed and maximum problem size of ML algorithms, on moderate cluster or cloud compute sizes (1-100 machines). PMLS can also be run on desktops or laptops.
