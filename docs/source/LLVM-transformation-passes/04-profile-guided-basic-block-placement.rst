``-block-placement``: Profile Guided Basic Block Placement
=====

Description
--------

``-block-placement`` pass 主要是通过对一个 function 内的所有 basic block 进行重排列，其目的是提升 cache utilization 和 branch mis-prediction，进而提升程序的 runtime performance。
重排列这里主要指的是，把 function 内的最经常执行（frequently executed）的 basic blocks 都放在函数的最开始。
所以其实这个 pass 的功能真的很简单，就是调整一下bb的位置。

那么，如何知道哪些 basic blocks 是 frequently executed 的呢？这个 profile 一下就行了。
如果 profile 也得不到结果的话，那么就按 depth-first order 来排列。
Depth-first order 指的是，如果一个 basic block 是直接就可以被执行到的，那么它的 order 就比较靠前；如果一个 basic block 是通过 for-loop 或者 if-else-condition 才能执行到的话，那么它的 order 就比较靠后。
所以我们可以知道 order 越靠前被执行的概率越大（如果 order 最小的话基本必执行）。

Code Example
--------

这是原始的 LLVM IR。

.. code-block:: llvm

    define void @foo(i32 %a) {
    entry:
        %cmp = icmp eq i32 %a, 0
        br i1 %cmp, label %iftrue, label %iffalse

    iftrue:
        ; Code for the true branch
        br label %end

    iffalse:
        ; Code for the false branch
        br label %end

    end:
        ret void
    }

这是执行过 ``-block-placement`` 之后的 IR。假定我们已经 profile 过程序了， ``iffalse`` 是更容易执行的。

.. code-block:: llvm
    :emphasize-lines: 6,7,8

    define void @foo(i32 %a) {
    entry:
        %cmp = icmp eq i32 %a, 0
        br i1 %cmp, label %iffalse, label %iftrue

    iffalse:
        ; Code for the false branch
        br label %end

    iftrue:
        ; Code for the true branch
        br label %end

    end:
        ret void
    }

可见，basic block 的顺序被调整了 -- ``iffalse`` 和 ``iftrue`` 的顺序被调整了。
让更容易被执行的放在程序之前，这样就不需要额外的跳转了。
