``-loweratomic``: Lower Atomic Intrinsics to Non-Atomic Form
=====

Description
--------

``-loweratomic`` pass 旨在将 atomic 的 intrinsics (一般来说指非常 low-level 的可以 directly access hardware resources 的 operation) 变成 non-atomic 的。
这个意义在于，在多线程程序中，一些操作被设定为 atomic 来处理 synchronization 等特殊场景。
但是很多时候，这种 atomic 的操作是非常昂贵的。
所以， ``-loweratomic`` pass 的目的是把所有 atomic 的操作都降级成非 atomic 的，这样可以降低 overhead。

注意， ``-loweratomic`` pass 并不会 verify 程序的执行环境是不是 non-preemptible 的，即分析 external library or entire call graph 来判断这么做对不对，它只能 simply 把所有的 atomic operation 给降级。
所以这个 pass 也不是特别智能，只能说慎用。

Code Example
--------

原始的 IR code。

.. code-block:: llvm
    :emphasize-lines: 2

    define void @increment(int* %ptr) {
        %atomic_inc = atomicrmw add i32* %ptr, i32 1 seq_cst
        ret void
    }

``-loweratomic`` transform 过的 IR code。

.. code-block:: llvm
    :emphasize-lines: 2,3,4

    define void @increment(int* %ptr) {
        %load = load i32, i32* %ptr
        %inc = add i32 %load, 1
        store i32 %inc, i32* %ptr
        ret void
    }
