``-bb-vectorize``: Basic Block Vectorization
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../compiler-basics/loop-transformation`。

Description
--------

``-bb-vectorize`` pass 主要是把 basic block 内部的 instruction 去 form 成 vector instruction (i.e. SIMD instructions)，提升 memory bandwidth utilization 的同时进而提升程序运行的 runtime performance。
``-bb-vectorize`` 是 ``-loop-vectorize`` pass 的升级版本（这个pass在7.0左右的版本出现过，后来进化成这个了）。
``-bb-vectorize`` 相比于 ``-loop-vectorize`` ，涵盖除了 loop 之外的更多 vectorization 的形式，比如不改变 loop 数量的同时直接把 basic block 内部符合条件的 instruction 优化成 vector instruction。

Code Example
--------

例子可以移步 :doc:`../compiler-basics/loop-transformation`。
这个文档已经写得很清楚了所以这里便不再重复。
