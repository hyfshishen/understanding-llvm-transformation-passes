``-loop-unroll``: Unroll Loops
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../compiler-basics/loop-transformation`。

Description
--------

``-loop-unroll`` 在做的事也就是将 loop 给 unroll 了，虽然增大了 code size 但是降低了 loop 的数量以及 branch penalty，所以 performance 可以 possibly 的提升。
注意， ``-loop-unroll`` pass 对 ``-loop-simplify`` 和 ``-indvars`` passes 给 canonicalized 过的 loop 优化程度最高。

虽然这里的描述比较少，但是 ``-loop-unroll`` 真的是个很常用很有效的技术。


Code Example
--------

Background 里非常详细，所以这里便不再赘述。
