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

We call such optimal path CPD path, or, more formally:

Given a graph G and its CPD, a source node s and target t, the cpd path from s to t is the path obtained by **recursively** contatenating the edge obtained from CPD of s and t with the cpd path from the sink of such edge and the same target. The cpd path is empty if s and t are the same. 

# Cpd-Search (1)

I will now explain the underlying ideas of our technique, CPD search.

CPD search is A* variant which computes bounded suboptimal solutions. To do so, the CPD paths are exploited in several way to improve the runtime search on the perturbated map.

One important property is that each A* search node $n$ (holding the location inside the map) has **implicitly** associated a cpd path which allows to optimally reach, form the given node location, the target node t in the map without perturbations:

 We will use h CPD n to refer to the cost of such path 


# Cpd-Search (2)

# Cpd-Search (3)

# Cpd-Search (4)

# Experimental Setup

# Optimal Scenario

# Anytime scenario

# Conclusion and future works

