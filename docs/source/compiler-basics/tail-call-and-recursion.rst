Tail Call and Recursion
=====

Tail Call
--------
Tail call 指 function 的最后一个操作是调用一个 function。
在 C code 中，tail call 表现为一个 function 的 return value 是调用另一个 function 并获得其 return value 的形式。
如下所示， ``yafan`` function 的 return value（i.e. 最后一个操作）是由调用 ``shihui`` function 所获得的。
这就是 tail call 的很典型的一种情况。

.. code-block:: C

    int yafan(int a, int b){
        return shihui(a) + shihui(b);
    }

Tail Recursion
--------
Tail recursion 是 tail call 的一种特殊形式，即该 function 最有一个操作所调用的 function 是自己（比如常见的 fibinaccio 数列的递归算法）。
当然这也是递归算法里非常常见的情况。
我们知道，递归算法常常会占用大量的堆栈空间，极大地降低程序的 performance，所以优化 tail recursion 也是一个很重要的任务。
下面以 resursive sum 为例，我们用一段 Python code 和其堆栈调用来描述一个简单的 tail recursion 的优化。

首先，我们看看完全没有任何优化的 tail recursion：

.. code-block:: python

    def recsum(x):
    if x == 1:
        return x
    else:
        return x + recsum(x - 1)

下面是在给定 function input 为 ``5``的时候，其堆栈调用的情况，很明显这个 overhead 太大了。

.. code-block:: console

    recsum(5)
    5 + recsum(4)
    5 + (4 + recsum(3))
    5 + (4 + (3 + recsum(2)))
    5 + (4 + (3 + (2 + recsum(1))))
    5 + (4 + (3 + (2 + 1)))
    5 + (4 + (3 + 3))
    5 + (4 + 6)
    5 + 10
    15

这里是简单优化过的 function：

.. code-block:: python

    def tailrecsum(x, running_total=0):
        if x == 0:
            return running_total
        else:
            return tailrecsum(x - 1, running_total + x)

我们查看一下其堆栈调用情况，可以看到明显降低了，每次都只调用一个 function 即可。

.. code-block:: console

    tailrecsum(5, 0) 
    tailrecsum(4, 5) 
    tailrecsum(3, 9)
    tailrecsum(2, 12) 
    tailrecsum(1, 14) 
    tailrecsum(0, 15) 
    15

当然，直接优化成一个循环是最简洁的，这里不再赘述。更多优化的信息可以查看这里 [#ref1]_。

References
--------
.. [#ref1] Tail Call: https://en.wikipedia.org/wiki/Tail_call
