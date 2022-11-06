# SPANN: Highly-efficient Billion-scale Approximate Nearest Neighbor Search
 - [paper](https://openreview.net/pdf?id=-1rrzmJCp4)
 - [code](https://github.com/microsoft/SPTAG/tree/main/AnnService/inc/Core/SPANN)

## Overview
 - Problem identified: Current ANNS are fast and have high recall, but are extremely expensive at scale (i.e. memory usage, which when large enough must be distributed --> extra problesms)
 - Idea: Efficient memory + disk hybrid indexing. Design indexing and search to minimise no. of disk accesses whilst returning quality postings.

## Current Approaches
 - DiskANN:PQ in-memory vectors. Full, navigable vectors on disk. Finds candidates from in-memory, reranks based on on-disk vectors.
 - HM-ANN: Pivots points in fast memory, NSW in slow memory. 1.5x memory usage than diskANN

  Slow memory is not worth the speed/cost tradeoff --> DiskANN is SOTA at billion scale.

## SPANN outline
Store centroid points of inverted index, posting lists on disk. 
During indexing, hierachical balanced clustering.
During search, query aware prune posting lists, thereby reducing disk access costs.
2x faster search latency than DiskANN.

## Background.
Two approaches to billion scale ANN:
1. Inverted index
2. Graph based. 

### Inverted Index
- Split \R_m into k voronoi regions via K means. At search, find 1-NN to centroids. THen only perform search in that region.
- PQ to store vectors in memory
- Subspace the vectors and generate code books. \R_m is just cartesian product of subspaces. 
- > 64GB memory for 1 billion 128 dimensional vectors. Suffer from lossy compression > 60% recall@1.
  - Can return more candidates (i.e. recall@n)

- Ex: Multi-LOPQ, GNO-IMI 

### Graph Based
 - DiskANN:PQ in-memory vectors. Full, navigable vectors on disk. Finds candidates from in-memory, reranks based on on-disk vectors.
    - High disk access cost limits graph traversals and returned candiates (prior to reranking)
 - HM-ANN: Pivots points in fast memory, NSW in slow memory. 1.5x memory usage than diskANN

- Ex: DIskANN, HM-ANN
TODO: Go through references in 2. Background and related works to find seminal papers in field.

## SPANN
 - In memory centroids
 - Posting lists on disk
 - Difficulties with IVF and disk
   - Need to manage posting list length, so that they can have minimal no. of disk access. 
   - Small no. of post list retrievals causes boundary problems.
   - No. of postings for recall@1 for % of queries: (visited postings, %queries) -> (6, 81.78), (12), 90.48) , (114, 99.02). Long tail.

 - Above challenges are why IVF approaches have preferred lossy compression, but in memory.
 - Solutions: 
   - Hierachical balanced clustering. iteratively sub cluster, stop when |sub cluster| <= k  (for some k)
     - SPTAG for nearest posting list in memory. Space partition tree and relative neighbourhood graph as vector index (TODO: look into [SPTAG](https://github.com/Microsoft/SPTAG))
   - epsilon-equal duplication of boundary vectors. Duplicate iff d(x, C_i) <= (1+ \epsilon) d(x, C_j)
   - Don't fix number of posting lists for query. Dynamically select based on distance. Use postings if  d(x, C_i) <= (1+ \epsilon) d(x, C_min)

 
## Datasets for Experiments
- SIFT1M, SIFT1B 1 million + Billion image vectors (128 dimenstions). 10000 query vectors.
- DEEP1B: Image classification model, 96  dim 
- SPACEV1B: data from production search engine. 100 dims. 29 316 test queries.

### Results
- SPANN performs well on SIFT, but drops below on the other two ~3-5ms latency mark (whatever recall that may be). 
- SPANN has much higher latency than in-memory ANNs (no surprise)
- Hierarchical balanced clustering had improved recall latency curves than other centroid approximations: k means and randomly.
- Query aware dynamic pruning only marginally better latency. Benefit may be in resource usage.

## Distributed
- Natural extension for multi-machine approach. Separate into M partitions, separate K posting lists, K/M per partition.
  - Suffer from hot spotting. Make K >> M, use best-fit bin-packing algorithm over historic query data.
  

  
## To consider reading:
- Liudmila Prokhorenkova and Aleksandr Shekhovtsov. 2020. Graph-based nearest neighbor search: From practice to theory. In Proceedings of the International Conference on Machine Learning (ICML). 7803–7813.
 
 
