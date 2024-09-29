``-constmerge``: Merge Duplicate Global Constants
=====

Description
--------

``-constmerge`` 的全称如上所示，其实就是 detect duplication 并且 merge 对应的 global variables to a shared one。
通过这种方式可以降低 memory footprint，进而 potentially improve the performance。
这个 pass 的应用场景包括：对于一些 pass 比如 TraceValues，它在执行的过程中会插入很多 string constants，其实对于有一些 string 已经是前面定义过并且 available 的了，这之后使用 ``-constmerge`` 就可以极大的降低 memory footprint。

Code Example
--------

例子还是使用了 merge string constants，如下所示，非常好理解：

原始的 IR。

.. code-block:: llvm

    @str1 = private constant [12 x i8] c"Hello, World\00"
    @str2 = private constant [12 x i8] c"Hello, World\00"
    @str3 = private constant [8 x i8] c"Welcome\00"

``-constmerge`` transform 之后的 IR。

.. code-block:: llvm

    @str1 = private constant [12 x i8] c"Hello, World\00"
    @str2 = @str1
    @str3 = private constant [8 x i8] c"Welcome\00"
