**************
LLVM Transformation Passes
**************

本节就会对于 LLVM Transformation Passes 中的每一个详细解释了。对于每一个 pass，我们主要从三块进行解释：
**0. 背景知识**。类似 prerequisite；只有一部分 pass 需要，很多 pass 都是可以直接理解的。所有的背景知识都可以在 :doc:`../compiler-basics/index` 中找到。
**1. 文字解释（Description）**。就是通过文字描述，僵硬的解释一下。
**2. 代码样例（Code Example）**。一般来说我们会用 LLVM IR 来写一下，transform 前 code 长啥样和 transform 后 code 长啥样。如果用 C 更直观的话我们就用 C 来解释了。

因为 Pass 比较多，所以 我把 Reference 集中写在这里了。
代码样例中的例子来源于，个人和 ChatGPT 疯狂的交互（然后再读 LLVM 文档 double check） [#ref1]_, LLVM 的官方文档 [#ref2]_，还有 Cornell CS:6120 [#ref3]_。
好的，那么我们现在就可以开始了。

.. toctree::
   :maxdepth: 1

   00-aggressive-dead-code-elimination

References
--------
.. [#ref1] ChatGPT: https://chat.openai.com/
.. [#ref2] LLVM Transformation Passes: https://llvm.org/docs/Passes.html#transform-passes
.. [#ref3] CS:6120@Cornell (2022 Spring), Advanced Compilers: https://www.cs.cornell.edu/courses/cs6120/2022sp/