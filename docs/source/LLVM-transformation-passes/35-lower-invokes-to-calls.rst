``-lowerinvoke``: Lower Invokes to Calls, for Unwindless Code Generators
=====

Description
--------

``-lowerinvoke`` pass 指对不支持 stack unwind 的 code generator，将所有的 ``invoke`` instruction 都降级成 ``call`` instruction。
这样虽然会导致原来的 exception handler 都变成 dead instructions/blocks，但是这些 dead instruction/blocks 也可以被之后用 ``-dce`` 或者 ``-simplifycfg`` 给 remove 掉。

上面可能听起来有点绕，这里介绍几个关键概念（LLVM-specific 的概念，和 compiler 的 general knowledge 关系不大，不过还是了解一下）：

1. Stack unwind 是 exception report 的一种技术，指如果程序 crash 了，从函数的 stack 中一层一层的 unwind，然后 find 并 report exception 的位置。详情可以看这个 `Link <https://www.geeksforgeeks.org/stack-unwinding-in-c/>`_。
2. ``call`` instruction: 碰到很多了，其实就是调用别的 function，跳转 control-flow。不过这个 function 都是 normal execution。用法如下所示：
    
    .. code-block:: llvm

        %result = call <return_type> <function_name>(<arg_types> <args>)

3. ``invoke`` instruction：同样地也是调用别的 function，跳转 control-flow。但是这里会增加额外的 exception handler，如果 function 正常运行完了，那么就会运行到 ``normal_label`` （一个 basic block ID） 里，否则运行到 ``exception_label`` （另一个 basic block ID）。

    .. code-block:: llvm

        %normal = invoke <return_type> <function_name>(<arg_types> <args>)
           to label <normal_label> unwind label <exception_label>

    我们再给一个 ``invoke`` instruction 在实际编译中的例子。代码段来自于 `Parboil benchmark suite <http://impact.crhc.illinois.edu/parboil/parboil.aspx>`_ 的 SGEMM.ll（LLVM 3.4 IR）

    .. code-block:: llvm

        ; <label>:26                                      ; preds = %80, %74, %71, %63, %56, %53, %50, %48, %45, %43, %41, %38, %36, %30, %0
        %27 = landingpad { i8*, i32 } personality i8* bitcast (i32 (...)* @__gxx_personality_v0 to i8*)
                cleanup
        %28 = extractvalue { i8*, i32 } %27, 0
        store i8* %28, i8** %6
        %29 = extractvalue { i8*, i32 } %27, 1
        store i32 %29, i32* %7
        invoke void @_ZNSt13basic_fstreamIcSt11char_traitsIcEED1Ev(%"class.std::basic_fstream"* %f)
                to label %88 unwind label %94

        ; <label>:30                                      ; preds = %24
        %31 = bitcast %"class.std::basic_fstream"* %f to i8*
        %32 = getelementptr inbounds i8* %31, i64 16
        %33 = bitcast i8* %32 to %"class.std::basic_ostream"*
        %34 = load i32* %3, align 4
        %35 = invoke %"class.std::basic_ostream"* @_ZNSolsEi(%"class.std::basic_ostream"* %33, i32 %34)
                to label %36 unwind label %26

通过理解，也可以看出来， ``call`` 确实是比 ``invoke`` 的代价更小的， ``-lowerinvoke`` pass 也可以理解成是 exception handler 和 runtime perfomrance 之间的 trade-off。

Code Example
--------

原始的 IR code。

.. code-block:: llvm

    declare void @callee()

    define void @caller() {
        invoke void @callee() to label %normal unwind label %exception

    normal:
        ret void

    exception:
        ; Exception handling code
        ret void
    }

``-lowerinvoke`` transform 之后的 IR code。

.. code-block:: llvm

    declare void @callee()

    define void @caller() {
        ; Transformed from invoke to call
        call void @callee()

    normal:
        ret void
    }

可以看到改动很好理解，就是 exception handler 全部都变成了 dead block（然后被 remove 了） 而已🥲。
