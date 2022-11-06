# Hierarchical Navigable Small World
## Overview
HNSW Paper: https://arxiv.org/pdf/1603.09320.pdf
- Vector similarity search

ANN Algorithms, ~3 categories:
 - Trees:
 - Hashes:
 - Graphs: HNSW is a proximity graph; nodes are tensors, edges based on distance metric

--> proximity graph --> HNSW
  1. Probability Skip List
  2. Navigable Small World Graphs

## Probability Skip List
 - Data structure
 - Fast search and insert
 - Complexity:

| First Operation  | Average | Worst |
| ------------- | ------------- | ------------- |
| Insertion | O(log(n))  | O(n)
| Deletion | O(log(n))  | O(n)
| Indexing | O(log(n))  | O(n)
| Search |   O(log(n))  | O(n)

- Finding element by position is O(n)
  - However, if each node stores length to next node, we can get back to O(log(n)
- Space: Average O(n)
 
### Construction
  - Start with a standard, sorted linked list.
  - Create another linked list, but include each element with probability, q. Often first element is always included. q = 1/2 is common


### Important points for analysis
 - element in the most lists, is in log_(1/p)(n) lists, on average
 - Each element contains four, possible null, pointers: left, right, up, down. 
   - left right == along single skip list
   - up down == between skip lists

### Operations
#### Search
 - Maps key to position, where v[p] is the suprememum of the set bound by the key (i.e. v[p] <= key)
 - Scan maximally down, then scan maximally forward. 

#### Index
 - Same as search, return index not value.

#### Insertion
  - Search for key. Since v[p] <= key, we can always insert after this value (if elements are equal, its unimportant).
  - Then add to each skip list with probabilty q
  - If we exceed current height, make new skip list

#### Deletion
  - Search element, delete position in linked list.
  - If empty layer now exists, remove

#### Search complexity
 - Number of skip lists ~ log_(1/p)(n) 

## Navigable Small World (NSW) Graphs
 - Proximity graph with small + large distance links.
 - Greedily find nodes, starting at an entry node, of closer metric distance.
 - Problem is that this method is not good for >> nodes (10^4-5 nodes), increase probability of just a local minima.
 - Could vary the trade-off in the average degree of nodes, with search efficiency.
 - Or start search on higher degree vertices.

## Creating HNSW
 - HNSW is NSW with skip lists
 - Entry layer of skip list contains large edges. Subsequent layers have progressive shorter edges.
 - Links with large edges generally have the largest degree (TODO: why is this true)
 - To search, perform subsequent NSW graph searches. WHen at a local minimum, go to lower layer.

### HNSW - Construction
 - Vector insertion is done similary to skip list. 
 - Vector is inserted into n layers by exponentially decaying distribution. m_l is a layer-normalised. 
   - m_l affects the mutual overlap over neighbours between layers (i.e if vectors are in more layers, % of layers with the same neighbours connected is much higher).
     For performance, we want to minimise overlap --> decrease m_l. But decreasing m_l, makes greedy search on each layer more expensive.
 - First, for each layer, find the ANN (default is ANN, could do A-KNN)  from the entry point (constant across layers), to the query
 - Then, for each layer, 
   - Find M neighbours. M being a parameter, Find neighbours by ANN or other heuristic (i.e. SelectNeighbours).
     - Heuristic suggested is a function of distances. 
     - nearest neighbour selection leads to clustering, cannot cross cluster boundaries.
   - Connect q to each neighbour
   - Ensure degree of neighbours does not exceed maximum allowed. If so must perform SelectNeighbours again for that neighbour.

 - M_{max0} = M leads to performance degradation (??). 
 - M linear in memory consumption
 efConstruction (number of candidate elements during construction of node connection at each layer) 

### Scaling/Performance/Usage
 - High memory usage, stores vectors and relations in memory. Requires memory heavy hardware. If not much throughput (i.e. low traffic, or single tenant), CPU oversupply == expensive. 
