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
   01-inline-for-always-inline-functions
   02-promote-by-reference-arguments-to-scalars
   03-basic-block-vectorization
   04-profile-guided-basic-block-placement
   05-break-critical-edges-in-cfg
   06-merge-duplicate-global-constants
   07-dead-code-elimination
   08-dead-argument-elimination
   09-dead-instruction-elimination
   10-dead-store-elimination
   11-deduce-function-attributes
   12-dead-global-elimination
   13-global-variable-optimizer
   14-global-value-numbering
   15-canonicalize-induction-variables
   16-function-integration-inlining
   17-partial-inliner
   18-combine-redundant-instructions
   19-combine-expression-patterns
   20-internalize-global-symbols
   21-sparse-conditional-constant-propagation
   22-interprocedural-sparse-conditional-constant-propagation
   23-jump-threading
   24-loop-invariant-code-motion
   25-delete-dead-loops
   26-extract-loops-into-new-functions
   27-loop-strength-reduction
   28-rotate-loops
   29-canonicalize-natural-loops
   30-unroll-loops
   31-unroll-and-jam-loops
   32-unswitch-loops
   33-loop-closed-ssa-form-pass
   34-lower-atomic-intrinsic-to-non-atomic-form
   35-lower-invokes-to-calls
   36-lower-switchinsts-to-branches
   37-promote-memory-to-register
   38-merge-functions
   39-unify-function-exits-nodes
   40-remove-unused-exception-handling-info
   41-reassociate-expressions
   42-demote-all-values-to-stack-slots
   43-scalar-placement-of-aggregates
   44-simplify-the-cfg
   45-code-sinking
   46-strip-unused-function-prototypes
   47-strip-all-symbols-from-a-module(including-others)
   48-tail-call-elimination

References
--------
.. [#ref1] ChatGPT: https://chat.openai.com/
.. [#ref2] LLVM Transformation Passes: https://llvm.org/docs/Passes.html#transform-passes
.. [#ref3] CS:6120@Cornell (2022 Spring), Advanced Compilers: https://www.cs.cornell.edu/courses/cs6120/2022sp/