``-loop-unroll-and-jam``: Unroll and Jam Loops
=====

在了解这个 pass 之前需要学习的背景知识是 :doc:`../compiler-basics/loop-transformation`。

Description
--------

``-loop-unroll-and-jam`` 是指对 nested loop 中 outer loop 给 unroll 的时候，同时对 inner loop 做一下 jam（i.e. fusion）。
这么做可以让代码更简洁，降低 branch penalty，进而提升 performance。这个听起来很抽象，不过下面看个例子其实就好了。

Code Example
--------

给定一段原始的 C code。

.. code-block:: C

    for(int i=0; i<100; i++)
        for(int j=0; j<100; j++)
            code_body(i, j);

``-loop-unroll`` 之后，可以 transform 成如下的 C code。

.. code-block:: C

    for(int i=0; i<100; i+=4){
        for(int j=0; j<100; j++)
            code_body(i, j);
        for(int j=0; j<100; j++)
            code_body(i+1, j);
        for(int j=0; j<100; j++)
            code_body(i+2, j);
        for(int j=0; j<100; j++)
            code_body(i+3, j);
    }

可以看到有点 redundant 对吧，我们看看 ``-loop-unroll-and-jam`` 会变成啥。

.. code-block:: C

    for(int i=0; i<100; i+=4)
        for(int j=0; j<100; j++){
            code_body(i, j);
            code_body(i+1, j);
            code_body(i+2, j);
            code_body(i+3, j);
        }

确实更简洁了。
