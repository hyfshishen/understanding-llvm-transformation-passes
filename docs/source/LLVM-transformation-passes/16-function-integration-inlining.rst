``-inline``: Function Integration/Inlining
=====

Description
--------

``-inline`` pass 其实和 ``-always-inline`` pass 做的事情非常相似，都是把 callee function 的 body 给 inline 在 caller function 中，所以这里便不再赘述了。
唯一的区别是， ``-always-inline`` 更 aggressive 一些 — 这会让很长的 function 也 ``inline``，导致 code size 非常长，让 compile 的代价变大； ``-inline`` 更 lightweight 一些，因为他会考虑 function size 等诸多因素。

Code Example
--------

例子可以看 ``-always-inline`` 里的，这里便不再赘述了。
