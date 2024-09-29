``-dse``: Dead Store Elimination
=====

Description
--------

``-dse`` 的目的是 remove unnecessary ``store`` instructions。
如何判断 ``store`` 是不是 necessary 的呢？
如果没有被之后的 instruction 读取到，那么它就是 unnecessary 的。
在 LLVM 的 transformation pass 中，这种情况比较少见（因为在 compile 的过程中，没有被后续读取的话也不会存的），所以这个 pass 比较 trivial，只看 basic block 内部的关系。

Code Example
--------

从 code 上看很好理解，在下面的 basic block 中，因为这个 ``store`` 没啥意义，所以直接删了。

原始的 IR code。

.. code-block:: llvm

    define i32 @example(i32 %a, i32 %b) {
        %sum = add i32 %a, %b   ; Compute the sum
        store i32 10, i32* %addr1     ; Store the value 10 in memory location addr1
        %result = add i32 %sum, %b    ; Compute the final result
        ret i32 %result
    }

``-dse`` transform 的 IR code。

.. code-block:: llvm

    define i32 @example(i32 %a, i32 %b) {
        %sum = add i32 %a, %b   ; Compute the sum
        %result = add i32 %sum, %b    ; Compute the final result
        ret i32 %result
    }
