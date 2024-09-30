``-sroa``: Scalar Replacement of Aggregates
=====

Description
--------

``-sroa`` pass 尝试将复杂变量（比如 array, struct）的 ``alloca`` 给变成一系列 scalar 变量（比如 ``double`` , ``int`` , ``char`` ）的 ``alloca`` 。
如果有可能的话，还会帮助优化成更 clean 的 SSA form。这些都可以帮助 code 变得更 clean，也可以提升其他 pass 的优化效率（比如 ``-sroa`` 和 ``-mem2reg`` 一起食用效果会更好）。

``alloca`` struct 听起来很怪，不过这个在 LLVM IR 上有很多应用，这里举个例子比如 LLVM 3.4 compile 的 `XSBench <https://github.com/ANL-CESAR/XSBench>`_ 程序里， ``%low = alloca %struct.NuclideGridPoint*, align 8`` 。
这也是 ``-sora`` target transformation 的地方。

Code Example
--------

这里为了便捷只给 Code after ``-sora`` transformation 的例子，我们可以看到 ``Person`` 这个 struct 的 initialization 变成了两个 scalar（ ``%i`` 和 ``%sum_age`` ）的 ``alloca``。

.. code-block:: llvm
    :emphasize-lines: 4,5

    %struct.Person = type { i32, double }

    define double @calculate_average(%struct.Person* %people, i32 %num_people) {
    entry:
        %sum_age = alloca double
        %i = alloca i32
        store double 0.0, double* %sum_age
        store i32 0, i32* %i

    for.cond:                             ; Loop condition
        %1 = load i32, i32* %i
        %2 = icmp slt i32 %1, %num_people
        br i1 %2, label %for.body, label %for.end

    for.body:                             ; Loop body
        %3 = load double, double* %sum_age
        %4 = load i32, i32* %i
        %person_ptr = getelementptr %struct.Person, %struct.Person* %people, i32 %4
        %age_ptr = getelementptr i32, i32* %person_ptr, i32 0       ; Access the age directly
        %age = load i32, i32* %age_ptr
        %age_double = sitofp i32 %age to double
        %5 = fadd double %3, %age_double
        store double %5, double* %sum_age
        %6 = add i32 %4, 1
        store i32 %6, i32* %i
        br label %for.cond

    for.end:                              ; Loop exit
        %7 = load double, double* %sum_age
        %average = fdiv double %7, double %num_people
        ret double %average
    }
