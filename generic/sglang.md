# SGLang

[Source](https://arxiv.org/pdf/2312.07104)

## Target problem

Previous serving systems often discarded the KV cache after completing a request or did not automatically reuse it across independent requests. As a result, even when a new request shared a long prefix with a previously processed request, the engine had to recompute the KV cache for that prefix during prefill.

In addition, processing requests with unrelated prefixes between requests that shared a prefix could reduce cache locality and cause premature eviction of reusable KV cache entries.

## Core idea

**RadixAttention** stores the KV cache in a radix tree, where each path represents a sequence of tokens. When a new request arrives, the runtime searches the tree for the longest cached prefix matching the beginning of the request.

If a match is found, the corresponding KV cache is reused, and the engine performs prefill only for the remaining, uncached suffix instead of recomputing the entire sequence.

RadixAttention combines this automatic prefix reuse with:

- **LRU eviction**, which removes the least recently used cache entries when GPU memory is needed;
- **cache-aware scheduling**, which prioritizes requests with longer cached prefixes and tries to process requests sharing the same prefix close to one another. This improves cache locality and reduces cache thrashing.

## Limitations

**LRU eviction considers only past accesses and does not account for future cache usage.**

As a result, the engine may evict an agent's KV cache because it has not been accessed recently, even though the agent is about to be invoked again. This leads to a cache miss and requires either recomputing the prefix during prefill or loading its KV cache back from a lower memory tier.
