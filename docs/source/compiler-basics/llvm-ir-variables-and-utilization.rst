LLVM IR Variables and Utilization
=====

LLVM IR Variables
--------
LLVM IR 里有两种 variables，分别是 global variables 和 local variables，下面介绍一下。

- **Global Variable:**
    
    LLVM IR 的 global variable 的格式是 ``@XXX``，比如 ``@global_variable``，它的 lifetime 是 entire program code。
    
- **Local Variables**
    
    LLVM IR 的 local variable 的格式是 ``%XXX``，比如 ``%local_variable`` 或者 ``%1``，它的 lifetime 当然只是 local 的，不过要分情况讨论：
    
    - **Registers:**
        
        大量的 local variables 最后都会变成 virtual registers（不完全是 machine 层面的 registers，but close to）。
        Register 代表了一个 value，而不是一个 address，所以不能被 modified。
        这些 virtual register 必须要遵守 SSA 的原则，所以只能被 assign value 一次。
        他们的 lifetime 通常非常 limited，有可能 across 几个 basic block 就不能用了。
        一般而言，能稳定 across basic-block 访问的 virtual register 只有 ``phi`` instruction 中的 virtual registers。
        这里 SSA 和 ``phi`` instruction 的详细解释可以见 :doc:`ssa-form-in-llvm-ir`。
        
    - **Stack/Alloca instructions:**
        
        这类 local variables 是 address-taken 的，其内存空间由 function 开始的 ``alloca`` instruction 赋予。
        他们指向一个 memory on the stack，并且访问的时候类似 global variables 都需要 ``store`` 和 ``load`` instructions。
        他们的 lifetime 是 entire function。

下面给一个代码例子来展示一下这些 variables，例子来源于由 LLVM 3.4 compile 出来的 Argonne Xsbench [#ref1]_ 的 IR：

.. code-block:: llvm

    @rn_v.seed = internal global i64 1337, align 8
    @.str60 = private unnamed_addr constant [12 x i8] c"XS_data.dat\00", align 1
    @.str161 = private unnamed_addr constant [3 x i8] c"wb\00", align 1
    @.str262 = private unnamed_addr constant [3 x i8] c"rb\00", align 1

    ; Function Attrs: nounwind uwtable
    define void @calculate_micro_xs(double %p_energy, i32 %nuc, i64 %n_isotopes, i64 %n_gridpoints, %struct.GridPoint* noalias %energy_grid, %struct.NuclideGridPoint** noalias %nuclide_grids, i32 %idx, double* noalias %xs_vector) #0 {
        %1 = alloca double, align 8
        %2 = alloca i32, align 4
        %3 = alloca i64, align 8
        %4 = alloca i64, align 8
        %5 = alloca %struct.GridPoint*, align 8
        %6 = alloca %struct.NuclideGridPoint**, align 8
        %7 = alloca i32, align 4
        %8 = alloca double*, align 8
        %f = alloca double, align 8
        %low = alloca %struct.NuclideGridPoint*, align 8
        %high = alloca %struct.NuclideGridPoint*, align 8
        store double %p_energy, double* %1, align 8
        store i32 %nuc, i32* %2, align 4
        store i64 %n_isotopes, i64* %3, align 8
        store i64 %n_gridpoints, i64* %4, align 8
        store %struct.GridPoint* %energy_grid, %struct.GridPoint** %5, align 8
        store %struct.NuclideGridPoint** %nuclide_grids, %struct.NuclideGridPoint*** %6, align 8
        store i32 %idx, i32* %7, align 4
        store double* %xs_vector, double** %8, align 8
        %9 = load i32* %2, align 4
        %10 = sext i32 %9 to i64
        %11 = load i32* %7, align 4
        %12 = sext i32 %11 to i64
        %13 = load %struct.GridPoint** %5, align 8
        %14 = getelementptr inbounds %struct.GridPoint* %13, i64 %12
        %15 = getelementptr inbounds %struct.GridPoint* %14, i32 0, i32 1
        %16 = load i32** %15, align 8
        %17 = getelementptr inbounds i32* %16, i64 %10
        %18 = load i32* %17, align 4
        %19 = sext i32 %18 to i64
        %20 = load i64* %4, align 8
        %21 = sub nsw i64 %20, 1
        %22 = icmp eq i64 %19, %21
        br i1 %22, label %23, label %42

这里可以看到， ``%1`` 到 ``%high`` 都是 Stack/Alloca 的 local variables，他们都集中在 function 最开始的部分，需要 ``alloca`` instruction 开辟内存空间之后才可以使用；
``%9`` 到 ``%42`` 这些都是 register 类型的 local variables，可以直接被访问并参与计算，但是必须遵守 SSA form，即只可以被赋值一次；
最上面的 ``@rn_v.seed`` 到 ``@.str262``，显而易见，他们是 global variables。
你可能也会好奇，为什么有的 local variables 的名字有意义比如 ``%low``，有的却是 ``%1`` 类似的格式呢？
前者是和 source code 中的名字对应的，而后者是由于 SSA form 生成的，每个 function 都从 ``%1`` 开始挨个向下排。
可以看到，LLVM IR 为了增强可读性还是保留了很多 source code-level 的特征的，比如 variable names；
如果你不想让别人看懂你的 IR 的话，你也可以把这些 variable name 用 ``-strip`` pass 变成类似 ``%1`` 的形式，这个 pass 的解释可以在下一章看到。

Accessing Address-taken Variables
--------
Address-taken variables 包括 global variables 和 local variables 里的 Stack/Alloca instructions。
需要使用 ``alloca``，  ``store``，和 ``load`` 这三个 instructions 去 access 他们。
我们用一个如下的代码样例来解释一下。
首先给出这段代码的 C code：

.. code-block:: C

    int add(int a, int b) {
        int result = a + b;
        return result;
    }

下面是这段 C code compile 成的 LLVM IR：

.. code-block:: llvm

    define i32 @add(i32 %a, i32 %b) {
    entry:
        ; Allocate space on the stack for the local variable 'result'
        %result = alloca i32

        ; Perform the addition and store the result in 'result'
        %sum = add i32 %a, %b
        store i32 %sum, i32* %result

        ; Load the value from 'result' and return it
        %loaded_result = load i32, i32* %result
        ret i32 %loaded_result
    }

我们可以看到，使用这三个 instructions 在这个过程中的用法如下所示：

- ``alloca``：开辟一块 memory space for a variable。
- ``store``：操作完了之后把 register variable 放在某个 memory space。
- ``load``：将一个 memory space 内的 variable 放在某个 register variable 里。


References
--------
.. [#ref1] Argonne National Lab, XSBench: https://github.com/ANL-CESAR/XSBench
