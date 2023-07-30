**************
A Quick Primer about LLVM
**************

这一章节会提供一些非常 basic 的 LLVM 相关的基础概念，也是学习 LLVM transformation passes 的前提。
如果有兴趣的话，还可以顺着 :doc:`llvm-installation` 和 :doc:`writing-an-llvm-pass` 学习一下怎么安装 LLVM 并写一个非常简单的 pass。
本章的知识主要来源于我之前在 CS:4980@UIowa 写的一个 Tutorial [#ref1]_，同时也感谢这个文档 [#ref2]_ 的帮助，写得太棒了。
当然，如果你已经是 LLVM 大佬的话可以直接跳过这一章了。

LLVM Compiler Infrastructure
--------
Low-Level Virtual Machine (LLVM) 是一个 compiler infrastructure，它 targets 成为一个全能的 compiler。
LLVM 的架构如下图所示。
简而言之，LLVM 的核心设计在于 language/platform agnostic 的 common optimizer。
传统的 compiler 每支持一种 programming language 和一个 platform 都得把整个 frontend + backend 给重写一遍。
而 common optimizer 的存在使得 LLVM 的编译过程异常灵活：支持一个新的 programming language 只需要写其对应的 frontend；支持一个新的 platform 只需要写其对应的 backend。
除此之外，common optimizer 还可以进行各种各样有趣的 program analysis 和 transformation （没错，也就是本文档专注的 pass），可以帮助用户更好的理解和优化代码。
LLVM 自从2004年被 Chris Lattner 提出之后已经迭代了20年，现在已经是一个很大的 project 了。
我们很多耳熟能详的工具都是 LLVM 的某个 subproject：比如著名的 C/C++ compiler Clang/Clang++，symbolic execution tool KLEE，等等。

.. figure:: figures/llvm-structure.png
   :alt: LLVM compiler infrastructure

   LLVM compiler infrastructure


LLVM Intermediate Representation (IR)
--------

.. figure:: figures/llvm-IR.jpeg
   :alt: LLVM compiler infrastructure and IR
   
   LLVM compiler infrastructure and IR


LLVM Pass
--------

Userful LLVM Tools
--------

Others
--------
.. toctree::
   :maxdepth: 1
    llvm-installation
    writing-an-llvm-pass

References
--------
.. [#ref1] LLVM Tutorial in CS:4980@UIowa: https://hyfshishen.github.io/tutorial-01-llvm.html
.. [#ref2] Mapping High Level Constructs to LLVM IR: https://mapping-high-level-constructs-to-llvm-ir.readthedocs.io/en/latest/
