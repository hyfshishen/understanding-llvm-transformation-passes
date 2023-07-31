Writing an LLVM Pass
=====

这里以 LLVM 3.4 和 Ubuntu 16.04 环境为例，一步步地 install LLVM 并且写一个非常简单的 pass **CallCount**，即统计 ``call`` type 的 instruction 在一个给定的程序的 static LLVM IR 的中的数量。
static 这里指的是程序不运行的原始 code，也就是一个简单的 static program analysis；与之对应的是 dynamic program analysis，就是需要程序运行的那种。
你可能会觉得这个环境有点老，因为今年是2023年， Ubuntu 和 LLVM 已经分别出到 23.04 和 17.0.0 了。其实只是因为我用这套工具搞了实验很久，比较熟悉，所以这里就偷懒了orz。
更详细更新的 LLVM pass tutorial 可以查看官方的原始文档 [#ref1]_。
不过别担心，后面的 :doc:`LLVM-transformation-passes/index` 文档里都是基于最新的 LLVM 17.0.0 写的。

LLVM Installation
------------
在 Ubuntu 16.04 里安装 LLVM 3.4 的 commands 如下所示，大概需要十几分钟到半个小时，安装完的 project 大概有不到 4GB。

.. code-block:: console

   $ # Note that LLVM source code of other versions can be found at https://github.com/llvm/llvm-project.
   $ git clone https://github.com/zjuacompiler/llvm.git          # download LLVM 3.4 source code
   $ cd llvm                                                     # change directory to uncompressed folder
   $ ./configure --enable-optimized --disable-assertions --enable-targets=host --with-python=“/usr/bin/python2” # configure dependencies before you build LLVM                                  
   $ mkdir build && cd build                                     # "build" folder is for release
   $ cmake ..                                                    # prepare cmake file, ".." indicates the path to source code
   $ make -j$(nproc)    

不过对于比较新的 LLVM （比如 ≥ 15.0.0）安装的时候得小心点，如果不指定 subproject 的话可能一下子 50GB 就出去了。

Writing an LLVM Pass
------------

References
--------
.. [#ref1] Writing an LLVM Pass: https://llvm.org/docs/WritingAnLLVMPass.html

To use Lumache, first install it using pip:

.. code-block:: console

   (.venv) $ pip install lumache

Creating recipes
----------------

To retrieve a list of random ingredients,
you can use the ``lumache.get_random_ingredients()`` function:

.. autofunction:: lumache.get_random_ingredients

The ``kind`` parameter should be either ``"meat"``, ``"fish"``,
or ``"veggies"``. Otherwise, :py:func:`lumache.get_random_ingredients`
will raise an exception.

.. autoexception:: lumache.InvalidKindError

For example:

>>> import lumache
>>> lumache.get_random_ingredients()
['shells', 'gorgonzola', 'parsley']

