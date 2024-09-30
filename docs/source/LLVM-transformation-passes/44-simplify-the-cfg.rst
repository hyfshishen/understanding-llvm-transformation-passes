``-simplifycfg``: Simplify the CFG
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../compiler-basics/loop-terminologies-in-llvm`。

Description
--------

``-simplifycfg`` pass，类似 instruction level 的 ``-dce`` 等 pass，是 basic block (i.e. the minimal units of program control-flow graph) level 的 optimization pass。
通过 simplifying program control-flow graph（例如 dead basic block elimination or basic block merging）的方式优化程序的 performance。具体而言它可以优化的情景有四种：

1. Removing basic blocks with no predecessors.
    因为没有 basic block 会通向这个 basic block, obviously dead 了，所以可以被remove掉。
2. Merging a basic block into its predecessor if there is only one and the predecessor only has one successor.
    很明显都是单向路径，所以直接 merge 就好了，这道理就像笔直的道路没有必要单独开一个红绿灯一样。
3. Eliminating Phi nodes for basic blocks with a single predecessor.
    因为只有一个predecessor，所以 ``Phi`` node 只有一个赋值的方向，还不如直接等于，所以可以被 eliminate。
4. Eliminating a basic block that only contains an unconditional branch.
    这个可能有点抽象，所以给出一个 code example。我们先给定一个原始的 IR code。

    .. code-block:: llvm
        :emphasize-lines: 6,7

        define i32 @example_func(i32 %x) {
        entry:
            %cmp = icmp sgt i32 %x, 10
            br i1 %cmp, label %greater, label %less_or_equal

        greater:                             ; Basic block with only an unconditional branch
            br label %exit

        less_or_equal:                       ; Basic block with useful instructions
            %sub = sub i32 10, %x
            %result = add i32 %sub, 5
            br label %exit

        exit:
            %final_result = phi i32 [ 0, %entry ], [ %result, %less_or_equal ], [ 0, %greater ]
            ret i32 %final_result
        }

    可以看到，这会 ``greater`` basic block 的 ``br`` instruction 只跳转到一个固定的 basic block，是一个 unconditional branch，没有任何实际的操作。
    所以这个 block 可以被删掉（i.e. 直接从其 predecessor 跳转到其 successor）。
    ``-simplifycfg`` transform 过的 IR code 如下所示：

    .. code-block:: llvm

        define i32 @example_func(i32 %x) {
        entry:
            %cmp = icmp sgt i32 %x, 10
            br i1 %cmp, label %less_or_equal, label %less_or_equal

        less_or_equal:                       ; Basic block with useful instructions
            %sub = sub i32 10, %x
            %result = add i32 %sub, 5
            br label %exit

        exit:
            %final_result = phi i32 [ 0, %entry ], [ %result, %less_or_equal ], [ 0, %entry ]
            ret i32 %final_result
        }


Code Example
--------

上面已经给过一个比较清楚的 code example 了，所以这里略。