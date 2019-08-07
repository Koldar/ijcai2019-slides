# Title page

Hello, I'm Massimo Bono and today I will talk about Path Planning with CPD Heuristics

# Introduction

In the context of single-agent shortest path planning, modern algorithms exploit *offline* preprocessed data structures to improve *runtime* search. This works if the graph representing the map remain unchanged, but when edge costs are dynamic the preprocessed data becomes invalid.

The objective of this talk is to solve the dynamic edge cost single agent path planning problem in a **specific context** using Compressed Path Databases, or CPDs. What we achieved is a new bounded suboptimal A* variant whose experimentally-evaluated performances show significant gains over previous algorithms.

# A simple example (1)

A a simple toy example, consider the following map, where each undirected edge has been labelled with its cost.

# A simple example (2)

When the map do not involve any edge cost changes, the optimal path from s 0 0 to s 2 2 is shown in bold.

# A simple example (3)

However, suppose that in this particular instance some edge costs have increased (due for example to an exogenous event) to a fixed value during the whole planning eposide. We need an algorithm that quickly computes the optimal path over the new, altered, graph. 

# Talk Outline

To achieve this, this talk will be organized as follows:

I will first show the context assumptions of our proposed technique, named CPD search, followed by some necessary background.

I will then expose the undcerlying ideas of CPD search. After that, I will show some experimental results when CPD search is used to compute the optimal path and when it is used as anytime search.

Finally I will draw some conclusions and possible future directions.

# Context

ok, first of all the context we're operating in.

For this work we assume that every path planning episode is independent from all the others and both start and target locations of each query remain fixed.

Furthermore, while we assume that the graph is known a priori, we don't make the same assumption about the number and the positions of the edge costs changes (namely *perturbations*). 

We will assume that:

- **first**, such perturbations will only increase the original edge costs, but never decrease it and
- **second**, such perturbations are detected at the beginning of the path planning episode and then remain fixed throughout the whole episiode.

# Compressed Path Database

Next, our approach deeply exploits Compressed Path Database (or CPD).

A cCPD is a data structure that compactly stores the first edge of an optimal path from any source node to any target node in the given graph.

This data structure can be used to quickly compute the shortest path from any node towards any other. 

As an example, consider the following graph and suppose we're interested in optimally going from node 2 to node 6.

First, We query the CPD asking which "edge" we need to follow from node 2 to optimally reach node 6. The CPD answers us edge "c".

We follow such edge and we end up in node 4. 

Then we query the CPD again and we ask the "edge" we need to follow from node 4 to optimally reach node 6. The CPD answers us edge "f".

We keep performing this pair of operations , namely querying CPD and applying the output edge, until we reach the target. By concatenating all the output edges it is possible to obtain an optimal path from the start node till the target. 
In the example, the obtained path is 2,4,5 and 6.

We call such **optimal** path CPD path, or, more formally:

Given a graph G and its CPD, a source node s and target t, the cpd path from s to t is the path obtained by **recursively** contatenating the edge obtained from CPD of s,t with the cpd path from the sink of such edge and the same target. The cpd path is empty if s and t are the same. 

# Cpd-Search (1)

I will now explain the underlying ideas of our technique, CPD search.

CPD search is A* variant which computes bounded suboptimal solutions. To do so, the CPD paths, which can be retrieved using the **preprocessed CPD**, are exploited in several ways to improve the **runtime** search the graph using perturbated weights.

One important property is that each A* search node $n$ (containing the location inside the map) has **implicitly** associated a cpd path which allows to optimally reach the target node t:

We will use h CPD n to refer to the cost of such path using the original weights while h prime CPD will refer to the cost of the same path but using the perturbated weights.

# Cpd-Search (2)

We firstly use the CPD to derive an admissible heuristic. Since perturbations can only increase the original edge-cost, h CPD of n (which is the cost of an optimal path on the **original** map) never overestimates the cost of the optimal path on the perturbated map, thus can be used as an admissible heuristic.

As an example, consider the given graph where we want to find the optimal path from node 2 to node 6: 
the involved search node is $n$ and its cpd path is the red one. the cost of the optimal solution is 8 (by going through nodes 1, 4, 5 and 6) while h cpd of $n$ is 6 (following red path using the **original** weights);

# Cpd-Search (3)

CPD paths can be used to early terminate the search as well. If the algorithm reaches a search node $n$ whose cpd path do not involve any perturbated edges (thus h prime cpd of n is the same of h cpd of n) then the cpd path is already an optimal path from n to the target node in the perturbated graph as well. Hence we can obtain solution by concatenating the path from node start to n with the cpd path from n to the target.

# Cpd-Search (4)

Lastly, we can exploit CPD to define both lowerbound and upperbound of the optimal solution over the perturbated graph.

In particular given a search node $n$, since perturbations always increase the edge costs, the cost of the path using **perturbated** weights from s to n plus the cost of the cpd path using **original** weights from n to t yields a **lowerbound** of the solution.

Viceversa, the cost of the path using **perturbated** weights from s to n plus the cost of the cpd path using **perturbated** weights from n to t yield an **upperbound** of the solution. 

By mantaining those 2 bounds and the best solution found (obtained by concatenating the path using perturbated weights from $s$ to $n$ with the cpd path from n to t) it is possible to derive a bounded suboptimal algorithm. The bound quality can be set with a user-defined threshold epsilon. When such threshold is set to 1, CPD search yields the optimal path. Furthermore, CPD search can be used in anytime search by outputting the best solution found so far. In this talk we will focus on the case where $\epsilon = 1$

# Experimental Setup

OK, to test CPD Search, we used the benchmarks from Moving AI. 

The perturbations over the maps have been generated in 2 ways:

- One, called AREA, where, given a path planning query, we choose a location on the optimal path and alter all the edges within a radius of 15 from the chosen location. Each edge cost is multiplied by a decaying function ranging from 4 to 1;

- Two, named RANDOM, where we randomly choose 10 percent of edges whose cost is tripled;



We tested CPD search in an optimal and in an anytime scenario: 

in the first we compared the algorithm against ALT, a popular technique using preprocessing which exploits a limited set of nodes named landmarks to derive an admissible heuristic for any node pair. In ALT it's sensitive the number of landmarks to use, since more landmarks produce more accurate estimates at expenses of a greater online time cost and preprocess space.

In the anytime scenario we compared CPD search against AWA: this algorithm is a WA* variant which seeks a suboptimal solution with the tighest possible suboptimality bound. The heuristic used is the landmark heuristic. 

# Optimal Scenario

In the optimal scenario we used the AREA policy to generate perturbations.
The figures here showcases a performance comparison between our approach and ALT using 6, 12 and 18 landmarks to find optimal solutions. The figures contain cactus plots comparing performances in terms of CPU time used to optimally solve the number of instances on the x axis. Note that the instances are, for every algorithm, independently sorted from the easisest one to the hardest one relative to the algorithm involved.

Generally speaking, except for very simple queries, CPD Search works better than ALT, even when the perturbations are generated near the target: in such difficult case, CPD Search needs to expand nodes up until it finds a cpd path with perturbation for early terminating the search; node which is likely to be found only beyond the last perturbation, thus near the target.

Map wise, CPD Search outperforms ALT in maps where the euclidean distance is not very informative (like in mazes, for example in maze512-1-4) shown on the left plot. 

In other maps (for example in hrt201n, right plot) CPD Search still outperforms ALT, albeit the gain here is less significant.

# Anytime scenario

Another experiment we performed was to analyze CPD search as anytime search. Here we used RANDOM policy to generate perturbations. Since we obtained less significant performance gains, we chose to test the algorithm on hrt201n.
We compared our algorithm against AWA using the landmark heuristics with 12 landmarks.

In this scenario we obtained that in 30 microseconds CPD Search had solutions for at least a quarter of instances, while AWA had solutions for almost 0%. 

By 300 microseconds CPD search had already a solution for at least three quarters of instances, while AWA had them for at most half of them;

By 1ms CPD search had a solution for every instances and had computed the optimal path for at least a quarter of them.

Finally in 10ms CPD Search has reached the optimal solution for every instance while AWA has to compute it for at least half of them.

# Conclusion and future works

In conclusion,

we proposed the usage of CPD in the dynamic edge cost settings when edge costs can only increase;
we developed a new bounded suboptimal A* variant, called CPD Search, which can be used in both optimal and anytime context as well;
The experimental analysis we performed show significant performance gains with respect to the state of the art.

In the future we will explore CPD usage not only in the single agent shortest path planning problem, but also in the multi agent path finding context.

Thank you for your attention.