``-internalize``: Internalize Global Symbols
=====

Description
--------

``-internalize`` 首先会 loop 所有当前 module 里的 function，找到 main function 之外，把其他 function 和只有在他们内部才能使用的 global variables 变成 internal 的。
在我们写代码的过程中，会定义很多 global variables，但是事实上我们只是想对于一些比较大的 code region（并不是真正意义的 entire program）来 cross 的访问。
所以这个时候完全意义的 global 就有点多余了，这也是 ``-interalize`` 这个 pass 所需要优化的情况。

在实际过程中，internal global variable 应该被很谨慎的使用。因为指不定这个 variable 在某一时刻就要扩大其 visibility 了。

Code Example
--------

这个代码表述比较复杂（很多时候得从全局 IR 进行分析），这里便不再赘述了。不过值得提一下 internal global variable 的 IR code，这个可以学习一下。

下面代码是一个很简单的 function ``@foo`` ，其中它调用了一个 internal 的 global variable ``@internal_var`` 。
为什么是 internal 的呢？因为这会只有这个 function 内部可以 access 它，所以其实就没必要 global 都访问了。

.. code-block:: llvm

    @internal_var = internal global i32 42

    define void @foo() {
        %value = load i32, i32* @internal_var
        ; perform operations using the internal variable
        ret void
    }
