Understanding LLVM Transformation Passes
===================================

这个文档对当前 LLVM（17.0.0）所有的 transformation passes 提供了 **文字解释 (Description)** 和 **代码样例 (Code example)**，旨在让每个对 LLVM transformation passes 感兴趣的人可以绕过冗长晦涩的 LLVM 官方文档，用一个更简洁更高效的方式快速理解每个 pass 在做什么。

在开始这个文档之前，我先尝试回答一下可能出现的问题。
- 为什么要写这个文档？
   亚凡在测试
- 如果我什么都不会，对 LLVM 和 compiler 一无所知，我能看懂这个文档吗？
- 为什么只包括 transformation passes？其他的比如 utility passes呢？
- 如果我只对某个 transformation pass 感兴趣，我要花多久读这个文档？
- 我读完这个文档之后能不能成为一个什么 LLVM pass 都会写的大佬？
- 为什么文档中出现大量的中英混杂？

**Lumache** (/lu'make/) is a Python library for cooks and food lovers
that creates recipes mixing random ingredients.
It pulls data from the `Open Food Facts database <https://world.openfoodfacts.org/>`_
and offers a *simple* and *intuitive* API.

Check out the :doc:`usage` section for further information, including
how to :ref:`installation` the project.

.. note::

   This project is under active development.

About
--------


.. toctree::
   :maxdepth: 1
   :caption: Contents:

   LLVM-a-quick-primer/index
   compiler-basics/index
   LLVM-transformation-passes/index

Contributing
--------
这份文档是 `Yafan Huang <https://hyfshishen.github.io/>`_ 在2023暑假的业余时间写的，文档源代码被在 `Github <https://github.com/hyfshishen/understanding-llvm-transformation-passes>`__ 上。
如果你发现了这个文档中的任何错误或者是问题（当然能花时间读我就已经很荣幸了QwQ），欢迎邮件联系 yafan-huang *at* uiowa *dot* com，多谢多谢！

License
--------

References
--------



