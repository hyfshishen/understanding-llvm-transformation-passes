``-strip``: Strip All Symbols from a Module
=====

Description
--------

``-strip`` pass 旨在删除一些 IR 中没意义的 symbol，比如把刚刚 instrument 的 ``%yafan`` variable 变成 ``%0``。
这种 strip 操作的目标是：

1. Names for virtual registers (也就是我上面拿 ``%yafan`` 举的例子)
2. Symbols for internal global variables and functions (e.g. function name from ``@foo`` to ``@0`` , global variable from ``@yafan`` to ``@2`` ).
3. Debug information

其实对 code 的 functionality 和 performance 不会有任何影响，不过这个 pass 会让 code 变得非常不可读。
所以其使用场景一般在：reducing code size，或者给 reverse engineering 制造麻烦🤣。

类似 ``-strip`` 的 pass 还有好几个，这里顺带一起说了。

- ``-strip-dead-debug-info``: 和 ``-strip`` pass 做的事类似，不过只 strip 没用到的 symbol 的 debug info。
- ``-strip-debug-declare``：和 ``-strip`` pass 做的事类似，不过只 strip 所有 debug info（和 ``-strip-dead-debug-info`` 相比更 aggressive 一些）。
- ``-strip-nondebug``：和 ``-strip`` pass 做的事类似，不过除了 debug info，其他都给 strip 了（including virtual register, global variable, and function names）。

Code Example
--------

这里再拿 function 举一个简单的例子。

原始的 IR code。

.. code-block:: llvm

    define i32 @foo(i32 %x) {
        %result = add i32 %x, 42
        ret i32 %result
    }

``-strip`` transform 之后的 pass。

.. code-block:: llvm

    define i32 @0(i32 %x) {
        %y = add i32 %x, 42
        ret i32 %y
    }

其实就是把 function 和 virtual register 的名字给改了。
