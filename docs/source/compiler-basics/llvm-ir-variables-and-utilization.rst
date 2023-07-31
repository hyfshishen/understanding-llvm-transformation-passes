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
        他们的lifetime是entire function。

    


Accessing Address-taken Variables
--------
Address-taken variables 包括 global variables 和 local variables 里的 Stack/Alloca instructions。
需要使用 ``alloca``，  ``store``，和 ``load`` 这三个 instructions 去 access 他们。

Local Variables Naming Rules
--------
