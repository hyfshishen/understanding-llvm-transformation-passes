``-tailcallelim``: Tail Call Elimination
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../compiler-basics/tail-call-and-recursion`。

Description
--------

``-tailcallelim`` pass 则是 eliminate 掉 tail call 的 code，以达到更大的优化。
虽然 background 里举了一个例子 elimiate tail recursion，但是如果所计算的数据非常大的话的时候这个 function 一层套一层所带来的 memory stack-heap 开销还是不可接受的的。
所以 LLVM 尝试去 eliminate 所有的情况。其实在 IR 层面他做的事很简单，只是对对应的 ``call`` instruction 添加一个 tail 的 key word。
但是在 assembly 层面上，当前的 function end 便可以直接 jump 到下一个 function。

值得一提的是，对于 tail recursion，如果有很明显的计算表达的话，这个 pass 还会将其优化成 associative expression to use an accumulator variable 的形式，可以极大地提升代码的简洁性。

Code Example
--------

这里给一个我常用的 `MiBench benchmark suite <https://vhosts.eecs.umich.edu/mibench/>`_ 的 bitcount 程序里的一个 function 为例子吧。代码如下所示。

.. code-block:: llvm
    :emphasize-lines: 5

    ; Function Attrs: nounwind uwtable
    define noalias i8* @alloc_bit_array(i64 %bits) #0 {
        %1 = add i64 %bits, 7
        %2 = lshr i64 %1, 3
        %3 = tail call noalias i8* @calloc(i64 %2, i64 1) #5
        ret i8* %3
    }

倒数第二个 instruction 其实只是一个 function ``call`` ；但是这里却多加了一个 ``tail`` 的 key words。

这样 compiler 就会在底层的编译中将其给 eliminate 掉，而进入下一个 function 的方式变为了直接的 goto，降低开销。
LLVM 的官方文档对于这段话的原文是这样的：replace the call of current function with a branch to the entry of the function, creating a loop。其实是一个意思。
