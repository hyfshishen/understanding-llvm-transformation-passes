``-mem2reg``: Promote Memory to Registers
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../compiler-basics/llvm-ir-variables-and-utilization`。

Description
--------

``-mem2reg`` pass 在做的事也是把 stack-allocated 的 variable 给 promote 成 SSA registers，这样就直接变成了 instruction-level computations，减少了很多不必要的 ``alloca`` ， ``store`` ，和 ``load`` instructions，进而提升performance。

Code Example
--------

给定一段如下的 C code。

.. code-block:: C

    int sum(int n) {
        int result = 0;
        for (int i = 1; i <= n; i++) {
            result += i;
        }
        return result;
    }

我们先把它 compile 成原始的 IR code。

.. code-block:: llvm

    define i32 @sum(i32 %n) {
    entry:
        ; Allocate space on the stack for 'result' and 'i'
        %result = alloca i32
        %i = alloca i32
        ; Initialize 'result' to 0
        store i32 0, i32* %result
        ; Initialize 'i' to 1
        store i32 1, i32* %i
        ; Jump to the condition check of the loop
        br label %loop.cond

    loop.cond:
        ; Load the value of 'i'
        %loaded_i = load i32, i32* %i
        ; Compare 'i' with 'n'
        %cmp = icmp sle i32 %loaded_i, %n
        ; Conditional branch based on the comparison result
        br i1 %cmp, label %loop.body, label %loop.end

    loop.body:
        ; Load 'result' and 'i'
        %loaded_result = load i32, i32* %result
        %loaded_i = load i32, i32* %i
        ; Add 'i' to 'result'
        %sum = add i32 %loaded_result, %loaded_i
        ; Store the updated 'result' back
        store i32 %sum, i32* %result

        ; Increment 'i'
        %next_i = add i32 %loaded_i, 1
        store i32 %next_i, i32* %i
        ; Jump back to the loop condition check
        br label %loop.cond

    loop.end:
        ; Load the final value of 'result'
        %final_result = load i32, i32* %result
        ; Return 'result'
        ret i32 %final_result
    }

经过 ``-mem2reg`` transform 之后的 IR 可以变成如下所示

.. code-block:: llvm

    define i32 @sum(i32 %n) {
    entry:

        ; 'result' and 'i' are promoted to SSA registers
        %result = phi i32 [ 0, %entry ], [ %sum, %loop.body ]
        %i = phi i32 [ 1, %entry ], [ %next_i, %loop.body ]
        ; Jump to the condition check of the loop
        br label %loop.cond

    loop.cond:
        ; Compare 'i' with 'n'
        %cmp = icmp sle i32 %i, %n
        ; Conditional branch based on the comparison result
        br i1 %cmp, label %loop.body, label %loop.end

    loop.body:
        ; Add 'i' to 'result' using the SSA registers directly
        %sum = add i32 %result, %i
        ; Increment 'i' using the SSA register directly
        %next_i = add i32 %i, 1
        ; Jump back to the loop condition check
        br label %loop.cond

    loop.end:
        ; 'result' is already in SSA form and does not need loading
        ; Return 'result' directly
        ret i32 %result
    }

可以看到，不必要的 memory access，包括 ``alloca`` ， ``store`` ， 和 ``load`` instructions 都被 remove 了。性能得到了提升。
