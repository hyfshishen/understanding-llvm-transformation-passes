``-lcssa``: Loop-Closed SSA Form Pass
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../compiler-basics/loop-transformation` 和 :doc:`../compiler-basics/ssa-form-in-llvm-ir`。

Description
--------

``-lcssa`` 会添加一些额外的 ``phi`` instruction，这个 ``phi`` instruction 是完全 redundant 的，而且可以被 ``-instcombine`` pass 删去。
这听起来没什么意义，不过对于 ``-loop-unswitch`` 等 loop passes 却很有用，会让他们的优化变得很简单。
此外， ``-lcssa`` 还会保持代码功能的正确性，以 ``-loop-unsiwtch`` 为例，因为它打破了原有的 basic block 结构，所以有些变量的命名可能会没有遵守 SSA 原则，这里 ``-lcssa`` 会对其补全并完善。

Code Example
--------

这个 transform 比较复杂，但是结果却是 trivial 的，所以这里就不写例子了。
详细的解释可以看一看这个链接 `LCSSA <https://llvm.org/docs/LoopTerminology.html#loop-terminology-lcssa>`_ 。