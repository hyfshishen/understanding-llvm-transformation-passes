``-globaldce``: Dead Global Elimination
=====

Description
--------

``-globaldce`` 通过一种非常 aggressive 的算法，判定哪些 global variable 是会被 access 的（i.e. alive 的，也就是被 store 且 load 然后参与 code execution）；然后对于其他的那就判定为 dead 了 并将 这些 dead global variable 全部都删了。
This also allows compiler to delete recursive chunks of the program which are unreachable。

Code Example
--------

原始的 IR code。

.. code-block:: llvm

    @gVariable = global i32 10   ; Global variable with initial value 10

    define void @example(i1 %flag) {
        %result = add i32 5, 7
        br i1 %flag, label %trueBranch, label %falseBranch

    trueBranch:
        store i32 20, i32* @gVariable
        ret void

    falseBranch:
        store i32 30, i32* @gVariable
        ret void
    }

``-globaldce`` transform 的 IR code。

.. code-block:: llvm

    define void @example(i1 %flag) {
        %result = add i32 5, 7
        br i1 %flag, label %trueBranch, label %falseBranch

    trueBranch:
        ret void

    falseBranch:
        ret void
    }

因为 ``@gVariable`` 被判定为只是被 store 没有参与执行，所以是 not accessible 的（i.e. 被判定已经 dead 了），所以这里就把其定义和与之相关的 instruction 全 remove 了。
