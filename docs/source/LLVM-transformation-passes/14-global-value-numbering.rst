``-gvn``: Global Value Numbering
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../compiler-basics/value-numbering` 和 :doc:`../compiler-basics/ssa-form-in-llvm-ir`。

Description
--------

主要的需要理解的内容都可以在 background document 里找到。
在理解 background之后， ``-gvn`` 的意义也就很显而易见了，其实就是考虑 SSA format 下的，更大 scope 的（比如 entire program）的 value numbering。
这里再总结理解一下，value numbering 的核心是将尽可能的将 runtime 时做的事放在 compile-time 来做，来 simplify redundant operands and instructions，进而提升 runtime performance。

值得一提的是在 LLVM pass 的 implementation 中， ``-gvn`` 有时候也会做一些 dead load elimination 的操作。


Code Example
--------

背景链接里有非常详细的解释，这里便不再赘述。
