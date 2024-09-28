``-adce``: Aggressive Dead Code Elimination
=====

Description
--------

``-adce`` 的全称如标题所示，主要就是对 Program IR 进行 dead code elimination 的 code transformation。
其实和 ``-dce`` pass 做的事一样，不过更 aggressive 一些，这个区别在 :doc:`07-dead-code-elimination` 那里讲。
从 high-level 上来说，``-adce`` pass 可以 identify 那些在程序中一定不会运行的 code (both instruction- and basic block-levels)，然后 remove 他们。

Code Example
--------

假装这是一段example。

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


假装这是一段example。

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
