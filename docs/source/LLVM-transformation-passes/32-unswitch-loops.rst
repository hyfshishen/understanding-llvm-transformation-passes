``-loop-unswitch``: Unswitch Loops
=====

Description
--------

``-loop-unswitch`` 和 ``-licm`` 很接近，但是是把 loop-invariant 的 ``if-else`` branch 给提到 loop 外面。
虽然这样会让 code size 几乎 double，但是这样可以让 loop body 内重复做的 ``if-else`` branch 变成只运行一次，极大地降低了 loop penalty，进而 possibly 提升 runtime performance。

注意， ``-loop-unswitch`` expect ``-licm`` pass 已经处理过这个 code 了。

Code Example
--------

同样，loop 相关的我们用 C code 来举例。

.. code-block:: C

    for(...){
        func_A();
        if(...)
            func_B();
        func_C();
    }

``-loop-unswitch`` 会把这个 code transform 成这个样子。

.. code-block:: C

    if(...){
        for(...){
            func_A();
            func_B();
            func_C();
        }
    }
    else{
        for(...){
            func_A();
            func_C();
        }
    }

可以看到，确实 code size 大了很多。但是只要能降低 runtime 时候的 branch penalty，那么就是有意义的。
总而言之这个的优化思路也是让 runtime 时候的 computation 在 compile-time 里完成。
