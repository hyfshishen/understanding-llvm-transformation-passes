``-lowerswitch``: Lower SwitchInsts to Branches
=====

Description
--------

``-lowerswitch`` pass 在做的事很好理解，也就是将原有程序中的 ``switch`` instruction 都变成 ``br`` instruction。除非用 ``switch`` instruction 会更便捷。

这里也顺便介绍一下这两个 instruction 是怎么用的。

- ``br`` instruction:

    Usages:

    .. code-block:: llvm

        br i1 <cond>, label <iftrue>, label <iffalse>   ; Conditional branch
        br label <dest>                                 ; Unconditional branch

    Examples:

    .. code-block:: llvm

        Test:
            %cond = icmp eq i32 %a, %b
            br i1 %cond, label %IfEqual, label %IfUnequal
        IfEqual:
            ret i32 1
        IfUnequal:
            ret i32 0

- ``select`` instruction: 

    Usages:

    .. code-block:: llvm

        switch <intty> <value>, label <defaultdest> [ <intty> <val>, label <dest> ... ]

    Examples:

    .. code-block:: llvm

        ; Emulate a conditional br instruction
        %Val = zext i1 %value to i32
        switch i32 %Val, label %truedest [ i32 0, label %falsedest ]

        ; Emulate an unconditional br instruction
        switch i32 0, label %dest [ ]

        ; Implement a jump table:
        switch i32 %val, label %otherwise [ i32 0, label %onzero
                                            i32 1, label %onone
                                            i32 2, label %ontwo ]

可以看出来，他们两个都是 control-flow 的跳转，不过 ``br`` 是固定在一个或两个之间跳转；而 ``switch`` 可以完成更复杂的跳转，比 ``br`` 更强大一些。

Code Example
--------

上面的例子已经说得很清楚了，所以这里便不再赘述。
