Understanding LLVM Transformation Passes
===================================

这个文档对当前 LLVM（17.0.0）所有的 transformation passes 提供了 **文字解释 (Description)** 和 **代码样例 (Code example)**，旨在让每个对 LLVM transformation passes 感兴趣的人可以绕过冗长晦涩的 LLVM 官方文档，用一个更简洁更高效的方式快速理解每个 pass 在做什么。

在开始这个文档之前，我先尝试回答一下可能出现的问题。

- 为什么要写这个文档？
   确实，现在有非常非常多优秀的文档都对 LLVM 或者是 compiler transformation 有着深入浅出地解释，比如 LLVM 和 Apple 官方的 tutorial 和 slides，亦或者是一些名校的课程（比如 Cornell 的 CS6120）。
   这确实会给人感觉我写的这个文档有种重复造轮子的感觉，而对于我来说我的 motivation 主要有如下几点：
   1. 首先，很多英文文档（比如 LLVM 的官方文档）写的太冗长了，很多时候我只是想精准定位某一个知识点而已，但是却发现自己在这个过程中看了很多没必要的东西。我希望把每个 pass 都整理成既简洁又生动（有代码例子）的知识点。
   3. 其次，我在写这个文档的时候本来就想自己随便写个私有笔记就完事了，但是在查询资料的过程中，我发现相关的中文资料实在是太少了。虽然有些文档确实也写得好，但是要不就是内容不完整，要不则是视频的形式有点啰嗦。所以希望自己写一个详细的中文文档，这样也可以顺便丰富下中文的预料社区，说不定哪天就被 ChatGPT 采用了呢。
   4. 然后，自己在之前搞科研的时候用到了很多 LLVM 相关的技术，但是总是没有形成一个体系，有种知识点非常支离破碎的感觉。这次也是希望通过这次系统性学习（不管是整理资料还是阅读并消化英文文档英文文档）的时候增加自己的知识储备，以后科研搞得更得心应手一些。
   5. 最后，我有一点整理癖，我感觉写文档的时候自己很快乐，很有成就感。
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



