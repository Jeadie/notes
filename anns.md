# Approximate nearest neighbour search (ANNS)

## Motive
- Finding nearest neighbours of vectors is important in information retrieval. 
- Exact solutions to find K-nearest neighbours are intractable in big data scenarios (time and space complexities don't scale well).
- Study of ANNS is to find approximate solutions with lower complexity
- Complexities and trade-offs that various approaches make/consider:
  - Accuracy
  - Time complexity (latency with respect to scaling)
  - Space complexity
  - Distributed search. If an approach cannot handle being performed across nodes (because of time/space complexity, algorithmic constraints), then it can only be used on problems where one machine is viable. An approach be inappropriate because of ircumstances where the dataset cannot fit on one node, or must be distributed (e.g. for reliability reasons). Further, since in practice data often grows over time in a system, approaches constrained in this way may not be adopted even when the dataset is still small.
  - Batch vs real-time: Updates to indexes and vector metadata have varying costs, per approach. Whether the data is fixed, or will be updated, can effect the approach.


## Landscape / Approaches
In this section, we enumerate many approaches in the field. If they're of interest, they may get their own document.

### Voronoi Cells
- Define centroids (vectors in the dataset, subset or full), then precompute the boundaries of each centroid, whereby, all vectors within the boundaries (for a given centroid), are nearest to that centroid (compared to other centroids only). 
- At search time, consider only those neighbours within the cell.
- As an extension, consider a hyperparamater, `neighbourCells`, which dictates how many of the closests cells to also exhaustively search. As `neighbourCells` --> /inf, this approach becomes an exhaustive search.

### Product Quantization (PQ)
- Approximate distance metrics between vectors by clustering on subvectors (i.e. a subset of dimensions of the original vectors).
  1. Construct subvectors. 
  2. Across each subspace, generate either
     1. Subspace specific centroids (e.g k means clustering)
     2. Encoding scheme, i.e. a function f : \R_{n/m} -> \Z_k. m == number of sub vectors/subspaces. k == number of codewords. In this way, 1. can be seen as a specific encoding scheme where each clustered centroid is a code word of the encoding scheme. 
    3. Over each subspace, precompute distance matrix between each codeword/centroid (will be dimension (n/m) x (n/m)).

 - To search: 
   1. Split across subspaces
   2. For each subvector, encode appropriately (i.e. as above)
   3. Do exact KNN across PQ encoded vectors. 
      - Each distance computation is just (n/m) look up operations in the distance table.

 - Can get an approximate reconstruction of original vector from PQ code. 

### Locality Sensitive Hashing (LHS)
- Avoid exhausitive similarity search across dataset by considering candidate pairs, i.e. potential similar values. 
- Use precomputation and efficient lookup to reduce the search space prior to similarity calcuations.
- For example, use several hashing functions (that attempt to maximise hash collisions, not minimise). For a search candidate, compute hashes, and retrieve candidates that match at least one (or could be k of n) hash value (i.e. k of n hashing functions put it in the same bucket).
- Hash functions that preserve the total ordering of the domain's metric space; i.e.
```math
  \forall x,y,z \in M, d(x,y) < d(y,z) \to \|f(x) - f(y)\| < \|f(y) - f(z)\|
```

#### LSH: Dense subvector hashing from a substring vocab
 1. Produce an ordered list of overlapping substrings of the original query string, q. Likewise, do the same for all searchable strings in the document, producing a codebook, or vocab. Define an ordering on this vocab.
 2. Convert query term into a sparse vector (of size equal to the vocab set), where v_i =1 iff the substring vocab[i] is in the search query.
 3. Produce a dense hash signature, with k digits.
    1. Produce k random permutations of 1..n (n== size of vocab).
    2. for each of the k permutations, iterately find the value, v from 1..n in the random permutation (i.e. indexOf(v) within the random permutation k).
    3. If sparsevector[indexOf(v)] == 1, then set digit k of signature to v. Otherwise v++, and check the index again. Since the sparse vector has at least 1 non-zero entry, it is guaranteed to stop in 1..n.
 4. Produce d_s subvectors of dense vector (of dim = k). Hash individual subvectors. Candidate vectors, when converted into dense vectors, must have one (or more < d_s)  of d_s matching subvector hashes. 

- The goal is to maintain similarity information through the underlying computation.

#### Random Projections for LSH
1. Reduce high-dimension vectors into low dimension binary vectors
   1. Compute random hyperplane (equivalent to random vector to act as hyperplane's norm)..
   2. Dot product of hyperplane's perpendicular vector and data vector. If n.v > 0, +ve bit.
   3. Dimension of binary vector == no. of hyperplanes used. Good part --> parallelism.
   4. These binary vectors act as buckets.
2. Compute similarity via hamming distance.
   1. Compute binary vector for search query, q.
   2. Find binary vector, b with smallest hamming distance (sum of xnor boolean operation across bits).
   3. Lookup all search results with binary representation, b.

Important theoretical note:
```math
 P(h(u) != h(v)) = \theta(u, v) /\ \pi
```
That is, a random plane divides two vectors linearly proportional to the angle between the two vectors (see cosine similarity).

Problem is, we can't distinguish within a bucket (i.e. data vectors with matching binary vectors) without extra compute.
Good part, since we are starting with high-dimension vectors (instead of strings, as above), it can be the backend behind embedding models. 
IDEA: Can we produce random hyperplanes that deconstruct the space well. This would be more than just hyperplanes that separate \R_n well as the data vectors won't be evenly distributed. 

### Inverted File Index (IVF)
- Mapping of file content (e.g. word, phrase, sentence) to locations in documents. Often, also a measure of term frequency (within a document) is stored. Could use relevance score.
1. Compute term frequency per document.
2. Store document, relevance per term (on-disk, potentially compressed)

O(n) time and space complexity.

#### Improvements: 
1. Partially invert sub-documents, then merge.
2. Compute term, document, relevance to temp disk. Sort. Read in-order (of term), write to inverted format.


### Hierachical navigable Small world (HNSW)
See [HNSW notes](./hnsw.md)


### Deep Neural Hashing
- Use ML to convert input vectors (any modality, in fact) to compact binary-coded representations.
- Two stage search:
  1. Hamming distance between binary codes
  2. Reranking/subset search within the returned results from 1. (given that hamming distance will/could produce many results of equal distance).

Binary coded represenations (at least by themselves) present a large storage/memory saving.
 - Consider a 768 dimensional vector, float precision == 3072 bytes
 - Using a 768 binary representation, 96 bytes

Also, hamming distance computation is CPU efficient.
 Large scale image search
 