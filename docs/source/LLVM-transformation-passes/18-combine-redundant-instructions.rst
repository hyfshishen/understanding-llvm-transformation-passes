``-instcombine``: Combine Redundant Instructions
=====

Description
--------

``-instcombine`` 做的 transformation 是将原有的 code 的 instructions 给 combine 成 fewer and simpler instructions，降低 code size 和 computation 的同时提升 runtime performance。
这个 pass 不修改任何的 program control-flow，所以 basic block 的跳转不会发生改变。
``-instcombine`` focus 的主要是 numerical (specifically algebraic) instructions。
此外， ``-instcombine`` 对于 transformated code 有如下的 guarantees：

1. 如果一个 binary operator (including arithmetic and bitwise) 有一个 operand 是 constant，那么这个 constant 一定在右边 (只有 ``%y= add i32 %x, 3`` 一定没有 ``%y = add 3, i32 %32`` )。
2. 如果多个 bitwise operator 有 constant operand，那么他们总会被 group 起来，而且顺序分别是：first ``shift``, and ``or`` , and ``and`` , finally ``xor`` .
3. 对于所有 ``cmp`` 的 instructions (including ``<`` ,  ``>`` ,  ``≤`` , or  ``≥`` ) 总是会被转换成 ``=`` or ``≠`` (在语法允许的情况下).
4. 对于所有 ``cmp`` 的 instructions，如果其 operand 是 boolean 的，则其会被替换成 logical operations。
5. 对于两个 operand 是相同的 variable (i.e. virtual registers) 的情况，这里可以用 constant operand 来优化。比如 ``add X X`` → ``mul X, 2`` 或者 ``shl X, 1``.
6. 对于 multiplication/division 的对象是以 2 为底的指数，都可以转化成 shift operation（这个其实我在 CUDA 里其实经常用，比如 ``div i32 %x, 32`` → ``shl i32 %3, 5`` ）。
7. 等等。其实都是 runtime 到 compile-time 的优化。

Code Example
--------

例子太多了，这里举一个很简单的例子（其实在上面的 guarantees 已经讲了很多例子了）。

原始的 IR code。

.. code-block:: llvm

    %Y = add i32 %X, 1
    %Z = add i32 %Y, 1

``-instcombine`` transform 后的 IR code。

.. code-block:: llvm

    %Z = add i32 %X, 2 ; merging two arithmetic instructions