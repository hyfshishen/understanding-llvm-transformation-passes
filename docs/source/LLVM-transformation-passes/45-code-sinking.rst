``-sink``: Code Sinking
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../compiler-basics/lazy-code-motion`。

Description
--------

``-sink`` pass 尽可能地将当前 basic block 的 instructions 给 move 到其 successors 中，当然与此同时会考虑程序的正确性和安全性。
其实也就是 :doc:`../compiler-basics/lazy-code-motion` 的 LLVM IR 版本了。

Code Example
--------

例子可以看 background 部分，这里便不再赘述了。
