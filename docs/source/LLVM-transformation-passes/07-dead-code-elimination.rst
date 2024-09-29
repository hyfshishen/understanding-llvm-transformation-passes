``-dce``: Dead Code Elimination
=====

Description
--------

``-dce`` 的全称如上所示，其实就是 less aggressive 版本的 ``-adce``。
从 high-level 上来说， ``-adce`` 可以把那些 conditionally executed but never actually executed at runtime 的 instructions 给 remove 掉；
而 ``-dce`` 如果是理论上可以被 conditionaly executed 那么就不 remove 了。
这个 conditional 听起来很玄学，不过我觉得可以这么理解：其实就是 ``-adce`` 的 pass 更复杂，可以从逻辑上推理程序的 control-flow，然后进行一些 remove 操作。
当然，如果没有 detect 到类似的代码段，那么 ``-adce`` 也就 degrade 成 ``-dce`` 了。

Code Example
--------

因为 ``-adce`` 那里已经给过 code elimination 的 example 了，那么这里主要是给一个 ``-dce`` 和 ``-adce`` 不同的 example（i.e. ``-adce`` 可以 remove code 但是 ``-dce`` 不行）。
下面有三个简单的 C 语言的代码段（C 比较好理解，当然所有的 pass 都是在 IR 上执行的）。

原始的 C code。

.. code-block:: C

    int x = 10;
    int y;

    if(x+1) y=1;
    else y=0;

``-dce`` transform 的 C code。

.. code-block:: C

    int x = 10;
    int y;

    if(x+1) y=1;
    else y=0;

``-adce`` transform 的 C code。

.. code-block:: C
    int x = 10;
    int y;

    y=1;

这里这个 controlf-low 其实是理论上可以执行到下一段的，但是在当前的常量赋值下一定不会执行 -- 因为 10+1 一定会被判定为 ``true``。
``-adce`` 可以推理出来这个情况，然后去 remove 掉对应的 unneccessary control-flow； ``-dce`` 没有推理，所以就不改变这里程序的逻辑了（程序没变）。

