SSA Form in LLVM IR
=====

SSA Form
--------
Static single assignment (SSA) 指 compiler 的 intermediate representation 设计中的一个原则，即所有 variable 只可被赋值一次。
这种设计虽然相比于源码增加了复杂度（其实本来复杂度也应该就会被增加），但是允许了各种复杂优化的实现（比如各种 dead code elimination, data-flow analysis, value numbering, phi-code placement, aggressive code motion，或者是 alias analysis）。
SSA 是 LLVM IR 以及 compiler 中间表示中非常重要的知识点。
这里再用一个代码样例 [#ref1]_ 来深化一下对 SSA form 的理解。
我们首先给出这段代码的 C code，意思非常简单，就是比较两个数的大小:

.. code-block:: C

    int max(int a, int b) {
        if (a > b) {
            return a;
        } else {
            return b;
        }
    }

我们现在将上面的的 C code compile 成 LLVM IR，如下所示。
注意，这是一个 valid 但是不那么 efficient 的 IR。

.. code-block:: llvm

    define i32 @max(i32 %a, i32 %b) {
    entry:
        %retval = alloca i32, align 4
        %0 = icmp sgt i32 %a, %b
        br i1 %0, label %btrue, label %bfalse

    btrue:                                      ; preds = %2
        store i32 %a, i32* %retval, align 4
        br label %end

    bfalse:                                     ; preds = %2
        store i32 %b, i32* %retval, align 4
        br label %end

    end:                                     ; preds = %btrue, %bfalse
        %1 = load i32, i32* %retval, align 4
        ret i32 %1
    }

读完上面的的代码之后，你可能会有点疑问。
一个小小的代码怎么需要 access 那么多次 memory（又是 ``store`` 又是 ``load``），这个效率也太低了。
而且还要多设计一个变量 ``%1`` 来存 ``%retrval`` 的值，太冗余了。
我能不能按照如下思路设计 IR code？这样感觉会更简洁一些。

.. code-block:: llvm

    define i32 @max(i32 %a, i32 %b) {
    entry:
        %0 = icmp sgt i32 %a, %b
        br i1 %0, label %btrue, label %bfalse

    btrue:
        %retval = %a
        br label %end

    bfalse:
        %retval = %b
        br label %end

    end:
        ret i32 %retval
    }

答案： **不行**！因为这里违反了 LLVM IR 的 SSA form: ``%retval`` 在两个 branch 被赋值了两次！
如果这样设计的话其他更 fancy 的 optimization 就不能用了，简直得不偿失。

``phi`` Instruction
--------
那怎么样才能不违反 SSA form 的同时避免这么多没意义的 memory access 呢？
在 LLVM IR 中，答案是神奇的 ``phi`` instruction。这个instruction的用法如下所示:

.. code-block:: llvm

    %val = phi type [%val1, %prev_bb1], [%val2, %prev_bb2]

如果当前执行的 basic block 的上一个是 ``%prev_bb1``，那么 ``%val`` 就赋值为 ``%val1``；反之为 ``%val2``。
通过这种设计，上面的代码避免了额外的 ``store`` 还有 ``alloca`` instructions，而且还避免了额外的 variable 的定义，也不违反SSA的设计原则，一举多得。
下面是使用了 ``phi`` 的 ``@max function`` 的实现：

.. code-block:: llvm

    define i32 @max(i32 %a, i32 %b) {
    entry:
        %0 = icmp sgt i32 %a, %b
        br i1 %0, label %btrue, label %bfalse

    btrue:                                      ; preds = %2
        br label %end

    bfalse:                                     ; preds = %2
        br label %end

    end:                                     ; preds = %btrue, %bfalse
        %retval = phi i32 [%a, %btrue], [%b, %bfalse]
        ret i32 %retval
    }

``phi`` in Machine Code
--------
我们其实还想知道一下真正的 machine code 是怎么做这件事的。
我们首先用 ``llc -O0 -filetype=asm`` 把上面的 LLVM IR 变成 X86 的 machine code：

.. code-block:: llvm

    max:                                    # @max
    # %bb.0:                                # %entry
        cmpl    %esi, %edi                  # %edi = %a, %esi = %b
        jle     .LBB0_2
    # %bb.1:                                # %btrue
        movl    %edi, -4(%rsp)              # mov src, dst
        jmp     .LBB0_3
    .LBB0_2:                                # %bfalse
        movl    %esi, -4(%rsp)              # mov src, dst
        jmp     .LBB0_3
    .LBB0_3:                                # %end
        movl    -4(%rsp), %eax              # return value in eax
        retq

我们再用 ``llc -O0 -filetype=asm`` 提升一下 optimization level 试一试，machine code 如下所示：

.. code-block:: llvm

    max:                                    # @max
    # %bb.0:                                # %entry
        cmpl    %esi, %edi
        jg      .LBB0_2
    # %bb.1:                                # %bfalse
        movl    %esi, %edi
    .LBB0_2:                                # %end
        movl    %edi, %eax
        retq

可以看到，因为有 ``move`` 和 ``jump`` instructions 的存在，这个过程比我们的想象更简洁一些。
哈哈，看来还是 machine code 更聪明。


References
--------
.. [#ref1] Single-Static Assignment Form and PHI: https://mapping-high-level-constructs-to-llvm-ir.readthedocs.io/en/latest/control-structures/ssa-phi.html