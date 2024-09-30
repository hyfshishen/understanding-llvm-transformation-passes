``-prune-eh``: Remove Unused Exception Handling Info
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../compiler-basics/llvm-pass-types` 。

Description
--------

``-prune-eh`` pass 做的事有点像 ``-lowerinvoke`` ，都是把 ``invoke`` instruction 给 downgrade 成 ``call`` instruction。
不过 ``-prune-eh`` 是一个interprocedual pass，它通过 walk through program control-flow 的 call graph，check 当前 ``invoke`` 所调用的 function（i.e. callee）有没有抛出 exception，如果没有的话那么就把 ``invoke`` 给 downgrade 成 ``call``。
相比于 ``-lowerinvoke`` ， ``-prune-eh`` 的操作要更智能一些。

Code Example
--------

多了个 check 过程，但是其工作方式和 ``-lowerinvoke`` 是一样的，都是把 ``invoke`` 给 downgrade 成 ``call`` ，所以这里例子看之前的 ``-lowerinvoke`` 的就好了，这里略一下。
