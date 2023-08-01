Loop Transformation
=====

Loop transformation 是一系列 compiler 层级修改 loop structure 的技术。
它可以 achieving better cache utilization，reducing loop overhead，亦或者 optimizing data access patterns，进而优化 overall program exectuion。
我们接下来结合代码样例，介绍一些比较常见的 loop transformation techniques。

Loop Unrolling
--------
Loop unrolling 其实就是把 loop 按照不同的 stride 给展开。
虽然增大了 code size，但是降低了 branch penalty 的开销。
当然，如果增大的 code size 太大的话，这个技术就得不偿失了。
下面用一段 C code 来举个例子。

.. code-block:: C

    // original loop
    int x;
    for (x = 0; x < 100; x++)
    {
        delete(x);
    }

    // loop after unrolling (stride size = 4)
    int x; 
    for (x = 0; x < 100; x += 5 )
    {
        delete(x);
        delete(x + 1);
        delete(x + 2);
        delete(x + 3);
        delete(x + 4);
    }

Loop Vectorization
--------
Loop vectorization 用 vector instruction 一次处理 loop 中的多个数据，通过提升 memory bandwidth utilization 进而降低 loop 内循环次数（i.e. branch penalty），进而提升程序的 overall performance。
它的本质其实是一种SIMD (single instruction multiple data) 的思想。
下面同样用一段 C code 举例解释一下：

.. code-block:: C

    // original loop
    for (int i = 0; i < n; i++) {
        a[i] = b[i] + c[i];
    }

    // loop after vectorization
    for (int i = 0; i < n; i += 4) {
        __m128d b_vec = _mm_load_pd(&b[i]);
        __m128d c_vec = _mm_load_pd(&c[i]);
        __m128d sum_vec = _mm_add_pd(b_vec, c_vec);
        _mm_store_pd(&a[i], sum_vec);
    }

本来 loop 要执行 ``n`` 次，不过这里使用了 vector 类型的 instruction，而且每个 vector 都包含四个 int 型的数字。
这样，loop 的每个 iteration 都可以进行四倍的计算量，从而使 loop 循环的次数减小到了原来的四分之一。
关于 loop Vectorization 还有一个例子。原理和上面例子相同，也可以参考一下 [#ref1]_。

Loop Fusion & Fission
--------

Loop fusion (i.e. loop jamming) 总是和 loop fission (i.e. loop distribution) 放在一起说。
Loop fusion 就是把多个 loop 合成一个 loop，而 loop fission 就是把一个 loop 给打破成多个 loop。
他俩不一定谁的 performance 会更好：一个 code size 小，一个 branch penalty 小；所以他俩并不能说这是一种 optimization，只能说这是一种 transformation，使用场景要具体问题具体分析。
下面同样用 C code 来举例：

.. code-block:: C

    // fused loop
    int i, a[100], b[100];
    for (i = 0; i < 100; i++) {
        a[i] = 1;
        b[i] = 2;
    }

    // ↑ to ↓: loop fission
    // ↓ to ↑: loop fusion

    // fissioned loop
    int i, a[100], b[100];
    for (i = 0; i < 100; i++) {
        a[i] = 1;
    }
    for (i = 0; i < 100; i++) {
        b[i] = 2;
    }

Loop Perforation
--------
Loop perforation 是通过跳过某个或几个 loop 或者跳过一个 loop 内的一些 branch，来提升程序 performance 的技术。
当然跳过一些 loop 会使结果不那么精确，所以这其实也是 approximate computing 的一部分知识，最开始被发表在 FSE’11 [#ref2]_ 上。
跳过一个或者几个 loop 很好理解，就是直接把一个 loop 给 drop 掉直接不运行了。
跳过一个 loop 内的某个 branch 可以理解成：以前循环条件是 ``i++``，现在变成 ``i+=2`` 了；所以在这里其实就 perforated rate 的概念，即 drop 的比率是多少。
更大的 drop rate 可以带来更多的 performance optimization，当然也会带来更大的 distortion。
CS:6120@Cornell 的同学曾经 implement 过这个东西，有兴趣的话可以看看这两个 links [#ref3]_ [#ref4]_。

Loop Unrolling vs Loop Vectorization
--------
他们两个听起来很像，都是把原有的 loop 给拆开，在一次循环里执行多个操作，降低 branch penalty，进而提升程序的 performance。但是他们的设计理念是不同的：

1. Loop unrolling 是一种 **instruction-level** 的操作，把原来原来多个循环能完成的事放在一个循环里的多个 instruction 里。它通过增加 code size，降低了 branch penalty，进而提升 performance。
2. Loop vectorization是一种 **data-level** 的操作，用 vector 类型在一个循环内一次处理多个数据。它通过 SIMD instructions (i.e. vectorized operations) 提升了memory bandwidth utilization，降低了 branch penalty，进而提升 performance。

所以很多时候他们其实是可以共同存在并提升程序性能的。

References
--------
.. [#ref1] Loop Autovectorization in CS:6120@Cornell: https://www.cs.cornell.edu/courses/cs6120/2019fa/blog/llvm-autovec/
.. [#ref2] ESEC/FSE'2011 Managing performance vs. accuracy trade-offs with loop perforation
.. [#ref3] Loop Perforation in CS:6120@Cornell: https://www.cs.cornell.edu/courses/cs6120/2019fa/blog/loop-perforation/
.. [#ref4] Loop Perforation LLVM Pass: https://github.com/avanhatt/llvm-loop-perforation/blob/master/loop-perf/LoopPerforation.cpp
