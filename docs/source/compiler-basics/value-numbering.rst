Value Numbering
=====

Value numbering 是一种常见的 compiler optimization technique，它通过 identify 那些计算相同 value/operand 的 expression/instruction，去 replace 这些重复的部分 with a single computation/instruction。
所以 value numbering 也很经常地用在 redundant code elimination 里。
我们看一下 local value numbering，这是 value numbering 在 basic block 内的应用（LLVM 里是这样，或者说是 limited scope 内）。
例子 [#ref1]_ 如下代码所示，这里 ``sum2`` 和 ``sum1`` 的 operand 完全相同，都是 ``a`` 和 ``b``；所以其实有一个是 redundant 的，这里可以将其优化掉。

.. code-block:: llvm

    ; original code
    define i32 @original_code(i32 %a, i32 %b) {
        %sum1 = add i32 %a, %b
        %sum2 = add i32 %b, %a
        %mul = mul i32 %sum1, %sum2
        ret i32 %mul
    }

    ; transformed code by value numbering
    define i32 @transformed_code(i32 %a, i32 %b) {
        %sum1 = add i32 %a, %b
        %mul = mul i32 %sum1 %sum1
        ret i32 %mul
    }

References
--------
.. [#ref1] Global Value Numbering in CS:6120@Cornell: https://www.cs.cornell.edu/courses/cs6120/2019fa/blog/global-value-numbering/