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

Writing ``CallCount`` Pass
------------

1. 为 ``CallCount`` pass 创建一个文件夹，并且在同一路径的 ``CMakeLists.txt`` 里指定这个文件夹。
   首先按照下面命令创建这个文件夹。

   .. code-block:: console

      $ # We assume that you have already installed LLVM 3.4 via above guidance.
      $ cd $YOUR-LOCAL-PATH$/llvm/lib/Transforms                               # change directory to the main folder that contains the LLVM Passes
      $ ls                                                                     # list files and folders in current path
      CMakeLists.txt  InstCombine      IPO            Makefile  Scalar  Vectorize
      Hello           Instrumentation  LLVMBuild.txt  ObjCARC   Utils
      $ # Except file CMakeList.txt, LLVMBuild.txt, and Makefile. Each of the rest folders denotes an individual LLVM Pass.
      $ mkdir CallCount

   然后按照如下命令，把这个文件夹的路径放在同一目录的 ``CMakeLists.txt`` 里。

   .. code-block:: console

      add_subdirectory(Utils)
      add_subdirectory(Instrumentation)
      add_subdirectory(InstCombine)
      add_subdirectory(Scalar)
      add_subdirectory(IPO)
      add_subdirectory(Vectorize)
      add_subdirectory(Hello)
      add_subdirectory(ObjCARC)
      add_subdirectory(CallCount)    # add path of folder that contains our target LLVM Pass

2. 写一下 ``CallCount`` pass 的 source code。
   首先移动到刚刚创建的文件夹里，然后创建一个空白文件 ``Hello.cpp``，这里面将是该 pass 的 source code。

   .. code-block:: console

      $ cd CallCount                 # change directory to target folder
      $ touch Hello.cpp              # create C++ source file of target LLVM Pass

   如刚才所说，``CallCount`` counts the number of ``call`` instructions in the static program IR。
   那我们在刚刚创建的 ``Hello.cpp`` 里写一下这个 pass。
   Source code 如下所示，我把我的解释都写在代码的 comments 里了。
   而且，没错，LLVM pass 的语法是基于 C++ 的，所以 C++ 的类和库都是可以使用的。

   .. code-block:: C++

      #include "llvm/ADT/Statistic.h"
      #include "llvm/IR/Function.h"
      #include "llvm/Pass.h"
      #include "llvm/Support/raw_ostream.h"
      #include "llvm/IR/Module.h"
      #include "llvm/IR/Type.h"
      #include "llvm/IR/Instructions.h"
      #include "llvm/IR/Instruction.h"
      #include "llvm/IR/IRBuilder.h"
      #include "llvm/Support/InstIterator.h"

      #include <iostream>
      #include <map>
      #include <list>
      #include <vector>
      #include <set>

      using namespace llvm;

      namespace{

         /****** analysis pass ********/
         struct CallCount : public ModulePass{                                                     // This pass is developed based on ModulePass.
                                                                                                   // There are also some other LLVM classes, such as:
                                                                                                   //    CallGraphSCCPass, FunctionPass, LoopPass, and RegionPass.
            static char ID;
            int call_count = 0;                                                                    // Global variable that records the number of call type instructions.

            CallCount() : ModulePass(ID) {}

            virtual bool runOnModule(Module &M){                                                   // For each program IR, load it as a Module.
                                                                                                   // Besides, you can regard this as the Main Function of this Pass.
                  for(Module::iterator F = M.begin(), E = M.end(); F!= E; ++F){                    // Iterate each function in this Module.
                     for(Function::iterator BB = F->begin(), E = F->end(); BB != E; ++BB){         // Iterate each basic-block in current function.
                        CallCount::runOnBasicBlock(BB, M.getContext());                            // Iterate each instructions in current basic-block.
                     }
                  }
                  return false;
            }

            virtual bool runOnBasicBlock(Function::iterator &BB, LLVMContext &context){            // The function that is used above, input is current basic-block.
                  for(BasicBlock::iterator BI = BB->begin(), BE = BB->end(); BI != BE; ++BI){      // Interate each instructions in current basic-block.
                     int opcode = BI->getOpcode();                                                 // Get opcode of current instruction:
                                                                                                   //     Opcode is the unique number for each instruction type.
                                                                                                   //     More details can be found at $YOUR-LOCAL-PATH$/llvm/include/llvm/IR/instruction.def
                     if(opcode == 49){                                                             // Record if the type of current instruction is "call".
                        call_count++;
                        outs() << call_count << '\n';
                     }
                  }
                  return true;
            }
         };
      }
      char CallCount::ID = 0;
      static RegisterPass<CallCount> X("CallCount", "Count call type instructions in given program IR", false, false);
                                                                                                   // "CallCount" is the unique flag of this pass while being loaded by opt command.

3. 接下来，我们准备 compile 一下刚写好的 pass。
   同样地，我把我的解释都写在代码的 comments 里了。
   首先，在和 ``Hello.cpp`` 相同的目录（也就是 ``CallCount`` 文件夹）里创建对应的 ``CMakeLists.txt`` 和 ``Makefile``。
   
   .. code-block:: console

      $ # Usually, the pass folder (i.e. CallCount here) should contain three files:
      $ #     Hello.cpp       --  The souce code of this LLVM Pass. Of course, you can give it another name. We have wrote as above.
      $ #     CMakeLists.txt  --  Link the source code with the compiled LLVM Pass. The LLVM Pass is .so format.
      $ #     Makefile        --  The compilation logic under this whole LLVM project.
      $ # So let's create the CMakeLists.txt and Makefile.
      $ touch CMakeLists.txt Makefile

   然后，我们写一下 ``CMakeLists.txt`` 里面的内容。

   .. code-block:: console

      add_llvm_loadable_module(CallCount      # The name of compiled LLVM Pass. So the output will be CallCount.so after the compilation.
         Hello.cpp                            # The source code of LLVM Pass.
      )

   我们再写一下 ``Makefile`` 里的内容。

   .. code-block:: console

      LEVEL = ../../..
      LIBRARYNAME = CallCount                 # Also the name of compiled LLVM Pass. This should be consistent with the name in CMakeLists.txt.
      LOADABLE_MODULE = 1
      USEDLIBS =

      # If we don't need RTTI or EH, there's no reason to export anything
      # from the hello plugin.
      ifneq ($(REQUIRES_RTTI), 1)
      ifneq ($(REQUIRES_EH), 1)
      EXPORTED_SYMBOL_FILE = $(PROJ_SRC_DIR)/Hello.exports
      endif
      endif

      include $(LEVEL)/Makefile.common
   
   现在我们就可以开始 compile 我们刚写好的 pass 了。
   具体命令如下所示（其实也就两条 commands），先移动一下路径，然后再 ``make`` 一下。

   .. code-block:: console

      $ cd $YOUR-LOCAL-PATH$/llvm/build      # change directory to where we build this LLVM project
      $ make -j$(nproc)                      # compile LLVM with multi-threads

4. 我们现在尝试用 LLVM optimizer ``opt`` load 一下 compile 好的 pass。
   一旦 compilation 完成的时候，你可以在 ``$YOUR-LOCAL-PATH$/llvm/build/lib/`` 里找到一个新生成的文件 ``CallCount.so``，这也就是 compile 好的 pass了。
   我们以一个叫 ``pathfinder.ll`` 的 LLVM IR 举例（recall that ``opt`` takes IR as both input and output），运行的命令如下所示：

   .. code-block:: console

      $ YOUR-LOCAL-PATH$/llvm/build/bin/ -load $YOUT-LOCAL-PATH$/llvm/build/lib/CallCount.so pathfinder.ll -CallCount -o output.ll
      $ # You can know how many "call" type instructions are in pathfinder.ll :)


上述所有代码可以在个 GitHub repo [#ref2]_ 找到。完结撒花！

References
--------
.. [#ref1] Writing an LLVM Pass: https://llvm.org/docs/WritingAnLLVMPass.html
.. [#ref2] Source code of ``CallCount`` pass: https://github.com/hyfshishen/LLFI-Quick-Start

