``-strip-dead-prototypes``: Strip Unused Function Prototypes
=====

Description
--------

``-strip-dead-prototypes`` pass 会 iterate 给定 code 中所有的 function，然后找到 dead 的，并将它们全部抹去。
Dead function 指没有实际的 functionality，只有一个 empty 的 declaration 的 function。

这里引申一点 LLVM IR 的背景知识，下面程序是 `NPB benchmark suite <https://www.nas.nasa.gov/software/npb.html>`_ 中 EP program 的 LLVM 3.4 compile 的 IR 中的某一行：

.. code-block:: llvm

    ; Function Attrs: nounwind
    declare double @log(double) #3

我们在碰到一个 function 的时候，一般只会留意这个 function 的 body（具体的 instruction）；但是其实在 LLVM IR 中，还会在某个地方对这个 function 重复 declare 一下。
``-strip-dead-prototypes`` 所 remove 的 target 就是，body 没了但是 declare 还在的 function。

Code Example
--------

只是 strip 一个 declaration 而已，上面已经描述的很详细了，所以这里便不再赘述。其实都是为了降低 code size 然后尝试去优化 performance。
