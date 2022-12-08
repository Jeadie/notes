# Opensearch KNN plugin
Build an opensearch-native fields specifically for KNN/ANN fields. 
 - i.e. in a `GET index-name/_mappings` there will be a field of `"type" == "knn_vector"` (as well as specified field).

Three ANN implementations:
 - FAISS: 
    - Fastest with GPU acceleration
 - NMSLIB
    - Fastest for CPU only.
 - Lucene
    - Optimal for small dataset (1-10 million vectors)
    - Is only Pure java implementation

Two algorithms:
 - HNSW:
 - IVF: 

### Memory Estimations
- `HNSW = 1.1 * (4 * dimension + 8 * M) ` bytes/vector
- `IVF = 1.1 * (((4 * dimension) * num_vectors) + (4 * nlist * d)) ` bytes/vector

GB for 1 Billion HNSW vectors 
MB for 1 Million HNSW vectors

|  Dim / M | 4 | 8    | 12   | 16   | 20   | 24     | 
| ---   | ---  | --   | ---  | ---  | ---  | --     |
|  128  | 598  | 633  | 668  | 704  | 739  | 774    |
|  256  | 1161 | 1196 | 1232 | 1267 | 1302 | 1337   |
|  512  | 2288 | 2323 | 2358 | 2393 | 2428 | 2464.0 |
|  768  | 3414 | 3449 | 3484 | 3520 | 3555 | 3590.4 | 
|  1024 | 4540 | 4576 | 4611 | 4646 | 4681 | 4716.8 |
|  2048 | 9046 | 9081 | 9116 | 9152 | 9187 | 9222.4 |


## Filter Search
- Filter Search: Performming KNN/ANN with filter criteria on other fields.
- Only supported via Lucene + HNSW.

How/when is filtering applied:
![](https://opensearch.org/docs/latest/images/hsnw-algorithm.png)
- N=documents in index
- P=documents in set after filters applied, search set 
- q=query vector
- k= number of neighbours to return

