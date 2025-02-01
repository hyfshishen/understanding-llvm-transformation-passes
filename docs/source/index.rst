Understanding LLVM Transformation Passes
===================================

这个文档对当前 LLVM（17.0.0）所有的 transformation passes 提供了 **文字解释 (Description)** 和 **代码样例 (Code example)**，旨在让每个对 LLVM transformation passes 感兴趣的人可以绕过冗长晦涩的 LLVM 官方文档，用一个更简洁更高效的方式快速理解每个 pass 在做什么。

About
--------
在开始这个文档之前，我先尝试回答一下可能出现的问题，也通过这些问题来介绍一下这个文档。

- 什么是 LLVM ？什么是 LLVM 的 transformation passes？
   LLVM (Low Level Virtual Machine) 是一个非常吊的开源 compiler infrastructure。它提供了一个通用的类似汇编代码的IR（intermediate representation），和在 IR 上工作的 optimizer。
   这样新支持一种语言只需要设计其对应的 frontend，新支持一种硬件只需要设计其对应的 backend，非常灵活。在 optimizer 上，LLVM 可以实现 IR 层级的代码重组和优化，也就是通过写 pass 来实现，可以极大地提升程序的性能。
   而 LLVM transformation passes 则是 LLVM 官方实现的开源 pass，可以实现程序的各种各样的优化。
   没关系，如果这段读的不太明白的话可以看后面的文档 :doc:`LLVM-a-quick-primer/index`，那里有详细的解释。
- 为什么要写这个文档？
   确实，现在有非常非常多优秀的文档都对 LLVM 或者是 compiler transformation 有着深入浅出地解释，比如 LLVM 和 Apple 官方的 tutorial [#ref1]_ [#ref2]_ 和 slides [#ref3]_，亦或者是一些名校的课程（比如 Cornell 的 CS:6120 [#ref4]_）。
   这些课程都在我学习的时候使我受益良多。
   而对于我来说，开始写这个文档的 motivation 主要有如下几点：
   **首先**，很多英文文档（比如 LLVM 的官方文档）写的太冗长了，很多时候我只是想精准定位某一个知识点而已，但是却发现自己在这个过程中看了很多没必要的东西。我希望把每个 pass 都整理成既简洁又生动（有代码例子）的知识点。
   **其次**，我在写这个文档的时候本来就想自己随便写个私有笔记就完事了，但是在查询资料的过程中，我发现相关的中文资料实在是太少了。虽然有些文档确实也写得好，但是要不就是内容不完整，要不则是视频的形式有点啰嗦。所以希望自己写一个详细的中文文档，这样也可以顺便丰富下中文的预料社区，说不定哪天就被 ChatGPT 采用了呢。
   **然后**，自己在之前搞科研的时候用到了很多 LLVM 相关的技术，但是总是没有形成一个体系，有种知识点非常支离破碎的感觉。这次也是希望通过这次系统性学习（不管是整理资料还是阅读并消化英文文档英文文档）的时候增加自己的知识储备，以后科研搞得更得心应手一些。
   **最后**，我有一点整理癖，我感觉写文档的时候自己很快乐，很有成就感。
- 如果我什么都不会，对 LLVM 和 compiler 一无所知，我能看懂这个文档吗？
   没问题的，因为我写之前也近乎是一个 compiler 白痴。在解释所有的 transformation passes 之前，我还准备了两个章节 :doc:`LLVM-a-quick-primer/index` 和 :doc:`compiler-basics/index`。
   如果对 LLVM 完全一无所知的，可以先读第一个文档，大概只要花半个小时就可以对 LLVM 的基本概念有一些了解了（如果你耐心的话还可以学会写一个简单的 pass）。
   如果对 LLVM 有一些了解，但是对一些 compiler optimization 不太了解的，可以看一看第二个文档，也是大概半个小时就可以了；compiler optimization 包罗万象而且还在不断进步，我很难把所有的知识都放进来（我自己也不会啊），所以这里只放了和后面的 pass 相关的知识点。
- 为什么只包括 transformation passes？其他的比如 utility passes 或者 analysis passes 呢？
   一般我们写完代码，并且用 optimizer 对代码的 IR 优化的时候，只有 transformation passes 可以真正地维持程序原本功能的情况下 transform 程序，并提升程序的 performance。
   其他的 LLVM 官方 passes 比如 utility/analysis passes 主要以分析为主，而对程序的性能没有改变。
   Utility passes，打个比方 ``-view-cfg``，其目的是用一种可视化脚本（graphViz）打印一下当前程序的 control-flow graph。
   Analysis passes，打个比方 ``-print-function``，用 stderr 打印出来目标程序中所有的 function 信息。
   可以看到，utility/analysis passes 旨在分析程序和帮助用户更好的理解程序，从优化的角度对程序没有提升，所以本文只 focus 在 transformation passes 上。
- 如果我只对某个 transformation pass 感兴趣，我要花多久读这个文档？
   只理解一个pass的话，十分钟应该足够了。对于 :doc:`LLVM-transformation-passes/index` 的每个 pass，我都标注了理解它需要那些 :doc:`compiler-basics/index` 里的知识。
   比如学习 ``-licm`` pass 只需要了解什么是 loop-invariant code motion 就好了，并不需要把所有知识点全部理解一遍，可以节省一些时间。
- 我读完这个文档之后能不能成为一个什么 LLVM pass 都会写的大佬？
   呃，其实并不能（因为我自己也不是什么 pass 都可以写出来的天才）。
   读完之后只能说对 LLVM 所有的 transformation passes 和很多 compiler optimization 的技术都非常了解。
   如果说想成为写 pass 的大佬，那么还是得一点一点读和细扣官方文档和源代码。心急是吃不了热豆腐的。

Content
--------

.. toctree::
   :maxdepth: 2

   LLVM-a-quick-primer/index
   compiler-basics/index
   LLVM-transformation-passes/index

Contributing
--------
这份文档主要是 `Yafan Huang <https://hyfshishen.github.io/>`_ 在2023暑假和2024秋天的业余时间写的（其中也包含了一些2020~2022陆续写的个人笔记），文档源代码被在 `Github <https://github.com/hyfshishen/understanding-llvm-transformation-passes>`__ 上。
如果你发现了这个文档中的任何错误或者是问题（当然能花时间读我就已经很荣幸了QwQ），欢迎邮件联系 yafan-huang *at* uiowa *dot* edu，多谢多谢！

License
--------
UNLESS OTHERWISE NOTED, THE CONTENTS OF THIS REPOSITORY/DOCUMENT ARE LICENSED
UNDER THE CREATIVE COMMONS ATTRIBUTION - SHARE ALIKE 4.0 INTERNATIONAL LICENSE

.. image:: https://i.creativecommons.org/l/by-sa/4.0/88x31.png
   :target: https://creativecommons.org/licenses/by-sa/4.0/
   :alt: Creative Commons Attribution-ShareAlike 4.0 International License
   
References
--------
.. [#ref1] LLVM Transformation Passes: https://llvm.org/docs/Passes.html#transform-passes
.. [#ref2] Apple Writing an LLVM Pass: https://opensource.apple.com/source/clang/clang-137/src/docs/WritingAnLLVMPass.html
.. [#ref3] LLVM Optimization Slides: https://llvm.org/devmtg/2020-09/slides/A_Deep_Dive_into_Interprocedural_Optimization.pdf
.. [#ref4] CS:6120@Cornell: https://www.cs.cornell.edu/courses/cs6120/2019fa/


