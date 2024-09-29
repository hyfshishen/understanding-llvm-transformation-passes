``-loop-simplify``: Canonicalize Natural Loops
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../compiler-basics/loop-terminologies-in-llvm`。

Description
--------

``-loop-simplify`` 也就是在让这个 natural loop 给标准化成 simplify form (i.e. canonical form)，也是为了后续的优化能够更精确的执行。
虽然在 background 文档里已经说过了，不过这里再描述一下。
在 LLVM 中，loop simplify form (i.e. canonical form) 指的是满足如下条件的 loop：

1. 一个 preheader。
2. 一个 latch。
3. Exits 被 header dominant（即，这个 loop 的 exit 没有 loop 以外的其他 node 作为 predecessor）。

其实仔细想想，在平时写代码的时候，我们写的 ``do-while``， ``for`` ， 和 ``while`` 其实都是很标准的 form。所以如果 code 写的比较简洁的话，单独使用这个 pass 其实优化没有想象中的那么高，还是得结合别的 loop transformation pass 一起来做比较高效。

Code Example
--------

Background 文档里已经很详细了，所以这里略一下。
