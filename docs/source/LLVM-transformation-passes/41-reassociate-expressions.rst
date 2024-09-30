``-reassociate``: Reassociate Expressions
=====

Description
--------

``-reassociate`` pass 通过调整 operator 中 operand 的顺序或者是 instruction 的顺序，使程序更有利于基于 constant propagation 的 optimization，比如 ``-gvn`` 和 ``-instcombine``。
打个比方，对于 ``4 + (x + 5)`` ， 我们可以使用 ``-reassociate`` pass 使其变成 ``x + (4 + 5)`` ，这时候后面的 ``(4 + 5)`` 都是 constant，所以这部分就可以通过 constant propagation pass 直接再 compile time 计算成 ``9``，进而提升 runtime performance。

Code Example
--------

上面给了一个 reorder operand 的例子，那么下面给一个 reorder instruction 的。
这里通过 reorder instructions， ``%mul1 = mul i32 %a, %b`` 和 ``%mul2 = mul i32 %b, %a`` 被连在了一起，这样其就可以被 ``-gvn`` pass 给判定是 redundant computations，然后直接优化掉。

原始的 IR code。

.. code-block:: llvm
    :emphasize-lines: 4,5

    define i32 @example_func(i32 %a, i32 %b, i32 %c) {
    entry:
        %mul1 = mul i32 %a, %b
        %add1 = add i32 %mul1, %c
        %mul2 = mul i32 %b, %a
        %add2 = add i32 %c, %mul2
        %result = add i32 %add1, %add2
        ret i32 %result
    }

``-reassociate`` transform 之后的 IR code。

.. code-block:: llvm
    :emphasize-lines: 4,5

    define i32 @example_func(i32 %a, i32 %b, i32 %c) {
    entry:
        %mul1 = mul i32 %a, %b
        %mul2 = mul i32 %b, %a
        %add1 = add i32 %mul1, %c
        %add2 = add i32 %mul2, %c
        %result = add i32 %add1, %add2
        ret i32 %result
    }
