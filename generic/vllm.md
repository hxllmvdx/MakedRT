# vLLM

[Source](https://arxiv.org/pdf/2309.06180)

## Target problem

Earlier inference engines allocated large contiguous memory regions for the KV cache, often based on the maximum possible sequence length rather than the actual length of a request. This led to memory underutilization and both internal and external fragmentation, leaving less memory available for new requests.

## Core idea

**PagedAttention** divides the KV cache into fixed-size blocks, each storing the keys and values for a predetermined number of tokens. A **block table** maps the logical blocks of a sequence to physical memory blocks, so consecutive tokens do not need to be stored in contiguous physical memory.

KV blocks are allocated on demand as the sequence grows, rather than being preallocated for its maximum possible length. This eliminates external fragmentation and limits internal fragmentation to unused slots in the last block of a sequence.

## Limitations

PagedAttention itself does not provide automatic prefix-cache reuse across independent requests. At the time of the paper's publication, if two requests shared the same prefix, the KV cache for that prefix could still be recomputed for the later request instead of being automatically reused.

This limitation concerns the original PagedAttention design, not later versions of vLLM with Automatic Prefix Caching.
