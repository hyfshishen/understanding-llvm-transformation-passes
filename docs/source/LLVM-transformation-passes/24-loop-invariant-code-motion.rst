``-licm``: Loop Invariant Code Motion
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../compiler-basics/loop-invariant-code-motion`。

Description
--------

``-licm`` pass其实做的也就是 loop invariant code motion 这件事（详情可看 background 部分）。
不过有一点在 LLVM IR 上是不同的，那就是 move store instruction outside 的 loop 的时候可能会影响 SSA 设计原则，所以要结合 ``-mem2reg`` 一起食用。

Code Example
--------

原始的 IR code。

.. code-block:: llvm
    :emphasize-lines: 12

    define void @foo(i32 %n) {
    entry:
        %i = alloca i32
        store i32 0, i32* %i
        br label %loop

    loop:
        %cmp = icmp slt i32 %n, 10
        br i1 %cmp, label %body, label %exit

    body:
        %load_i = load i32, i32* %i
        %add = add i32 %load_i, 1
        store i32 %add, i32* %i
        br label %loop

    exit:
        ret void
    }

``-licm`` transform 的 IR code。

.. code-block:: llvm
    :emphasize-lines: 5

    define void @foo(i32 %n) {
    entry:
        %i = alloca i32
        store i32 0, i32* %i
        %load_i = load i32, i32* %i
        br label %loop

    loop:
        %cmp = icmp slt i32 %n, 10
        br i1 %cmp, label %body, label %exit

    body:
        %add = add i32 %load_i, 1
        store i32 %add, i32* %i
        br label %loop

    exit:
        ret void
    }

可以看到， ``%load_i = load i32, i32* %i`` 被拿到 loop 外面了。
