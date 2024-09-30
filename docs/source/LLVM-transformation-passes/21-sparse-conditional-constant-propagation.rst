``-sccp``: Sparse Conditional Constant Propagation
=====

Description
--------

``-sccp`` pass 尝试将 LLVM IR 中看起来是 variable 但是实际计算过程中是 constant 的 variable 都变成 constant（这可以优化掉一些 computation 和 conditional branch）。
简而言之也是通过将 runtime 的 computation 给放在 compile-time 中，进而对程序的 performance 进行优化。
其实根据我目前的开发经验来说， ``-sccp`` 很难对程序有非常非常高速的优化，因为很多程序的 global variable 都是 user-defined input，这个基本都不是 constant。
但是对于一类定值的程序（比如 Fibonacci）优化性能可以很高，根据 CS6120 `at` Cornell 的 report，可以把程序的 both static and dynamic instructions优化 5%~10%。
同时，这个 pass 带给人的启发也是，写 C/C++/CUDA 代码的时候对于某些固定值的 variable 可以尝试定义为 const 类型，这样在后续的优化中性能会更好。

Code Example
--------

CS6120 `at` Cornell 的例子太好了，所以我这里直接拿过来用了 (`Original Link <https://www.cs.cornell.edu/courses/cs6120/2019fa/blog/sccp/>`_)。
不过这里的 code 不是 LLVM IR 是他们自己做的教学用 BRIL IR，不过也差不多。

原始的 BRIL IR code。

.. code-block:: llvm

        a: int = const 1;
        b: int = add a a;
        cond: bool = const false;
        br cond then else;
    then:
        b: int = add b a;
    else:
        print b;

把原始的 BRIL IR code 给 compile 成 SSA form。

.. code-block:: llvm

        a: int = const 1;
        b_0: int = add a a;
        cond: bool = const false;
        br cond then else;
    then:
        b_1: int = add b_0 a;
    else:
        b_2: int = phi b_0 b_1;
        print b_2;

``-sccp`` transform 过的 BRIL IR code。

.. code-block:: llvm
    :emphasize-lines: 3

    ; the previous if-else branch is optimized here
        a: int = const 1;
        b: int = const 2; ; The complex condition can be optimized into a const value assignment
        print b;

