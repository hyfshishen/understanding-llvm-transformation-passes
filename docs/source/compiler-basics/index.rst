**************
Compiler Optimization Basics
**************

在正式学习 LLVM transformation passes 之前，我们也得学习一下这些 pass 里使用到的 compiler optimization basics。
为什么呢？打个比方，对于 ``-bb-vectorize`` pass，如果不知道什么是 loop vectorization 的话，理解这个 pass 简直是寸步难行。
当然，compiler optimization 是个非常大的研究领域，包罗万象；所以本章节只整理了 LLVM 所有的 transformation passes 里需要的知识，并且整理在了下面。
如果你有系统学习这一块知识的需求的话，可以看一下这个文档 [#ref1]_，写的超级棒。
当然，如果你真的只想看某一个特定的 pass，不需要了解任何别的 pass。也可以直接移步 :doc:`../LLVM-transformation-passes/index` 里对应的位置，那里写了了解当前的 pass 哪些 basics 是必须的。这样的回溯学习应该也能提升一些效率。


Contents
--------

.. toctree::
   :maxdepth: 1

   llvm-ir-variables-and-utilization
   ssa-form-in-llvm-ir
   llvm-pass-types
   loop-terminologies-in-llvm
   loop-invariant-code-motion
   loop-transformation
   critical-edges-in-cfg
   value-numbering
   lazy-code-motion
   tail-call-and-recursion

References
--------
.. [#ref1] CS:6120@Cornell, Advanced Compilers: https://www.cs.cornell.edu/courses/cs6120/2022sp/
