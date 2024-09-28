``-adce``: Aggressive Dead Code Elimination
=====

Description
--------

``-adce`` 的全称如标题所示，主要就是对 Program IR 进行 dead code elimination 的 code transformation。
其实和 ``-dce`` pass 做的事一样，不过更 aggressive 一些，这个区别在 :doc:`07-dead-code-elimination` 那里讲。
从 high-level 上来说，``-adce`` pass 可以 identify 那些在程序中一定 **不会运行** 的 code (both instruction- and basic block-levels)，然后 remove 他们。
这种一定不会运行的 code 主要包括：

- Control-flow 没有一个可以到达这个 basic block 的，而且对后续变量没有任何影响的（e.g. 我写了个函数，但是没有任何地方调用它）
- Data-flow 无关的变量。（e.g. 我定义了一个变量，但是后面压根没使用它）


Code Example
--------

下面是一段原始的 LLVM IR。

.. code-block:: llvm

    define i32 @foo(i32 %x, i32 %y) {
    entry:
        %a = add i32 %x, %y
        %b = add i32 %a, 1
        %c = add i32 %a, 2
        %d = icmp slt i32 %y, 0
        br i1 %d, label %then, label %else

    then:
        %e = add i32 %b, 1
        ret i32 %e

    else:
        %f = add i32 %c, 1
        ret i32 %f
    }


这是经过 ``-adce`` pass 之后的 IR。

.. code-block:: llvm

    define i32 @foo(i32 %x, i32 %y) {
    entry:
        %a = add i32 %x, %y
        %b = add i32 %a, 1
        %d = icmp slt i32 %y, 0
        br i1 %d, label %then, label %else

    then:
        %e = add i32 %b, 1
        ret i32 %e

    else:
        %f = add i32 %b, 2
        ret i32 %f
    }

因为 ``%c`` 变量这里没被使用，所以就被 eliminate 了。