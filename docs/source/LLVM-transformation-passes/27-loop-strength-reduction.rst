``-loop-reduce``: Loop Strength Reduction
=====

Description
--------

``-loop-reduce`` pass，顾名思义，在做的事其实就是降低 loop 的 strength，即降低 loop 内运算量。
其实 strength reduction 是一个 compiler optimization 上常用的技巧（在 ``-instcombine`` pass 中其实也有涉猎），比如我在写 CUDA 的时候想要得到 warp 的 index，会用到 ``warp_idx = thread_idx/32;`` 的语句；这个计算可以被优化成更轻量级的 bitwise operation： ``warp_idx=thread_idx>>5;`` ；这其实就是一种 strength reduction。

那什么是 loop 中的 strength reduction 呢？LLVM 规范了一种如下的情况，对于一个 loop 的 induction variables（比如经常作为循环条件的 ``i`` , ``j`` , ``k`` ），如果 loop 内有一个与之相关的新的 variable（需要每次进 loop 都 define 一次的那种），则可以把与之相关的计算拿到 loop 外面，并用等价的计算替换原有的 loop，这样其实就把 loop 内的运算量给大大降低了。\
描述有点抽象，看看下面的例子会很好理解。

Code Example
--------

这里用一个 C code 作为 example 来讲解（我感觉只要是和 loop 相关的概念，C 的解释会比 LLVM IR 直观很多）。

原始的 C code。

.. code-block:: C

    int i = 0;
    while( i < 10 ) {
        int j = 3 * i + 2;
        a[j] = a[j] - 2;
        i = i + 2;
    }

``-loop-reduce`` transform 过的 C code。

.. code-block:: C

    int i = 0;
    int j = 2; // j = 3 * 0 + 2
    while( i < 10 ) {
        a[j] = a[j] - 2;
        i = i + 2;
        j = j + 6; // j = j + 3 * 2
    }

与右侧相比，左侧的 original code 每次都要在 loop 内 define 一个新的 variable ``j``，而且 update ``j`` 要进行两次 arithmetic operation（一次 ``add`` 一次 ``mul`` ）。
然而优化过之后 ``j`` 的 definition/initialization 被放在 loop 外了，而且 update ``j`` 也只需要一次 arithmetic operation。
Performance 自然也就可以提升很多了。
