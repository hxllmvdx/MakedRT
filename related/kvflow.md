# KVFlow

[Source](https://arxiv.org/pdf/2507.07400)

## Target problem

In agentic workflows, conventional cache eviction policies such as LRU do not account for the future execution order of agents. As a result, the engine may evict an agent's KV cache because it has not been accessed recently, even though the agent is about to be invoked again.

This leads to cache misses and additional overhead from either recomputing the evicted prefix or loading its KV cache back from CPU memory. In the latter case, reactive CPU-to-GPU loading can also stall generation.

## Core idea

KVFlow represents an agentic workflow as an **Agent Step Graph**, which captures dependencies between agent invocations. For each agent, it computes a **steps-to-execution** value — an estimate of how many steps remain until the agent's earliest possible next invocation.

KVFlow uses this information for two main optimizations:

- **Workflow-aware eviction:** eviction priorities are assigned to individual nodes of the prefix-cache tree. KV entries belonging to agents that are expected to run sooner are retained longer, while entries that will not be needed soon are evicted first.
- **Proactive KV prefetching:** KV caches required by upcoming agents are asynchronously loaded from CPU to GPU memory before those agents are scheduled. This allows KV transfers to overlap with the current agent's computation.

KVFlow also introduces **status-aware scheduling**, which temporarily skips requests whose KV caches are still being loaded and prioritizes other ready requests.

## Limitations

The effectiveness of KVFlow depends on the availability of information about the workflow structure. Although Agent Step Graphs can represent conditional branches and synchronization barriers, eviction and prefetching become less informed when the future execution order is highly dynamic or unavailable.

Prefetching also does not always finish before the next agent needs to run, especially when the current agent's execution time is shorter than the KV transfer time or when multiple workflows compete for CPU-to-GPU bandwidth. Status-aware scheduling mitigates this problem but does not always eliminate it completely.

Finally, the performance gain diminishes when decoding dominates the overall execution time, because KVFlow primarily optimizes prefix-cache management and prefill-related overhead rather than token generation itself.
