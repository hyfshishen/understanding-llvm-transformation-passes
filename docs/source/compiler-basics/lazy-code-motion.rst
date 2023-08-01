Lazy Code Motion
=====

Lazy code motion 尝试将当前 component (e.g. basic block in LLVM IR) 内的 code 尽可能地往后面的 component 移动。
这样，如果下面的 component 是只有某些条件才运行的话，相当于有概率地减轻了当前 component 的 computation overhead。
LLVM document 对此的解释是：so they aren’t executed on paths where their results aren’t needed。
下面结合一个例子来理解：这里 ``; Perform lazy code motion`` 代表了可以被移动到下一个 basic block 的 code，其目标位置是 ``; Lazily compute the result when needed``。

.. code-block:: llvm
    define i32 @foo(i32 %x, i32 %y) {
    entry:
        %add = add i32 %x, %y

        ; Perform lazy code motion
        %result = call i32 @lazyCompute(i32 %add)

        ret i32 %result
    }

    define i32 @lazyCompute(i32 %input) {
    entry:
        ; Lazily compute the result when needed
        %mul = mul i32 %input, 10
        %div = sdiv i32 %mul, 2

        ret i32 %div
    }

虽然看起来 code size 和 computation 都没有降低或者减少，但是如果 ``@lazyCompute`` 这个 function call 是选择性执行的，那就相当于把原有 path 中的 unnecessary code 给放到后面了。
相当于尽可能地精炼了 necessary path 中的 computation，进而去 possibly 优化 performance。
CS:6120@Cornell 的同学们也对此做了实验分析，更细致的了解可以看一下这个 link [#ref1]_。

References
--------
.. [#ref1] Lazy Code Motion in CS:6120@Cornell: https://www.cs.cornell.edu/courses/cs6120/2019fa/blog/lazy-code-motion/
