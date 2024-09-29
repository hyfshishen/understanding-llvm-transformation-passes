``-indvars``: Canonicalize Induction Variables
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../loop-invariant-code-motion`， :doc:`../compiler-basics/ssa-form-in-llvm-ir`， 和 :doc:`../compiler-basics/loop-terminologies-in-llvm`。

Description
--------

``-indvars`` 的全称如标题所示，旨在规范化当前 loop (or other forms) 中的 induction variables，将其变成更 simple 的格式，这样可以方便后续的 code analysis 和 transformation，从而提升 code performance。
根据官方文档， ``-indvars`` 这个 pass 会对每个 loop 做如下几件事：

1. 所有的 loop 都会被 transform 只有一个 **single** induction variable，而且这个 induction variable 是 canonical 的（starting from 0）。
2. The transformed canonical induction variable 一定是 loop 的 head basic block 的 ``first`` PHI node。
3. 指针的递归操作会使用数组角标完成。

Code Example
--------

官方文档有个例子挺好的，我们也直接用这个了。

原始的 C code。

.. code-block:: C

    for (i = 7; i*i < 1000; ++i)

``-indvars`` transform 之后的 C code。

.. code-block:: C

    for (i = 0; i != 25; ++i) // i starting from 0
