``-globalopt``: Global Variable Optimizer
=====

Description
--------

``-globalopt`` 和 ``-globaldce`` 一样，都是针对 global variable 进行优化的 transformation pass。
不过 ``-globalopt`` 比 ``-globaldec`` 更全面一些（也没直接删除那么 aggressive）。
``-globalopt`` 分析并识别出从来没有被读取过地址的 global variable，然后对其优化，比如把 read-only 的 global variable 变成 constant，删除没有 load 的只有 store 的 global variable（后者其实就是 ``-globaldec`` 在做的事），等等。

Code Example
--------

其实我感觉这个例子不是特别直观，不过 anyway，差不多有意义就可以了。之后记得 global variable 相关的优化可以用这个 pass 走一遍就行了。

原始的 IR code。

.. code-block:: llvm

    @gVar1 = global i32 10, align 4   ; Global variable with initial value 10

    define i32 @example() {
        %result = load i32, i32* @gVar1
        ret i32 %result
    }

    define void @example2(i32 %value) {
        store i32 %value, i32* @gVar1
        ret void
    }

``-globalopt`` transform 的 IR code。

.. code-block:: llvm

    @gVar1 = global i32 10, align 4 readnone   ; Read-only global variable

    define i32 @example() {
        %result = load i32, i32* @gVar1
        ret i32 %result
    }

    define void @example2(i32 %value) {
        store i32 %value, i32* @gVar1 writeonly
        ret void
    }
