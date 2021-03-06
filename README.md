Case studies of Pregel algorithms
---

This repository collects examples for Pregel-like graph processing frameworks. 


### B-matching

Given a weighted undirected graph with integer capacity b(v) assigned to each vertex v, the *Maximum B-matching Problem* is to select a subgraph of maximum weight such that the degree of each vertex in the subgraph does not exceed its capacity. A greedy algorithm described in [Social Content Matching in MapReduce (PVLDB'11)](http://www.vldb.org/pvldb/vol4/p460-morales.pdf) provides a 1/2-approximation guarantee for this problem.

The [Okapi](https://github.com/grafos-ml/okapi) library implements a parallel version of the above algorithm in Giraph. The computation works as follows. Each vertex v maintains a capacity c(v), which is b(v) in the beginning. At each superstep, each vertex v i) proposes its c(v) edges with maximum weight to its neighbors, and ii) computes the intersection between its own proposals and the proposals of its neighbors from the last superstep. The size of this intersection is subtracted from the capacity c(v). If c(v) becomes zero, vertex v is removed from the graph.
        
### Assignment Problem

The [assignment problem](http://en.wikipedia.org/wiki/Assignment_problem) is concerned with finding a maximum matching in a weighted bipartite graph. [This project](https://github.com/cloudera/matching) implements Bertsekas' [auction algorithm](https://en.wikipedia.org/wiki/Auction_algorithm) for the assignment problem on top of Giraph.


### Connected Components

The appendix of [Pregel Algorithms for Graph Connectivity Problems with Performance Guarantees (VLDB'14)](http://www.cse.cuhk.edu.hk/pregelplus/papers/ppa.pdf) describes a Pregel implementation of the [Shiloach-Vishkin algorithm](http://www.cs.cmu.edu/~scandal/alg/connectivity.html) that finds connected components in a graph.

### Graph Partitioning

The [Okapi](https://github.com/grafos-ml/okapi) library implements the [Spinner graph partitioning algorithm](http://arxiv.org/abs/1404.3861), which computes an edge-based balanced k-way partitioning of a graph. Spinner splits the vertices across k partitions, trying to maximize _locality_ and _balancing_. The former means that vertices that are connected through an edge tend to be assigned to the same partition. The latter means that a similar number of edges is assigned to each partition. The partitioning produced by Spinner can be used to minimize network communication and maximize load balance for distributed computation frameworks.

Spinner is based on iterative vertex migrations, with decisions taken independently based on per-vertex local information. In the beginning, vertices are assigned randomly to partitions. After initialization, vertices communicate their partition to their neighbors through messages, and they try to migrate to the partition where most of their neighbors occur. This pattern is also known as *label propagation*. Spinner supports a number of interesting features such as incremental computation and ability to adapt a partitioning to changes to the graph (e.g. vertices and edges are added or removed) and to the partitions (e.g. partitions are added or removed). These features allow Spinner to partition massive graphs and keep the partitioning up-to-date with minimal re-computations.

The computation of the Spinner algorithm works as follows. The algorithm first assigns vertices to partitions according to the following heuristics: 
  - If we are computing a new partitioning, assign the vertex to a random partition.
  - If we are adapting the partitioning to changes of graph:<br />
    a) If the vertex was partitioned before, assign it to the previous partition.<br />
    b) If the vertex is new, assign it to a random partition.
  - If we are adapting the graph to changes of partitioning:<br />
    a) If we are going to add partitions, assign the vertex to the new partitions with probability p (see paper).<br />
    b) If we are going to remove partitions, assign vertices belonging to the removed partitions to the other partitions uniformly at random.

After the vertices are initialized, they communicate their labels to their neighbors, and update the partition loads according to their assignments.

1. Each vertex computes the score for each label based on loads and the labels from incoming neighbors. If a new partition has higher score, the vertex will try to migrate to the partition in next superstep. Otherwise, it does nothing.
2. Interested vertices try to migrate according to the ratio of the vertices trying to migrate and the remaining capacity of the target partition. Vertices that succeed in the migration update the partition loads and communicate their labels to their neighbors.

The algorithm keeps alternating the above two steps until it converges. Convergence is reached when the global score of the partitioning does not change for a number of times over a certain threshold.

### K-Core

The [k-core](en.wikipedia.org/wiki/K-core) of a graph is the subgraph in which all vertices have degree of at least k. The [Okapi](https://github.com/grafos-ml/okapi) library implements a k-core finding algorithm described in [Using Pregel-like Large Scale Graph Processing Frameworks for Social Network Analysis (ASONAM '12)](http://dl.acm.org/citation.cfm?id=2457085). The algorithm iteratively removes vertices with degrees less than k, and stops when there is no vertex to remove. At the end of the execution, the remaining graph represents the k-core. It is possible that the result is an empty graph.

### Social Capital Value

[This project](https://github.com/dhruv-shr/GASSocCAp) provides a Giraph implementation of the algorithm presented in [Social Capital: The Power of Influencers in Networks (AAMAS'13)](http://systemg.research.ibm.com/papers/T3.3-social-capital-power-of-influencers.pdf) to compute the social capital values for nodes in social networks. 

### Frequent Pattern Mining
[This project](https://github.com/dhruv-sharma/distributed-fpm) provides a Giraph implementation of the [APriori algorithm](https://en.wikipedia.org/wiki/Apriori_algorithm) for frequent pattern mining. The [Spark MLlib](https://spark.apache.org/docs/latest/mllib-frequent-pattern-mining.html) implements some frequent pattern mining algorithms without using GraphX.

### Facility Location 

[This project](https://github.com/gvrkiran/giraph-facility-location) provides a Giraph implementation of the Facility Location Algorithm described in [Scalable Facility Location for Massive Graphs on Pregel-like Systems (CIKM'15)](http://dl.acm.org/citation.cfm?id=2806508).

### Graph Drawing 

[This project](https://github.com/EngAAlex/GiLA) implements a [distributed force-directed algorithm](https://en.wikipedia.org/wiki/Force-directed_graph_drawing) in Giraph. The algorithm is described in [A Distributed Force-Directed Algorithm on Giraph](https://arxiv.org/pdf/1606.02162v1.pdf).

<!--
### Jaccard

The Jaccard similarity represents how similar two nodes of a graph are, based on their common neighbors. In particular, it is computed as the size of the intersection of the nodes’ neighbor sets over the size of the union of the nodes’ neighbor sets. The similarity value between two nodes is a number in the range [0, 1]. The closer the value is to 1, the more similar the nodes. (The same metric can be easily converted to a distance with values in the range [0, INF].) 

The computation consists of 2 supersteps. During the first superstep, each vertex sends a list of its friends to all its neighbors in the graph. In the second superstep, each vertex computes the Jaccard similarity value for each of its edges. For every message received, it compares the friend list with its own neighborhood and computes common and total number of friends. It then sets the edge value to the Jaccard similarity index. If the configuration option for distance conversion is enabled, then an additional superstep is executed, where the Jaccard index is converted to a distance index, using the function f(x) = (1/x) - 1.

This implementation only computes similarity or distance for existing edges of the input graph and not for every pair of nodes.
-->

### Semi-Clustering

A cluster is formed by vertices that have stronger connections with each other compared to other vertices. A semi-cluster differs from a typical cluster in that it allows a vertex to exist in more than one cluster. The [Okapi](https://github.com/grafos-ml/okapi) library implements a semi-clustering algorithm described in the [Pregel paper](https://www.researchgate.net/profile/James_Dehnert/publication/221257383_Pregel_A_system_for_large-scale_graph_processing/links/00b7d537c615821fa4000000.pdf). 

The logic behind the algorithm lies in the calculation of a score for each semi-cluster and the dismissal of the semi-clusters with the lowest scores. Each vertex maintains a list of semi-clusters to which it belongs. The list is bounded in length and sorted by score. The score of a semi-cluster is computed by `S = (I-f*B)/(V*(V-1)/2)`, where `I` is the sum of weights of all internal edges, `B` is the sum of weights of all boundary edges, `V` is the number of vertices in the semi-cluster, and `f` is a user-specified boundary edge score factor with a value between 0 and 1. In each superstep, every vertex

1. Receives messages which are semi-clusters created in previous supersteps.
2. For each message, it checks whether it is included in the message/clusters. If not, it adds itself to the semi-clusters and calculates the scores for the new clusters. In any moment, a vertex belongs to a bounded number of semi-clusters. Hence, if a new semi-cluster has a higher score than the last semi-cluster in the list, then the latter gets discarded. Note that a vertex cannot add itself to a semi-cluster if the size of the semi-cluster reaches its maximum, which is defined by the user.
3. It sends a message with its updated list of semi-clusters.

For the first superstep, each vertex creates an empty cluster and adds itself to it. Thus the first set of messages sent in the network contain the vertices themselves. The algorithm finishes when the semi-cluster lists don't change or after a maximum number of iterations.

### BTG Computation

[This project](https://github.com/dbs-leipzig/giraph-algorithms) provides a Giraph implementation of the algorithm described in [BIIIG: Enabling Business Intelligence with Integrated Instance Graphs](http://dbs.uni-leipzig.de/de/publication/title/biiig), which extracts Business Transactions Graphs (BTGs) from an Integrated Instance Graph (IIG). An IIG contains nodes belonging to two different classes: master data and transactional data. A BTG is a sub-graph of an IIG which has only master data nodes as boundary nodes and transactional data nodes as inner nodes. In the business domain, a BTG describes a specific case inside a set of business cases involving master data like Employees, Customers and Products and transactional data like SalesOrders, ProductOffers or Purchases. 

The algorithm finds all BTGs inside a given IIG. The idea is to find connected components by communicating the minimum vertex id inside a connected sub-graph and storing it. The minimum id inside a sub-graph is the BTG id. Only transactional data nodes are allowed to send ids, so the master data nodes work as a communication barrier between BTGs. The master data nodes receive messages from transactional data nodes out of BTGs in which they are involved. They store the minimum incoming BTG id by vertex id in a map and when the algorithm terminates, the set of unique values inside this map is the set of BTG ids the master data node is involved in.

### Label Propagation    

[Label Propagation](https://en.wikipedia.org/wiki/Label_Propagation_Algorithm) is used to detect communities inside a graph by propagating vertex labels (communities). Each vertex stores a unique id, a value (its label) and its outgoing edges. The label represents the community that the vertex belongs to. Vertices migrate to the community represented by the majority of labels sent by its neighbors.

[The Giraph implementation](https://github.com/dbs-leipzig/giraph-algorithms) of the algorithm can be sketched as follows. In superstep 0, each vertex propagates its initial value to its neighbors. In the remaining supersteps, each vertex will adopt the value of the majority of their neighbors or the smallest one if there is only one neighbor. If a vertex adopts a new value, it will propagate the new value to its neighbors. The computation will terminate if no new values are assigned or the maximum number of iterations is reached. The implementation adds a stabilization mechanism and avoids oscillating states.

### Adaptive Repartitoning

[This project](https://github.com/dbs-leipzig/giraph-algorithms) implements the Adaptive Repartitioning (ARP) algorithm as described in [Adaptive Partitioning of Large-Scale Dynamic Graphs (ICDCS'14)](http://www.few.vu.nl/~cma330/papers/ICDCS14.pdf) to partition a graph using label propagation. The implementation exploits Giraph aggregators to share global knowledge about the partition load and demand. For each of the k partitions, it uses two aggregators: The first aggregator stores the capacity (CA_i) of partition i, the second stores the demand (DA_i) for that partition (= number of vertices that want to migrate in the next superstep).

###### Phase 0 : Initialization Phase

If the input graph is an unpartitioned graph, the algorithm will at first initialize each vertex with a partition-id i (hash based). This is skipped if the graph is already partitioned. After that, each vertex will notify CA_i that it is currently a member of the partition and propagates the partition-id to its neighbors.

The main computation is divided in two phases:

###### Phase 1 : Demand Phase (odd-number supersteps)

Based on the information about their neighbors, each vertex calculates its desired partition, which is the most frequent one among the neighbors (label propagation). If the desired partition and the actual partition are not equal the vertex will notify the DA of the desired partition that the vertex want to migrate in.

###### Phase 2 : Migration Phase (even-number supersteps)

Based on the information of the first phase, the algorithm will calculate which vertices are allowed to migrate in their desired partition. If a vertex migrates to another partition it notifies the new and old CA.

The computation will terminate if no vertex wants to migrate, the maximum number of iterations (configurable) is reached or each vertex reaches the maximum number of partition switches (configurable). 

### Community Detection

[This project](https://github.com/pramodschavan/BigDataInfrastructure-Project) implements Girvan-Newman's [community detection algorithm](https://en.wikipedia.org/wiki/Girvan–Newman_algorithm) in Giraph.

### Graph Isomorphism

[This project](https://github.com/nosrepus/giraph-graph-isomorphism) implements a brute-force graph isomorphism algorithm in Giraph.

### Travelling Salesman Problem

[This project](https://github.com/edaboussi/Giraph) provides a brute-force Giraph algorithm for the [Travelling Salesman Problem](https://en.wikipedia.org/wiki/Travelling_salesman_problem).

### Diffusive Clustering

[This project](https://github.com/galpha/giraph-didic) implements the [Distributed Diffusive Clustering Algorithm](http:// parco.iti.kit.edu/henningm/data/distribClust.pdf) in Giraph.

### Minimum Spanning Tree

[This paper](http://ilpubs.stanford.edu:8090/1077/3/p535-salihoglu.pdf) describes a distributed MST algorithm on top of the GPS framework. However, the algorithm needs mater computation, global aggregators and mutation of graph. It is not easy to implement the algorithm in Spark Pregel.

Materials and references
---

1. Malewicz, Grzegorz, et al. [Pregel: a system for large-scale graph processing.](https://www.researchgate.net/profile/James_Dehnert/publication/221257383_Pregel_A_system_for_large-scale_graph_processing/links/00b7d537c615821fa4000000.pdf) ACM SIGMOD 2010. ([digest](https://gist.github.com/shagunsodhani/af9677bdc79bb34be698))
1. Xin, Reynold S., et al. [GraphX: Unifying data-parallel and graph-parallel analytics.](http://arxiv.org/pdf/1402.2394) arXiv preprint (2014). ([digest](https://gist.github.com/shagunsodhani/c72bc1928aeef40280c9))
1. Gonzalez, Joseph E., et al. [Graphx: Graph processing in a distributed dataflow framework.](https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-gonzalez.pdf) OSDI 2014.
1. Avery Ching. Giraph: Large-scale graph processing infrastructure on hadoop. Proceedings of the Hadoop Summit 2011.
1. Ching, Sergey E., et al. [One Trillion Edges: Graph Processing at Facebook-Scale.](http://www.vldb.org/pvldb/vol8/p1804-ching.pdf) PVLDB 2015.
1. Zhang, Hao, et al. [In-memory big data management and processing: A survey.](http://ieeexplore.ieee.org/iel7/69/7116676/07097722.pdf?arnumber=7097722) IEEE Transactions on Knowledge and Data Engineering 2015.
1. Han, Minyang, et al. [An experimental comparison of pregel-like graph processing systems.](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.637.1252&rep=rep1&type=pdf) PVLDB 2014.
1. Salihoglu, Semih, et al. [Optimizing graph algorithms on pregel-like systems.](http://ilpubs.stanford.edu:8090/1077/3/p535-salihoglu.pdf) PVLDB 2014.
1. McCune, Robert R., et al. [Thinking like a vertex: a survey of vertex-centric frameworks for large-scale distributed graph processing.](http://arxiv.org/pdf/1507.04405) ACM CSUR 2015.
1. Yan, Da, et al. [Pregel algorithms for graph connectivity problems with performance guarantees.](http://www.cse.cuhk.edu.hk/pregelplus/papers/ppa.pdf) PVLDB 2014.
