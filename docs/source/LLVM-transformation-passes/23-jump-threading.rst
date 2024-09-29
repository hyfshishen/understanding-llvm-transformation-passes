``-jump-threading``: Jump Threading
=====

Description
--------

``-jump-threading`` pass 在做的事是，如果发现一个程序有多个 branch，而某些 branch 如果执行了会让接下来的某个 branch 明显不执行，那么就可以把那个明显不执行的 branch 在这里优化掉（直接jump过去）。
从 basic block 的 control-flow 层面来说，这个过程可以这么解释：如果一个 basic blocks 有多个 predecessors 和 successors，而且一个或多个 predecessor 的执行总是会指向一个特定的 successor，那么就直接把这个 edge 给 forward 过去（内容 duplicate 一下）。
通过这种方式可以把一些 jumping 放在在 compile-time，进而提升 runtime 的 performance。

还是有点抽象，那么用一段代码举例一下。
如下所示，这里如果 ``x=4`` 执行了，那么 ``x`` 一定不会小于 ``3`` ，那么这时候其实可以把 ``x<3`` 对应的 else branch 给 duplicate 一下，然后直接和第一个 if branch 连起来。
这样就降低了 runtime 过程中的 basic block jumping 和 control-flow 的复杂程度。

.. code-block:: c

    if (...) X = 4;
    if (X < 3) {...

其实对于搞 parallel computing 的人来说，这里 threading 这个单词还是很 confusing 的！感觉如果是 ``-jump-bb`` 听起来会更精确一些。

Code Example
--------

原始的 IR code。

.. code-block:: llvm

    define i32 @foo(i32 %x) {
        %cmp = icmp slt i32 %x, 10
        br i1 %cmp, label %then, label %else

    then:
        %add1 = add i32 %x, 1
        br label %merge

    else:
        %add2 = add i32 %x, 2
        br label %merge

    merge:
        %result = phi i32 [ %add1, %then ], [ %add2, %else ]
        ret i32 %result
    }

``-jump-threading`` transform 过的 IR code。

.. code-block:: llvm

    define i32 @foo(i32 %x) {
        %cmp = icmp slt i32 %x, 10
        %add1 = add i32 %x, 1
        %add2 = add i32 %x, 2
        %result = select i1 %cmp, i32 %add1, i32 %add2
        ret i32 %result
    }

简而言之，就是如果发现 basic block 的 control-flow 走向是固定的，就直接给 merge 掉有歧义的部分。
