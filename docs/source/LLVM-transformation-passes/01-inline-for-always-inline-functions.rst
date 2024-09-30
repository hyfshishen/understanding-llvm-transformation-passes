``-always-inline``: Inliner for always_inline Functions
=====

Description
--------

``-always-inline`` pass 主要是进行把所有标记有 ``__always_inline`` 的 function 都在 IR level 上 inline 的一种 code transformation。
这样，只有当用户执行 ``opt -always-inline -S input.ll -o output.ll`` 的时候，这些有标记的 function 才会被 inline。
相比于 ``inline`` 这个function attribute 更灵活一些（因为它会在 compile 过程中被强行 inline）。

**Inline** 的意思就是把 callee function 中的每个 instruction 都给变成 caller function 中的 instruction，这样就解决了 function call 和 value return 的 overhead。
这里再解释一下 caller-callee 的关系，打个比方我的 ``main()`` function 调用了 ``vector_add()`` function；这里调用别人的 ``main()`` 就是 caller，另一个则是 callee。

Code Example
--------

给定一段 C code。

.. code-block:: C

    __always_inline int add(int a, int b) {
        return a + b;
    }

    int main() {
        int x = 5, y = 10;
        int z = add(x, y);
        return 0;
    }

我们先把它变成 LLVM IR。

.. code-block:: llvm
    :emphasize-lines: 1,2,3,4,13

    define i32 @add(i32 %a, i32 %b) #0 {
        %1 = add i32 %a, %b
        ret i32 %1
    }

    define i32 @main() #0 {
        %1 = alloca i32, align 4
        %2 = alloca i32, align 4
        store i32 5, i32* %1, align 4
        store i32 10, i32* %2, align 4
        %3 = load i32, i32* %1, align 4
        %4 = load i32, i32* %2, align 4
        %5 = call i32 @add(i32 %3, i32 %4)
        ret i32 0
    }  

经过 ``-always-inline`` 之后，这个 IR 变成了。

.. code-block:: llvm
    :emphasize-lines: 8

    define i32 @main() #0 {
        %1 = alloca i32, align 4
        %2 = alloca i32, align 4
        store i32 5, i32* %1, align 4
        store i32 10, i32* %2, align 4
        %3 = load i32, i32* %1, align 4
        %4 = load i32, i32* %2, align 4
        %5 = add i32 %3, %4
        ret i32 0
    }