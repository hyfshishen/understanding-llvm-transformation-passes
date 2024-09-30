``-reg2mem``: Demote All Values to Stack Slots
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../compiler-basics/llvm-ir-variables-and-utilization` 。

Description
--------

``-reg2mem`` pass 其实也就是 ``-mem2reg`` 的 reverse 版本。
这听起来好像不太 make sense，既然 ``-mem2reg`` 可以避免没有必要的 memory access 进而提升 performance，那么为啥反过来的 ``-reg2mem`` 有意义呢？
``-reg2mem`` 主要有以下几个应用场景，比如 debugging，写一些不支持 SSA form 的 pass，更清晰的 alias analysis（这时候 ``alloca-store`` 会让 memory usage 更明确一些），等等。
所以这时候 ``-reg2mem`` 就派上用场了。

Code Example
--------

因为是 ``-mem2reg`` 的 reverse 过程，所以这个 code example 直接读 ``-mem2reg`` 就好了（直接颠倒过来）。
