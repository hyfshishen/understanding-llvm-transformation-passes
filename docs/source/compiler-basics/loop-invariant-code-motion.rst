Loop Invariant Code Motion
=====


Induction Variables in Loop
--------
Induction variable 是在每个 iteration 中都增加或者减少固定的 value 的 variable（打个比方，判定循环次数的变量 ``i``, ``j``, ``k``）。
因为他们在不同 iterations 中的变化是可推理归纳的，所以就叫做 induction variable。
此外，由此 variable 进行线性变换，即 ``y=kx+b``，生成的 variable 也是 induction variables。

文字叙述不太直观。下面有一段代码样例：

.. code-block:: C

    // loop1
    for (i = 0; i < 10; ++i) {
        j = 17 * i;
    }

    // loop2
    int sum = 0;
    for (i = 1000; i > 10; i-=10) {
        j = 4 * i;
            sum += j;
    }

在 ``loop1`` 和 ``loop2`` 中，判定循环条件的 ``i`` 就是 induction variable。
因为其在不同 iteration 中的增加和减少是固定的。
此外， ``j`` 和 ``sum`` 都由 ``i`` 线性变换计算而得，所以他们也是 induction variable。

Loop Invariant Code Motion
--------
Loop invariant code motion (LICM) 是 compiler optimization 中的一种常见的技术 [#ref1]_ [#ref2]_。
LICM 的原理其实很简单，就是把 loop 中不跟随 iteration 改变的计算给放到 loop 的外面。
打个比方，下面有一个程序:

.. code-block:: C

    // original loop
    for (int i = 0; i < n; i++) {
        x = y + z;
        a[i] = 6 * i + x * x;
    }

    // loop after loop-invariant code motion
    x = y + z;
    t1 = x * x;
    for (int i = 0; i < n; i++) {
        a[i] = 6 * i + t1;
    }

很明显， ``x = y + z`` 和 loop 的 induction variables (``i`` 以及 ``a[i]``) 没有关系。
那就把它放在 loop 外面就好了。
这样 loop 内每次执行的时候就可以减少一些不必要的操作，进而提升程序的 runtime performance。
不过话说回来，一个优秀的 programmer 应该不会在代码里写出来这种低效地逻辑的，人不能总是想着用 compiler 兜底，对吧。

References
--------
.. [#ref1] Loop invariant code motion: https://en.wikipedia.org/wiki/Loop-invariant_code_motion
.. [#ref2] LICM in CS:6120@Cornell: https://www.cs.cornell.edu/courses/cs6120/2019fa/blog/loop-reduction/