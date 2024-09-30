``-function-attrs``: Deduce Function Attributes
=====

Description
--------

``-function-attrs`` 通过分析 function，然后得出哪些 attribute 可以被用来帮助 declare 和 define 这个 function。
Attribute 提供了额外的信息，这可以帮助来定义 function 的 behavior（比如 ``inline`` , ``noreturn`` , ``readonly`` , ``argmemonly`` 等等）。
这个 pass 没办法一下子就提升 code performance，但是通过这些额外的信息，可以让 LLVM pass 理解这个function，进而再其他的优化中更加提升它的 performance。

这里补充点 LLVM IR 的语法知识，一般来说 function 的 attribute 都在它的 declare 的上面一行。
下面的 code block 是 NPB suite 的 EP 程序在 LLVM 3.4 下 compile 的 IR。

.. code-block:: llvm

    ; Function Attrs: nounwind
    declare i32 @sprintf(i8*, i8*, ...) #3

    ; Function Attrs: nounwind
    declare double @pow(double, double) #3

我们可以看到这两个 function 定义的上面一行就是其对应的 attribute。
总而言之， ``-function-attrs`` pass 也是自动化分析 function 然后给出更详细的 attributes 信息。

Code Example
--------

下面是一个 function ，而且这个 function 里没有 throw exception，所以通过 ``-function-attrs`` 这个 pass 的分析之后，未来应该会推断出 ``nounwind`` 这个attribute（然后写在其对应的declaration的地方）。

``nounwind`` 的意思包含：(1) not throwing exceptions and (2) not performing stack unwinding.

.. code-block:: llvm
    :emphasize-lines: 1

    ; Function Attrs: nounwind
    define void @example(i32* %ptr) {
        %val = load i32, i32* %ptr
        call void @someFunction()
        store i32 %val, i32* %ptr
        ret void
    }

