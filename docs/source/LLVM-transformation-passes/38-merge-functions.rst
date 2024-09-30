``-mergefunc``: Merge Functions
=====

Description
--------

``-mergefunc`` pass 尝试 detect functionality 上相同的两个 functions，然后将他们 merge 起来。
进而可以降低 code size 和 control-flow graph 的 complexity，进而提升 runtime performance。
它的做法会分为三步：

1. 把 functions 分成 smaller components。
2. 比较两个 function 所分别分成的 smaller components 的个数，如果个数一样的话进行下一步，否则直接判定不同。
3. 遍历每个 smaller component 并且一个一个比较，如果最后判定 functionality 一样的话那么就准备开始 merge，否则判定不同。

这其实听起来听魔幻的，"我怎么可能会写出两个一摸一样的 function 呢？这岂不是质疑我的智商？" 其实很多时候真的是有可能的，打个比方：

- 在一个只支持 64-bit 作为 minimal unit 的 machine，第一个 function 上开一个 64-integer 的 space，第二个 function 开一个 minimal unit 的 space；这两个 function 其实是一样的。
- 两个 function 都以一个 integer 作为 argument，第一个 function 把这个 integer 乘2，第二个 function 把这个 integer 里所有的 bit 给 left shift 1；这两个 function 其实也是一样的。

总而言之，这个 pass 所面对的情况非常复杂，LLVM 官方也对此写了非常详细的文档解释，可以查看 `LLVM Merge Functions <https://llvm.org/docs/MergeFunctions.html>`_

Code Example
--------

原始的 IR code。

.. code-block:: llvm

    define i32 @add_i32(i32 %a, i32 %b) {
        %1 = add i32 %a, %b
        ret i32 %1
    }

    define i64 @add_i64(i64 %a, i64 %b) {
        %1 = add i64 %a, %b
        ret i64 %1
    }

``-mergefunc`` transform 之后的 IR code。

.. code-block:: llvm

    define { i64, i1 } @add(i64 %a, i64 %b, i1 %is_i32) {
        %1 = add i64 %a, %b
        %2 = trunc i64 %1 to i32
        %3 = select i1 %is_i32, i32 %2, i64 %1
        ret { i64, i1 } %3
    }

这里 ``%is_i32`` 是一个 boolean，验证当前的 input arguments 是不是 ``i32`` type 的，如果 true 的话在 return 之前 truncate 成 ``i32``，否则就 return ``i64`` type 的版本。
不得不感叹 LLVM 实在是太智能了，这个 merge 真是高级。
