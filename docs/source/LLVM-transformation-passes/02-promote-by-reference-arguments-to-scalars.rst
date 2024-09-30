``-argpromotion``: Promote 'by reference' Arguments to Scalars
=====

Description
--------

``-argpromotion`` pass 主要是把 function 的 arguments 形式给升级一下（如果满足条件的话），即从 "by reference" 提升为 "by scalar"。

- "by reference" 指参数调用的是个引用，比如是个指针（pointer arguments），传递的是形参。
- "by scalar" 指参数调用的调用的是实际的值，比如是个 int type 的数，传递的是实参。

这样做可以省去额外的 ``load`` 还有 ``alloca`` instructions，进而提升程序的 runtime performance。
当然， ``-argpromotion`` 是有条件的，需要通过 alias analysis (i.e. pointer analysis) 来判定这个 arguments 只被读取过，而没有进行过写的操作，这之后 ``-argpromotion`` 才能用。
反之，如果说这个参数被写了：比如 input argument 是个动态 malloc 过的 int type pointer，且里面的数值都被大量修改了。
那么 ``-argpromotion`` 就万万不能用了，因为写的值传不回去。

``-argpromotion`` 可以对 code 进行 recursive simplification，并且 eliminating 很多 ``load`` 还有 ``alloca``（这对 C++ template code 尤其高效，比如STL）。

Code Example
--------

原始的 IR code。

.. code-block:: llvm
    :emphasize-lines: 1,2,3

    define i32 @foo(i32* %a, i32* %b) {
        %1 = load i32, i32* %a
        %2 = load i32, i32* %b
        %sum = add i32 %1, %2
        ret i32 %sum
    }

经过 ``-argpromotion`` 之后的 IR code。

.. code-block:: llvm
    :emphasize-lines: 1,2

    define i32 @foo(i32 %a, i32 %b) {
        %sum = add i32 %a, %b
        ret i32 %sum
    }

很好理解，就是把函数的 argument 从 pointer 给变成 int type data 了，减少了额外的 ``load`` instruction 从而降低了代码开销。