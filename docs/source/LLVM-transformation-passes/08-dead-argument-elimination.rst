``-deadargelim``: Dead Argument Elimination
=====

Description
--------

``-deadargelim`` 其实就是把 never execute（也就是 dead 的意思）的 function arguments 给 remove 掉；进而降低 register usage 并提升 runtime performance。
打个比方，我有一个 function 传入了两个 arguments，但是我的 function body 只是 print 了一个 "hello world"，这时候这两个传入的 arguments 就是 dead 的了，应该被 remove掉。
其实平时我们很难写出这种傻逼的没意义的函数，但是在一些 LLVM 的内置 pass 对程序进行优化的时候，可能会引入这种 dead arguments，这时候 ``-deadargelim`` 就是很有意义的了。

Code Example
--------

原始 IR。

.. code-block:: llvm

    define i32 @add_numbers(i32 %a, i32 %b) {
        ret i32 0
    }

``-deadargelim`` transform 之后的 IR。

.. code-block:: llvm

    define i32 @add_numbers() {
        ret i32 0
    }

