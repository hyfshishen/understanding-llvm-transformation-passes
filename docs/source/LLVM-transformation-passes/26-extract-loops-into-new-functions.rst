``-loop-extract``: Extract Loops into New Functions
=====

Description
--------

``-loop-extract`` pass 做的事是把一个 function 中的 top-level 的 loop 给 extract 出来，然后变成一个新的 function。
所谓 top-level 的 loop 指最外层的 loop。
如果这个 function 仅有这一个 loop 组成，这相当于已经被 extract 好了，所以就不再 touch。
这个 pass 看起来和 performance optimization 没什么关系，事实上也确实没什么关系。
这个 pass 其实就是在 debug phase 的 setting up bugpoints 的时候才有用。

Code Example
--------

这个例子用 LLVM IR 不太直观，所以我用一个 C 的小例子来解释一下。

原始的 C code。

.. code-block:: C

    int yafan(int yafan_paper_count) {
        for(int i=0; i<100; i++) // Most outter loop in function
            for(int j=0; j<100; j++)
                yafan_paper_count++;
        yafan_paper_count ++;
        return yafan_paper_count;
    }

``-loop-extract`` transform 过的 C code。

.. code-block:: C

    int yafan(int yafan_paper_count) {
        yafan_paper_count = shihui(yafan_paper_count);
        yafan_paper_count ++;
        return yafan_paper_count;
    }

    int shihui(int shihui_paper_count) { // extract this loop to a new function
        for(int i=0; i<100; i++)
            for(int j=0; j<100; j++)
                    shihui_paper_count++;
        return shihui_paper_count;
    }

可以看到其实就是把 function 里的 outer loop 给 extract 成一个新的 function 了，这个 transformation 甚至会让速度变慢很多（增加了无数个 function call）。
但是无所谓，反正是 bugpoint 才用的，不考虑 performance。
