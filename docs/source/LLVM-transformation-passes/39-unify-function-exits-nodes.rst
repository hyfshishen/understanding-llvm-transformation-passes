``-mergereturn``: Unify Function Exit Nodes
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../compiler-basics/loop-terminologies-in-llvm` 和 :doc:`../compiler-basics/ssa-form-in-llvm-ir`。

Description
--------

``-mergereturn`` pass 尝试 unify 一个 function 的 exit basic block，使一个 function 至多只有一个 ``ret`` instruction（包括其所在的 basic block ）。
这样可以简化 program control-flow，并且可以使其他的 optimization pass（主要是 control-flow 的 optimization）更高效。

Code Example
--------

这个例子可以帮助理解一下， 为什么多个 ``ret`` 可以被 merge 成一个 ``ret`` 。

给定原始的 IR code。

.. code-block:: llvm
    :emphasize-lines: 8,12

    define i32 @example_func(i32 %x) {
    entry:
        %cmp = icmp sgt i32 %x, 10
        br i1 %cmp, label %greater, label %less_or_equal

    greater:                             ; Basic block with a return statement
        %result_greater = add i32 %x, 5
        ret i32 %result_greater

    less_or_equal:                       ; Basic block with a return statement
        %result_less = sub i32 %x, 3
        ret i32 %result_less
    }

``-mergereturn`` transform 之后的 IR code 如下所示：

.. code-block:: llvm
    :emphasize-lines: 16

    define i32 @example_func(i32 %x) {
    entry:
        %cmp = icmp sgt i32 %x, 10
        br i1 %cmp, label %greater, label %less_or_equal

    greater:
        %result_greater = add i32 %x, 5
        br label %exit

    less_or_equal:
        %result_less = sub i32 %x, 3
        br label %exit

    exit:                                ; Merged exit node
        %final_result = phi i32 [ %result_greater, %greater ], [ %result_less, %less_or_equal ]
        ret i32 %final_result
    }

可以看到，通过 ``phi`` node， ``ret`` 的 variable 被巧妙地 merge 成一个了，所以最后也就只剩了一个 ``ret`` instruction 和其所在的 basic block。
