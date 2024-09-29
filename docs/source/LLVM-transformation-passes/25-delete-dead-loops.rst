``-loop-deletion``: Delete Dead Loops
=====

Description
--------

``-loop-deletion`` pass 做的事非常简单，其实就是对于一个 loop，如果其计算对于 function 的 return value 没有任何影响，那么这个 loop 就会被判定为 dead 的，然后 ``-loop-deletion`` 就会将其删去（逻辑很类似 ``-dce`` pass 之于 code）。

Code Example
--------

原始的 IR code。

.. code-block:: llvm

    define i32 @foo(i32 %n) {
    loop:
        %cmp = icmp slt i32 %n, 10
        br i1 %cmp, label %body, label %exit

    body:
        %add = add i32 %n, 1
        store i32 %add, i32* %n
        br label %loop

    exit:
        ret i32 %n
    }

``-loop-deletion`` transform 过的 IR code。

.. code-block:: llvm

    define i32 @foo(i32 %n) {
        ret i32 %n
    }

对于一开始的的 IR code，虽然 loop 里天花乱坠地计算了一大堆，但是它们对于该 function 的 return value ``%n`` 没有任何影响，那么就直接将其判定为 dead，然后删去（如上code所示）。
代码就很自然地简洁多了。
