``-aggressive-instcombine``: Combine Expression Patterns
=====

Description
--------

``-aggressive-instcombine`` 是一个比 ``-instcombine`` 更 aggressive 的 transformation pass。
``-aggressive-instcombine`` 也不修改 program control-flow。
除了 arithmetic 和 bitwise 的 combinmation 外（这些都是O(1)复杂度），他还能 combine 更复杂的 expression patterns。
比如，一个 ``i8`` 的数字和一个整数相加，这里其实使用更简单的 ``i8`` 格式的 variable 就行了，所以这里 ``-aggressive-instcombine`` 会自动添加 TruncInst instruction 去 reduce the width of expressions。

Code Example
--------

其实就是可以识别 expression pattern (e.g. bit truncate) 的更复杂的 ``-instcombine``，故原理相似，可以查看前文，这里便不再赘述。