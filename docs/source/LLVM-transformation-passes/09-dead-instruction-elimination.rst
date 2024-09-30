``-die``: Dead Instruction Elimination
=====

Description
--------

``-die`` 的名字全称如标题所示，和 ``-dce`` 和 ``-adce`` passes 的功能非常类似，主要是进行 eliminating dead instruction 的 code transformation。
听起来好像没啥区别，细致还是有点不同的。
为了理解，这里首先明确一个概念：在 compiler optimization 中， dead instruction 和 dead code 是两个不同维度的操作：

- **Dead instruction** : 从 instruction level 去思考，这个 instruction 执行不执行是没两样的。打个比方，我定义了一个变量然后给他赋值为 1 然后又不使用他。这些就是 dead instruction。
- **Dead code** : 从 program 本身去思考，这段 code 执行不执行是没两样的。打个比方，我定义了一个 function 但是没调用；亦或者是我定义了一个变量，这个变量虽然和别的变量有关会进行运算，但是和 function 的 return value 没关系。这些就是 dead code。

所以可以看出来， ``-dce`` 和 ``-die`` 很像，但是 ``-die`` 的作用更 conservative 一些。

Code Example
--------

上面的描述有点抽象，但是来个例子就很好理解了。
下面有两段 code blocks：第一个代表了可以被 ``-dce`` remove 但是不能被 ``-die`` remove 的例子；第二个代表了 ``-die`` 可以 remove 的例子（当然， ``-dce`` 这会也是可以 remove 的）。

我们从第一个例子开始。

.. code-block:: llvm
    :emphasize-lines: 4,5

    ; Original IR code
    define i32 @foo(i32 %a, i32 %b) {
        %sum1 = add i32 %a, %b
        %prod = mul i32 %a, %b
        %sum2 = add i32 %prod, %a
        ret i32 %sum1
    }

    ; IR code after -dce transformation
    define i32 @foo(i32 %a, i32 %b) {
        %sum = add i32 %a, %b
        ret i32 %sum
    }

第上述例子中， ``%prod`` 和 function 最后的返回值 ``%sum`` 无关，所以可以被 ``-dce`` remove 掉，因为他从 code 的角度上来看已经 dead 了。
但是从 instruction 的角度来看， ``%prod`` 这一个 instruction 并不是 dead 的，因为它还是调用了 ``%a`` 和 ``%b`` 这两个变量并且同时被变量 ``%sum2`` 所调用。

现在我们来看第二个例子。

.. code-block:: llvm
    :emphasize-lines: 3

    ; Original IR code
    define i32 @bar(i32 %x) {
        %a = add i32 %x, 0
        %b = mul i32 %a, 1
        %c = add i32 %a, %b
        ret i32 %c
    }

    ; IR code after -die/-dce transformation
    define i32 @bar(i32 %x) {
        %b = mul i32 %x, 1
        %c = add i32 %x, %b
        ret i32 %c
    }

在第二个例子中， ``%a`` 从 instruction 的角度来看已经是 dead 的了，因为只是加减常量，其作用可以被 ``%x`` 和 ``%b`` 完全替代，所以可以被 ``-die`` remove掉。
当然也可以被 ``-dce`` remove掉。
这个例子不太准确，也有点像 value numbering 的东西，不过问题不大，能理解到 ``-dce`` 比 ``-die`` 更智能就行了。

话说我在写完这个文档之后，再整理在 ReadTheDoc 上已经一年了。我现在回头看看这个区别，实在是太扯了。
在第一个例子里， 做一次 ``-die`` pass 虽然不能 remove ``%prod``，但是可以 remove ``%sum2``，那这会 ``%prod`` 不就成为了 newly dead 的 instruction 了么，再来一次 ``-die`` 就可以被 remove 了。
所以现在自己回头想想，强行理解他们的不同其实意义不大，更多的可能是 collaboration 或者 implementation 的时候的历史遗留问题。
只要能理解这种思想，并且写出更高效的 code 其实就够了。
