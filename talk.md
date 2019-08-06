# Title page

Hello, I'm Massimo Bono and today I will talk about Path Planning with CPD Heuristics

# Introduction

In the context of single-agent shortest path planning, modern algorithms exploit *offline* preprocessed data structures to improve performances in *runtime*. This works if the graph representing the map remain unchanged, but when edge costs are dynamic the preprocess is likely to need repair.

The objective of this talk is to solve the dynamic-cost single agent path planning in a **specific context** using Compressed Path Databases, or CPD. What we achieved is a new bounded suboptimal A* variant whose experimentally-evaluated performances show significant gains over previous algorithms.

# A simple example (1)

A a simple toy example, consider the following map, where each undirected edge has been labelled with its cost.

# A simple example (2)

When the map do not involve any edge cost changes, the optimal path from s 0 0 to s 2 2 is shown in bold.

# A simple example (3)

However, suppose that in this particular instance some edge costs have increased (due to an exogenous event) to a fixed value during the whole planning eposide. We need an algorithm that quickly computes the optimal path over the new, altered, map. 

# Talk Outline

To achieve this, this talk will be organized as follows:

I will first show the context assumptions of our proposed technique, followed by some necessary background.

I will then expose the ideas of our proposed technique, CPD search. After that, I will show some experimental results when CPD search is used to compute the optimal path and when is used as anytime search.

Finally I will draw work conclusions and possible future directions.

# Context

ok, first of all the context we're operating in.

For this work we assume that every path planning episodes is independent from another one and both start and target location of each query remain fixed.

Furthermore, while we assume that the graph is known a priori, we don't make the same assumption about the number and the position of the edge costs changes (namely *perturbations*). 

We will assume that:

- first, such perturbations will only increase the original edge costs, but never decrease it and
- second, such perturbations are detected at the beginning of the path planning episodes and then remain fixed throughout the whole episiode.

# Compressed Path Database

Next, our approach deeply exploits Compressed Path Database technique (or CPD).

CPD is a data structure that compactly stores the first edge of an optimal path from any source node to any target node in the given graph.

This data structure can be used to quickly computes the shortest path from any node towards any other. 

As an example, consider the following graph and suppose we're interested in going from node 2 to node 6.

First, We query the CPD asking which is the "edge" we need to follow from node 2 in order to optimally reach node 6. The CPD tell us we need to follow edge "c".

We follow such edge and we reach node 4. 

Then we query the CPD again but this time we ask which is the "edge" we need to follow from node 4 to optimally reach node 6. The CPD tell us we need to follow "f" edge now.

We keep performing this pair of querying CPD and applying the output edge until we reach the goal. By concatenating all the output edges it is possible to obtain an optimal path from the starting node till the target. 
In the example, the obtained path is 2,4,5 and 6.

We call such **optimal** path CPD path, or, more formally:

Given a graph G and its CPD, a source node s and target t, the cpd path from s to t is the path obtained by **recursively** contatenating the edge obtained from CPD of s and t with the cpd path from the sink of such edge and the same target. The cpd path is empty if s and t are the same. 

# Cpd-Search (1)

I will now explain the underlying ideas of our technique, CPD search.

CPD search is A* variant which computes bounded suboptimal solutions. To do so, the CPD paths, which can be retrieved using the **preprocess CPD**, are exploited in several ways to improve the **runtime** search on the perturbated map.

One important property is that each A* search node $n$ (holding the location inside the map) has **implicitly** associated a cpd path which allows to optimally reach the target node t in the map with no perturbations:

We will use h CPD n to refer to the cost of such path using the original weights (namely without perturbations) while h prime CPD will refer to the cost of the same path but using the perturbated weights.

# Cpd-Search (2)

We firstly use the CPD to derive an admissible heuristic. Since perturbations can only increase the original edge-cost, h CPD of n (which is the cost of an optimal path on the **original** map) never overestimates the cost of the optimal path on the perturbated map, thus can be used as an admissible heuristic.

As an example, consider the given graph where we want to find the optimal path from 2 to 6: 
the involved search node is $n$ and its cpd path is the red one. the cost of the optimal solution is 8 (by going through nodes 1, 4, 5 and 6) while h cpd of $n$ is 6 (following red path using the **original** weights);

# Cpd-Search (3)

CPD paths can be used to early terminate the search as well. If the algorithm reaches a search node $n$ whose cpd path do not involve any perturbated edges (thus h prime cpd of n is the same of h cpd of n) then the cpd path is already an optimal path from n to the target node in the perturbated map as well. Hence we can obtain the optimal path on the perturbated map from start node to target node by concatenating the path from node start to n with the cpd path from n to the target.

# Cpd-Search (4)

Lastly, we can exploit CPD to define both lowerbound and upperbound of the optimal solution over the perturbated map.

In particular given a search node $n$, since perturbations always increase the edge costs, the cost of the path on the **perturbated** map from s to n plus the cost of the cpd path using **original** weights from n to t yields a **lowerbound** of the solution.

Furthermore, the cost of the path on the **perturbated** map from s to n plus the cost of the cpd path using **perturbated** weights from n to t yield an **upperbound** of the solution. 

By mantaining those 2 bounds and the best solution found (obtained by concatenating the path over perturbated map from start to n with the cpd path from n to t) it is possible to derive a bounded suboptimal algorithm. The bound quality can be set wth user-defined threshold epsilon. When such threshold is set to 1, CPD search yields the optimal path. Furthermore, CPD search can be used in anytime search by outputting the best solution found so far.

# Experimental Setup

OK, to test CPD Search, we used the benchmarks from Moving AI. 

The perturbations over the maps have been generated in 2 ways:

- One, named RANDOM, where we randomly choose 10 percent of edges whose cost is then multiplied by a factor of 3;

- Two, called AREA, where we choose a location on the optimal path of a query and alter all the edges within a radius of 15 from the chosen location. Each edge cost is multiplied by a decaying function ranging from 4 to 1.

We tested CPD search in an optimal and in an anytime scenario: in the first we compared the algorithm against ALT, a popular technique using preprocessing where exploiting a limited set of nodes named landmarks to derive admissible heuristic for any node pair

# Optimal Scenario

# Anytime scenario

# Conclusion and future works



ridurre abstract ma non eliminare. ridurlo per fare spazio al contesto.
frase che illustra un risultato di effetto in anytime.