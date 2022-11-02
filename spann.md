# SPANN: Highly-efficient Billion-scale Approximate Nearest Neighbor Search
 - [paper](https://openreview.net/pdf?id=-1rrzmJCp4)
 - [code](https://github.com/microsoft/SPTAG/tree/main/AnnService/inc/Core/SPANN)

## Overview
 - Problem identified: Current ANNS are fast and have high recall, but are extremely expensive at scale (i.e. memory usage, which when large enough must be distributed --> extra problesms)
 - Idea: Efficient memory + disk hybrid indexing. Design indexing and search to minimise no. of disk accesses whilst returning quality postings.


 
 
