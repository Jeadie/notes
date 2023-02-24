# Neural Machine Translation By Jointly Learning to Align and Translate
https://arxiv.org/pdf/1409.0473.pdf

## Overview
The paper introduces a neural machine translation (NMT) model that jointly learns to align and translate. The model consists of an encoder, a decoder, and an attention mechanism that aligns source words with target words. The encoder maps the source sentence to a continuous representation, and the decoder generates the target sentence based on this representation and the attention weights. The attention mechanism allows the model to dynamically focus on different parts of the source sentence when generating each target word. The authors evaluate their model on a variety of language pairs and show that it outperforms previous NMT models. The paper suggests that the joint learning of alignment and translation is an effective way to improve NMT performance.

- Instead of encode -> fixed embedding -> decoding, at each point of target decoding, allow network to soft-search across whole input source. 
- Framework:
    - Input vector sequence $x = ( x_1, x_2, ...x_{T_x})$
    - Internal state, h at time t, updated at each time $h_t = f(x_t, h_{t-1})$
    - Non-linear function from h to target decoding, $c = q(\{ h_1, h_2, ... \})$
    - Probability output for target, y_t: `p(y_t | \{ y_1, y_2, ..., y_{t-1} \}, c)`, produced by arbitrary non-linear function (perhaps multi-layer ANN), `g(t_{t-1}, s_t, c)`
    - Translation $\bf{y} = ( y_1, y_2, ...y_{T_y})$ is decomposing joint probability $P(\bf{y}) = \Pi_{t=1}^T p(y_t | \{ y_1, y_2, ..., y_{t-1} \}, c)$

## Neural Machine Translation Architecture
 - Encoding: Compute hidden state bidirectionally, produce internal state $h_i$ in both forward and backwards. Concatenate hidden state to create $/bf{h}$. RNNs tend to focus on recent input tokens ($h_i$ updates ), therefore hidden state, $h_i$,  will be focused on tokens around $x_i$.
 - Decoding: Decode via forward pass over previous output tokens, hidden state (different from encoding hidden state) $s_t$ and full-length context $c_i$  `p(y_t | \{ y_1, y_2, ..., y_{t-1} \}, c_i) = g(y_{i-1}, s_i, c_i)`
   - Context is produced by alignment model over encoding hidden state:
     - $c_i = \sum_{j=1}^{T_x} \alpha_{ij}h_j$ (easy to be done over matrix multiplication)
     - $\alpha_{ij} = \frac{e^{e_{ij}}}{\sum_{k=1}^{T_x}e^{e_{ik}}$
     - $e_{ij} = a(s_{i-1}, h_j)$. $a(..$ is not latent, i.e. backpropgation updates the soft attention. Learn what hidden state from encoding to use (pay attention to) for context vector $c_i$ that is used as context for decoding $y_i$. Alignment model.