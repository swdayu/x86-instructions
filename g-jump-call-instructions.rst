流程跳转指令
============

JMP
----

无条件跳转： ::

    操作码              指令             操作数  模式    简要描述
            E9 cw       JMP rel16           D   N.S./V  相对近跳转，相对于下一条指令
            E9 cd       JMP rel32           D   V/V     相对近跳转，RIP += 32-bit偏移符号扩展
            EA cd       JMP ptr16:16        S   Inv./V  绝对远跳转，地址由段+绝对地址给定
            EA cp       JMP ptr16:32        S   Inv./V  绝对远跳转, 地址由段+绝对地址给定
            EB cb       JMP rel8            D   V/V     短跳转，RIP += 8-bit偏移符号扩展
            FF /4       JMP r/m16           M   N.S./V  绝对近跳转，间接地址为 r/m16 零扩展
            FF /4       JMP r/m32           M   N.S./V  绝对近跳转，间接地址为 r/m32 零扩展
            FF /4       JMP r/m64           M   V/N.E.  绝对近跳转，间接地址 RIP = r/m64
            FF /5       JMP m16:16          M   V/V     绝对远跳转，间接地址由 m16:16 给定
            FF /5       JMP m16:32          M   V/V     绝对远跳转，间接地址由 m16:32 给定
    REX.W + FF /5       JMP m16:64          M   V/N.E.  绝对远跳转，间接地址由 m16:64 给定

    * cw 操作码之后跟随2字节代码偏移
    * cd 操作码之后跟随4字节代码偏移，并可能为代码段寄存器指定新的段
    * cp 操作码之后跟随6字节代码偏移，并可能为代码段寄存器指定新的段

    类型    操作数1             操作数2         操作数3     操作数4
    S       段+绝对地址          无              无          无
    D       Offset              无              无          无
    M       ModRM:r/m (r)       无              无          无

    JMP 跳转（非64位模式）
    段内短程跳转                        1110 1011 : disp8           EIP += disp8符号扩展
    段内直接跳转                        1110 1001 : full disp       EIP += disp符号扩展
    段内间接跳转                        1111 1111 : 11 100 reg      EIP = reg
    段内间接跳转                        1111 1111 : mm 100 mem      EIP = mem
    跨段直接跳转                        1110 1010 : full disp       EIP = Sreg : disp
    跨段间接跳转                        1111 1111 : mm 101 mem      EIP = Sreg : mem

    JMP 跳转（64位模式）
    段内短程跳转                        1110 1011 : disp8           RIP += disp8符号扩展
    段内直接跳转                        1110 1001 : disp32          RIP += disp32符号扩展
    段内间接跳转            0100 W00B : 1111 1111 : 11 100 r64      RIP = r64
    段内间接跳转            0100 W0XB : 1111 1111 : mm 100 m64      RIP = m64
    跨段间接跳转            0100 00XB : 1111 1111 : mm 101 m32      RIP = Sreg : m32
    跨段间接跳转            0100 10XB : 1111 1111 : mm 101 m64      RIP = Sreg : m64

    JMP (E9H)      (EAH)     (EBH)
        near↑f64   far↑f64   short↑f64
          Jz         Ap        Jb

    JMP (FFH)   mm 100 r/m      mm 101 mem
                near↑f64        far JMP
                  Ev              Mp

    * ↑f64  在 64 位模式下，操作数大小被强制为 64 位操作数大小
    * Jz    操作数大小对于16位是两字节，对于32/64位是四字节，位于偏移字段，是相对指令指针的偏移
    * Ap    操作数大小根据属性决定，32/48/80位指针，位于偏移字段是直接地址
    * Jb    操作数大小为字节，位于偏移字段，是相对指令指针的偏移
    * Ev    操作数大小根据属性决定，位于r/m字段，是寄存器或内存数据
    * Mp    操作数大小根据属性决定，32/48/80位指针，位于r/m字段只能引用内存

**操作**

该指令可以执行四种不同类型的跳转：

1. 近跳转（near jump）- 段内跳转，该段为 CS 段寄存器当前指向的代码段
2. 短跳转（short jump）- 跳转范围限制在 -128 到 127 范围内，相对于当前的指令指针
3. 远跳转（far jump）- 跨段跳转，跳转到拥有相同特权等级的另外一个代码段的一个指令位置
4. 任务切换（task switch）- 远跳转到另一个任务的一个指令位置

任务切换仅能在保护模式执行，参见第3卷第8章。近跳转的目标操作数可以是一个绝对偏移，或者
相对偏移，相对偏移通常对应为汇编代码中的一个标签，但是在机器代码级别，它是一个8/16/32
位的有符号立即数，然后加到当前的指令指针寄存器上。当前的指令指针的值，表示的是当前JMP
指令之后的那个指令的地址。

指令顺序。在远跳转之后的指令可能在较早指令完成执行之前从内存中预取，但它们不会执行，直到
所有远跳转之前的指令执行完毕，但可能在之前的指令产生的数据完成存储并全局可见之前执行。而
间接近跳转之后的指令可能被推测执行，如果软件需要防止这种情况，可以在间接近跳转指令之后放
置一个 INT3 或 LFENCE 指令操作码，以阻止推测执行。

**标志位**

如果发生了任务切换所有标志位都受影响，如果没有任务切换不影响标志位。

Jcc
----

条件跳转： ::

    操作码              指令             操作数  模式    简要描述
    70 cb               JO rel8             D   V/V     短跳转如果溢出 OF=1
    71 cb               JNO rel8            D   V/V     短跳转如果不溢出 OF=0
    72 cb               JB rel8             D   V/V     短跳转如果低于 CF=1
    72 cb               JC rel8             D   V/V     短跳转如果进位 CF=1
    72 cb               JNAE rel8           D   V/V     短跳转如果不高于等于 CF=1
    73 cb               JAE rel8            D   V/V     短跳转如果高于等于 CF=0
    73 cb               JNB rel8            D   V/V     短跳转如果不低于 CF=0
    73 cb               JNC rel8            D   V/V     短跳转如果不进位 CF=0
    74 cb               JE rel8             D   V/V     短跳转如果等于 ZF=1
    74 cb               JZ rel8             D   V/V     短跳转如果是零 ZF=1
    75 cb               JNE rel8            D   V/V     短跳转如果不等于 ZF=0
    75 cb               JNZ rel8            D   V/V     短跳转如果不是零 ZF=0
    76 cb               JBE rel8            D   V/V     短跳转如果低于等于 CF=1 或 ZF=1
    76 cb               JNA rel8            D   V/V     短跳转如果不高于 CF=1 或 ZF=1
    77 cb               JA rel8             D   V/V     短跳转如果高于 CF=0 ZF=0
    77 cb               JNBE rel8           D   V/V     短跳转如果不低于等于 CF=0 ZF=0
    78 cb               JS rel8             D   V/V     短跳转如果有符号位 SF=1
    79 cb               JNS rel8            D   V/V     短跳转如果没有符号 SF=0
    7A cb               JP rel8             D   V/V     短跳转如果是偶数个1 PF=1
    7A cb               JPE rel8            D   V/V     短跳转如果是偶数个1 PF=1
    7B cb               JPO rel8            D   V/V     短跳转如果不是偶数个1 PF=0
    7B cb               JNP rel8            D   V/V     短跳转如果不是偶数个1 PF=0
    7C cb               JL rel8             D   V/V     短跳转如果小于 SF≠OF
    7C cb               JNGE rel8           D   V/V     短跳转如果不大于等于 SF≠OF
    7D cb               JGE rel8            D   V/V     短跳转如果大于等于 SF=OF
    7D cb               JNL rel8            D   V/V     短跳转如果不小于 SF=OF
    7E cb               JLE rel8            D   V/V     短跳转如果小于等于 ZF=1 或 SF≠OF
    7E cb               JNG rel8            D   V/V     短跳转如果不大于 ZF=1 或 SF≠OF
    7F cb               JG rel8             D   V/V     短跳转如果大于 ZF=0 SF=OF
    7F cb               JNLE rel8           D   V/V     短跳转如果不小于等于 ZF=0 SF=OF
    E3 cb               JCXZ rel8           D   N.E./V  短跳转如果 CX 是 0
    E3 cb               JECXZ rel8          D   V/V     短跳转如果 ECX 是 0
    E3 cb               JRCXZ rel8          D   V/N.E.  短跳转如果 RCX 是 0
    0F 80 cw            JO rel16            D   N.S./V  近跳转如果溢出 OF=1
    0F 80 cd            JO rel32            D   V/V     近跳转如果溢出 OF=1
    0F 81 cw            JNO rel16           D   N.S./V  近跳转如果不溢出 OF=0
    0F 81 cd            JNO rel32           D   V/V     近跳转如果不溢出 OF=0
    0F 82 cw            JB rel16            D   N.S./V  近跳转如果低于 CF=1
    0F 82 cd            JB rel32            D   V/V     近跳转如果低于 CF=1
    0F 82 cw            JC rel16            D   N.S./V  近跳转如果进位 CF=1
    0F 82 cd            JC rel32            D   V/V     近跳转如果进位 CF=1
    0F 82 cw            JNAE rel16          D   N.S./V  近跳转如果不高于等于 CF=1
    0F 82 cd            JNAE rel32          D   V/V     近跳转如果不高于等于 CF=1
    0F 83 cw            JNB rel16           D   N.S./V  近跳转如果不低于 CF=0
    0F 83 cd            JNB rel32           D   V/V     近跳转如果不低于 CF=0
    0F 83 cw            JAE rel16           D   N.S./V  近跳转如果高于等于 CF=0
    0F 83 cd            JAE rel32           D   V/V     近跳转如果高于等于 CF=0
    0F 83 cw            JNC rel16           D   N.S./V  近跳转如果不进位 CF=0
    0F 83 cd            JNC rel32           D   V/V     近跳转如果不进位 CF=0
    0F 84 cw            JE rel16            D   N.S./V  近跳转如果等于 ZF=1
    0F 84 cd            JE rel32            D   V/V     近跳转如果等于 ZF=1
    0F 84 cw            JZ rel16            D   N.S./V  近跳转如果是零 ZF=1
    0F 84 cd            JZ rel32            D   V/V     近跳转如果是零 ZF=1
    0F 84 cw            JZ rel16            D   N.S./V  近跳转如果是零 ZF=1
    0F 84 cd            JZ rel32            D   V/V     近跳转如果是零 ZF=1
    0F 85 cw            JNE rel16           D   N.S./V  近跳转如果不等于 ZF=0
    0F 85 cd            JNE rel32           D   V/V     近跳转如果不等于 ZF=0
    0F 85 cw            JNZ rel16           D   N.S./V  近跳转如果不是零 ZF=0
    0F 85 cd            JNZ rel32           D   V/V     近跳转如果不是零 ZF=0
    0F 86 cw            JBE rel16           D   N.S./V  近跳转如果低于等于 CF=1 或 ZF=1
    0F 86 cd            JBE rel32           D   V/V     近跳转如果低于等于 CF=1 或 ZF=1
    0F 86 cw            JNA rel16           D   N.S./V  近跳转如果不高于 CF=1 或 ZF=1
    0F 86 cd            JNA rel32           D   V/V     近跳转如果不高于 CF=1 或 ZF=1
    0F 87 cw            JA rel16            D   N.S./V  近跳转如果高于 CF=0 ZF=0
    0F 87 cd            JA rel32            D   V/V     近跳转如果高于 CF=0 ZF=0
    0F 87 cw            JNBE rel16          D   N.S./V  近跳转如果不低于等于 CF=0 ZF=0
    0F 87 cd            JNBE rel32          D   V/V     近跳转如果不低于等于 CF=0 ZF=0
    0F 88 cw            JS rel16            D   N.S./V  近跳转如果有符号位 SF=1
    0F 88 cd            JS rel32            D   V/V     近跳转如果有符号位 SF=1
    0F 89 cw            JNS rel16           D   N.S./V  近跳转如果没有符号 SF=0
    0F 89 cd            JNS rel32           D   V/V     近跳转如果没有符号 SF=0
    0F 8A cw            JP rel16            D   N.S./V  近跳转如果是偶数个1 PF=1
    0F 8A cd            JP rel32            D   V/V     近跳转如果是偶数个1 PF=1
    0F 8A cw            JPE rel16           D   N.S./V  近跳转如果是偶数个1 PF=1
    0F 8A cd            JPE rel32           D   V/V     近跳转如果是偶数个1 PF=1
    0F 8B cw            JPO rel16           D   N.S./V  近跳转如果不是偶数个1 PF=0
    0F 8B cd            JPO rel32           D   V/V     近跳转如果不是偶数个1 PF=0
    0F 8B cw            JNP rel16           D   N.S./V  近跳转如果不是偶数个1 PF=0
    0F 8B cd            JNP rel32           D   V/V     近跳转如果不是偶数个1 PF=0
    0F 8C cw            JL rel16            D   N.S./V  近跳转如果小于 SF≠OF
    0F 8C cd            JL rel32            D   V/V     近跳转如果小于 SF≠OF
    0F 8C cw            JNGE rel16          D   N.S./V  近跳转如果不大于等于 SF≠OF
    0F 8C cd            JNGE rel32          D   V/V     近跳转如果不大于等于 SF≠OF
    0F 8D cw            JNL rel16           D   N.S./V  近跳转如果不小于 SF=OF
    0F 8D cd            JNL rel32           D   V/V     近跳转如果不小于 SF=OF
    0F 8D cw            JGE rel16           D   N.S./V  近跳转如果大于等于 SF=OF
    0F 8D cd            JGE rel32           D   V/V     近跳转如果大于等于 SF=OF
    0F 8E cw            JLE rel16           D   N.S./V  近跳转如果小于等于 ZF=1 或 SF≠OF
    0F 8E cd            JLE rel32           D   V/V     近跳转如果小于等于 ZF=1 或 SF≠OF
    0F 8E cw            JNG rel16           D   N.S./V  近跳转如果不大于 ZF=1 或 SF≠OF
    0F 8E cd            JNG rel32           D   V/V     近跳转如果不大于 ZF=1 或 SF≠OF
    0F 8F cw            JG rel16            D   N.S./V  近跳转如果大于 ZF=0 SF=OF
    0F 8F cd            JG rel32            D   V/V     近跳转如果大于 ZF=0 SF=OF
    0F 8F cw            JNLE rel16          D   N.S./V  近跳转如果不小于等于 ZF=0 SF=OF
    0F 8F cd            JNLE rel32          D   V/V     近跳转如果不小于等于 ZF=0 SF=OF

    * cb 操作码之后跟随1字节代码偏移
    * cw 操作码之后跟随2字节代码偏移
    * cd 操作码之后跟随4字节代码偏移，并可能为代码段寄存器指定新的段
    * cp 操作码之后跟随6字节代码偏移，并可能为代码段寄存器指定新的段

    类型    操作数1             操作数2         操作数3     操作数4
    D       Offset              无              无          无

    Jcc↑f64, Jb - short disp jump on cc
        (70H)   (71H)   (72H)   (73H)   (74H)   (75H)   (76H)   (77H)
          O       NO   B/NAE/C NB/AE/NC  Z/E    NZ/NE   BE/NA   NBE/A
        (78H)   (79H)   (7AH)   (7BH)   (7CH)   (7DH)   (7EH)   (7FH)
          S       NS    P/PE    NP/PO   L/NGE   NL/GE   LE/NG   NLE/G

    JrCXZ↑f64   (E3H)
                 Jb

    Jcc↑f64, Jz - long/near disp jump on cc
        (80H)   (81H)   (82H)   (83H)   (84H)   (85H)   (86H)   (87H)
          O       NO   B/C/NAE AE/NB/NC  E/Z    NE/NZ   BE/NA   A/NBE
        (88H)   (89H)   (8AH)   (8BH)   (8CH)   (8DH)   (8EH)   (8FH)
          S       NS    P/PE    NP/PO   L/NGE   NL/GE   LE/NG   NLE/G

    * ↑f64  在 64 位模式下，操作数大小被强制为 64 位操作数大小
    * Jb    操作数大小为字节，位于偏移字段，是相对指令指针的偏移
    * Jz    操作数大小对于16位是两字节，对于32/64位是四字节，位于偏移字段，是相对指令指针的偏移

**操作**

根据标志位条件进行跳转： ::

    if cc {
        temp := EIP/RIP + SignExtend(DEST)
        if temp is not within code segment limit {
            #GP(0);
        } else {
            EIP/RIP := temp;
        }
    }

**标志位**

不影响标志位。

LOOP
-----

循环： ::

    操作码              指令             操作数  模式    简要描述
    E0 cb               LOOPNE rel8         D   V/V     计数器减1，短跳转如果 count≠0 ZF=0
    E1 cb               LOOPE rel8          D   V/V     计数器减1，短跳转如果 count≠0 ZF=1
    E2 cb               LOOP rel8           D   V/V     计数器减1，短跳转如果 count≠0

    类型    操作数1             操作数2         操作数3     操作数4
    D       Offset              无              无          无

    LOOP        (E0H)                   (E1H)           (E2H)
        LOOPNE↑f64/LOOPNZ↑f64   LOOPE↑f64/LOOPZ↑f64    LOOP↑f64
                 Jb                      Jb               Jb

    * ↑f64  在 64 位模式下，操作数大小被强制为 64 位操作数大小
    * Jb    操作数大小为字节，位于偏移字段，是相对指令指针的偏移

**操作**

根据 CX/ECX/RCX 计数进行循环，先对计数器减1，再判断是否为0，如果为0结束循环，否则继续
循环。跳转的偏移必须在 -128 到 127 范围内，相对于当前的指令指针 IP/EIP/RIP。 ::

    Count := Count - 1;
    <LOOP>
        if Count ≠ 0 {
            BranchCond := 1
        } else {
            BranchCond := 0
        }
    <LOOPE LOOPZ>
        if ZF = 1 and Count ≠ 0 {
            BranchCond := 1
        } else {
            BranchCond := 0
        }
    <LOOPNE LOOPNZ>
        if ZF = 0 and Count ≠ 0 {
            BranchCond := 1
        } else {
            BranchCond := 0
        }
    if BranchCond = 1 {
        temp := RIP/EIP + SignExtend(DEST);
        if temp is not canonical {
            #GP(0);
        } else {
            RIP/EIP := temp;
        }
    } else {
        Terminate loop and continue at RIP/EIP;
    }

**标志位**

不影响标志位。

CMP
----

大小比较： ::

    操作码              指令             操作数  模式    简要描述
            38 /r       CMP r/m8, r8        MR  V/V     比较 r8 和 r/m8
      REX + 38 /r       CMP r/m8*, r8*      MR  V/N.E.  比较 r8 和 r/m8
            39 /r       CMP r/m16, r16      MR  V/V     比较 r16 和 r/m16
            39 /r       CMP r/m32, r32      MR  V/V     比较 r32 和 r/m32
    REX.W + 39 /r       CMP r/m64,r64       MR  V/N.E.  比较 r64 和 r/m64
            3A /r       CMP r8, r/m8        RM  V/V     比较 r/m8 和 r8
      REX + 3A /r       CMP r8*, r/m8*      RM  V/N.E.  比较 r/m8 和 r8
            3B /r       CMP r16, r/m16      RM  V/V     比较 r/m16 和 r16
            3B /r       CMP r32, r/m32      RM  V/V     比较 r/m32 和 r32
    REX.W + 3B /r       CMP r64, r/m64      RM  V/N.E.  比较 r/m64 和 r64
            3C ib       CMP AL, imm8        I   V/V     比较 imm8 和 AL
            3D iw       CMP AX, imm16       I   V/V     比较 imm16 和 AX
            3D id       CMP EAX, imm32      I   V/V     比较 imm32 和 EAX
    REX.W + 3D id       CMP RAX, imm32      I   V/N.E.  比较 imm32符号扩展到64位 和 RAX
            80 /7 ib    CMP r/m8, imm8      MI  V/V     比较 imm8 和 r/m8
      REX + 80 /7 ib    CMP r/m8*, imm8     MI  V/N.E.  比较 imm8 和 r/m8
            81 /7 iw    CMP r/m16, imm16    MI  V/V     比较 imm16 和 r/m16
            81 /7 id    CMP r/m32, imm32    MI  V/V     比较 imm32 和 r/m32
    REX.W + 81 /7 id    CMP r/m64, imm32    MI  V/N.E.  比较 imm32符号扩展到64位 和 r/m64
            83 /7 ib    CMP r/m16, imm8     MI  V/V     比较 imm8 和 r/m16
            83 /7 ib    CMP r/m32, imm8     MI  V/V     比较 imm8 和 r/m32
    REX.W + 83 /7 ib    CMP r/m64, imm8     MI  V/N.E.  比较 imm8 和 r/m64

    类型    操作数1             操作数2         操作数3     操作数4
    RM      ModRM:reg (r)       ModRM:r/m (r)   无          无
    MR      ModRM:r/m (r)       ModRM:reg (r)   无          无
    MI      ModRM:r/m (r)       imm8/16/32      无          无
    I       AL/AX/EAX/RAX (r)   imm8/16/32      无          无

    CMP (38H)   (39H)   (3AH)   (3BH)   (3CH)   (3DH)
        Eb,Gb   Ev,Gv   Gb,Eb   Gv,Ev   AL,Ib   rAX,Iz

    GMP (80H) mm 111 r/m
        Eb,Ib
        (81H) mm 111 r/m
        Ev,Iz
        (83H) mm 111 r/m
        Ev,Ib

    * Eb    操作数大小是字节，位于r/m字段，是寄存器或内存数据
    * Ev    操作数大小根据属性决定，位于r/m字段，是寄存器或内存数据
    * Gb    操作数大小是字节，位于reg字段，是一个寄存器
    * Gv    操作数大小根据属性决定，位于reg字段，是一个寄存器
    * Ib    操作数大小是字节，位于立即数字段
    * Iz    操作数大小根据属性决定，是两个字节（16位）或四个字节（32/64位），位于立即数字段
    * AL    操作数大小是字节，不使用ModR/M字段，规定为AL寄存器
    * rAX   操作数大小根据属性决定，不使用ModR/M字段，规定为AX/EAX/RAX寄存器

**操作**

比较两个操作数，比较的结果是第一操作数减去第二操作数，结果会被丢弃： ::

    TEMP := SRC1 - SignExtend(SRC2)
    ModifyStatusFlags as SUB instruction;

**标志位**

影响 CF OF SF ZF AF PF 标志位。

TEST
-----

位与测试： ::

    操作码              指令             操作数  模式    简要描述
            84 /r       TEST r/m8, r8       MR  V/V     r8 & r/m8
      REX + 84 /r       TEST r/m8*, r81     MR  V/N.E.  r8 & r/m8
            85 /r       TEST r/m16, r16     MR  V/V     r16 & r/m16
            85 /r       TEST r/m32, r32     MR  V/V     r32 & r/m32
    REX.W + 85 /r       TEST r/m64, r64     MR  V/N.E.  r64 & r/m64
            A8 ib       TEST AL, imm8       I   V/V     imm8 & AL
            A9 iw       TEST AX, imm16      I   V/V     imm16 & AX
            A9 id       TEST EAX, imm32     I   V/V     imm32 & EAX
    REX.W + A9 id       TEST RAX, imm32     I   V/N.E.  imm32符号扩展 & RAX
            F6 /0 ib    TEST r/m8, imm8     MI  V/V     imm8 & r/m8
      REX + F6 /0 ib    TEST r/m8*, imm8    MI  V/N.E.  imm8 & r/m8
            F7 /0 iw    TEST r/m16, imm16   MI  V/V     imm16 & r/m16
            F7 /0 id    TEST r/m32, imm32   MI  V/V     imm32 & r/m32
    REX.W + F7 /0 id    TEST r/m64, imm32   MI  V/N.E.  imm32符号扩展 & r/m64

    类型    操作数1             操作数2         操作数3     操作数4
    I       AL/AX/EAX/RAX       imm8/16/32      无          无
    MI      ModRM:r/m (r)       imm8/16/32      无          无
    MR      ModRM:r/m (r)       ModRM:reg (r)   无          无

    TEST (84H)  (85H)
         Eb,Gb  Ev,Gv

    TEST (A8H)  (A9H)
         AL,Ib  rAX,Iz

    TEST (F6H)  (F7H)
        Eb,Ib   Ev,Iz

    * Eb    操作数大小是字节，位于r/m字段，是寄存器或内存数据
    * Ev    操作数大小根据属性决定，位于r/m字段，是寄存器或内存数据
    * Gb    操作数大小是字节，位于reg字段，是一个寄存器
    * Gv    操作数大小根据属性决定，位于reg字段，是一个寄存器
    * Ib    操作数大小是字节，位于立即数字段
    * Iz    操作数大小根据属性决定，是两个字节（16位）或四个字节（32/64位），位于立即数字段
    * AL    操作数大小是字节，不使用ModR/M字段，规定为AL寄存器
    * rAX   操作数大小根据属性决定，不使用ModR/M字段，规定为AX/EAX/RAX寄存器

**操作**

计算两个操作数的位与并设置标志，位与的结果被丢弃： ::

    TEMP := SRC1 SRC2;
    SF := MSB(TEMP);
    IF TEMP = 0
        THEN ZF := 1;
        ELSE ZF := 0;
    FI;
    PF := BitwiseXNOR(TEMP[0:7]);
    CF := 0;
    OF := 0;

**标志位**

OF 和 CF 清零，SF ZF PF 根据结果设置，AF 的状态未定义。

SETcc
------

条件置位： ::

    操作码              指令             操作数  模式    简要描述
          0F 90         SETO r/m8       M       V/V     置1如果溢出 OF=1
    REX + 0F 90         SETO r/m8*      M       V/N.E.  置1如果溢出 OF=1
          0F 91         SETNO r/m8      M       V/V     置1如果不溢出 OF=0
    REX + 0F 91         SETNO r/m8*     M       V/N.E.  置1如果不溢出 OF=0
          0F 92         SETB r/m8       M       V/V     置1如果低于 CF=1
    REX + 0F 92         SETB r/m8*      M       V/N.E.  置1如果低于 CF=1
          0F 92         SETC r/m8       M       V/V     置1如果进位 CF=1
    REX + 0F 92         SETC r/m8*      M       V/N.E.  置1如果进位 CF=1
          0F 92         SETNAE r/m8     M       V/V     置1如果不高于等于 CF=1
    REX + 0F 92         SETNAE r/m8*    M       V/N.E.  置1如果不高于等于 CF=1
          0F 93         SETAE r/m8      M       V/V     置1如果高于等于 CF=0
    REX + 0F 93         SETAE r/m8*     M       V/N.E.  置1如果高于等于 CF=0
          0F 93         SETNB r/m8      M       V/V     置1如果不低于 CF=0
    REX + 0F 93         SETNB r/m8*     M       V/N.E.  置1如果不低于 CF=0
          0F 93         SETNC r/m8      M       V/V     置1如果不进位 CF=0
    REX + 0F 93         SETNC r/m8*     M       V/N.E.  置1如果不进位 CF=0
          0F 94         SETE r/m8       M       V/V     置1如果等于 ZF=1
    REX + 0F 94         SETE r/m8*      M       V/N.E.  置1如果等于 ZF=1
          0F 94         SETZ r/m8       M       V/V     置1如果是零 ZF=1
    REX + 0F 94         SETZ r/m8*      M       V/N.E.  置1如果是零 ZF=1
          0F 95         SETNE r/m8      M       V/V     置1如果不等于 ZF=0
    REX + 0F 95         SETNE r/m8*     M       V/N.E.  置1如果不等于 ZF=0
          0F 95         SETNZ r/m8      M       V/V     置1如果不是零 ZF=0
    REX + 0F 95         SETNZ r/m8*     M       V/N.E.  置1如果不是零 ZF=0
          0F 96         SETBE r/m8      M       V/V     置1如果低于等于 CF=1 或 ZF=1
    REX + 0F 96         SETBE r/m8*     M       V/N.E.  置1如果低于等于 CF=1 或 ZF=1
          0F 96         SETNA r/m8      M       V/V     置1如果不高于 CF=1 或 ZF=1
    REX + 0F 96         SETNA r/m8*     M       V/N.E.  置1如果不高于 CF=1 或 ZF=1
          0F 97         SETA r/m8       M       V/V     置1如果高于 CF=0 ZF=0
    REX + 0F 97         SETA r/m8*      M       V/N.E.  置1如果高于 CF=0 ZF=0
          0F 97         SETNBE r/m8     M       V/V     置1如果不低于等于 CF=0 ZF=0
    REX + 0F 97         SETNBE r/m8*    M       V/N.E.  置1如果不低于等于 CF=0 ZF=0
          0F 98         SETS r/m8       M       V/V     置1如果有符号位 SF=1
    REX + 0F 98         SETS r/m8*      M       V/N.E.  置1如果有符号位 SF=1
          0F 99         SETNS r/m8      M       V/V     置1如果没有符号 SF=0
    REX + 0F 99         SETNS r/m8*     M       V/N.E.  置1如果没有符号 SF=0
          0F 9A         SETP r/m8       M       V/V     置1如果偶数个1 PF=1
    REX + 0F 9A         SETP r/m8*      M       V/N.E.  置1如果偶数个1 PF=1
          0F 9A         SETPE r/m8      M       V/V     置1如果偶数个1 PF=1
    REX + 0F 9A         SETPE r/m8*     M       V/N.E.  置1如果偶数个1 PF=1
          0F 9B         SETPO r/m8      M       V/V     置1如果不是偶数个1 PF=0
    REX + 0F 9B         SETPO r/m8*     M       V/N.E.  置1如果不是偶数个1 PF=0
          0F 9B         SETNP r/m8      M       V/V     置1如果不是偶数个1 PF=0
    REX + 0F 9B         SETNP r/m8*     M       V/N.E.  置1如果不是偶数个1 PF=0
          0F 9C         SETL r/m8       M       V/V     置1如果小于 SF≠OF
    REX + 0F 9C         SETL r/m8*      M       V/N.E.  置1如果小于 SF≠OF
          0F 9C         SETNGE r/m8     M       V/V     置1如果不大于等于 SF≠OF
    REX + 0F 9C         SETNGE r/m8*    M       V/N.E.  置1如果不大于等于 SF≠OF
          0F 9D         SETNL r/m8      M       V/V     置1如果不小于 SF=OF
    REX + 0F 9D         SETNL r/m8*     M       V/N.E.  置1如果不小于 SF=OF
          0F 9D         SETGE r/m8      M       V/V     置1如果大于等于 SF=OF
    REX + 0F 9D         SETGE r/m8*     M       V/N.E.  置1如果大于等于 SF=OF
          0F 9E         SETLE r/m8      M       V/V     置1如果小于等于 ZF=1 或 SF≠OF
    REX + 0F 9E         SETLE r/m8*     M       V/N.E.  置1如果小于等于 ZF=1 或 SF≠OF
          0F 9E         SETNG r/m8      M       V/V     置1如果不大于 ZF=1 或 SF≠OF
    REX + 0F 9E         SETNG r/m8*     M       V/N.E.  置1如果不大于 ZF=1 或 SF≠OF
          0F 9F         SETNLE r/m8     M       V/V     置1如果不小于等于 ZF=0 SF=OF
    REX + 0F 9F         SETNLE r/m8*    M       V/N.E.  置1如果不小于等于 ZF=0 SF=OF
          0F 9F         SETG r/m8       M       V/V     置1如果大于 ZF=0 SF=OF
    REX + 0F 9F         SETG r/m8*      M       V/N.E.  置1如果大于 ZF=0 SF=OF

    类型    操作数1             操作数2         操作数3     操作数4
    M       ModRM:r/m (w)       无              无          无

    SETcc, Eb
        (90H)   (91H)   (92H)   (93H)   (94H)   (95H)   (96H)   (97H)
          O       NO   B/C/NAE AE/NB/NC  E/Z    NE/NZ   BE/NA   A/NBE
        (98H)   (99H)   (9AH)   (9BH)   (9CH)   (9DH)   (9EH)   (9FH)
          S       NS    P/PE    NP/PO   L/NGE   NL/GE   LE/NG   NLE/G

    * Eb    操作数大小是字节，位于r/m字段，是寄存器或内存数据

**操作**

根据条件设置字节值1或0： ::

    if cc {
        DEST := 1;
    } else {
        DEST := 0;
    }

**标志位**

不影响标志位。

CMOVcc
-------

条件移动

过程调用指令
============

CALL
------

过程调用： ::

    操作码              指令             操作数  模式    简要描述
            E8 cw       CALL rel16          D   N.S./V  近调用，偏移相对于下一条指令，16位有符号偏移
            E8 cd       CALL rel32          D   V/V     近调用，偏移相对于下一条指令，32位有符号偏移
            FF /2       CALL r/m16          M   N.E./V  近调用，绝对间接地址 r/m16
            FF /2       CALL r/m32          M   N.E./V  近调用，绝对间接地址 r/m32
            FF /2       CALL r/m64          M   V/N.E.  近调用，绝对间接地址 r/m64
            9A cd       CALL ptr16:16       D   Inv./V  远调用，绝对直接地址，段和偏移给定直接地址
            9A cp       CALL ptr16:32       D   Inv./V  远调用，绝对直接地址，段和偏移给定直接地址
            FF /3       CALL m16:16         M   V/V     远调用，绝对间接地址 m16:16
            FF /3       CALL m16:32         M   V/V     远调用，绝对间接地址 m16:32
    REX.W + FF /3       CALL m16:64         M   V/N.E.  远调用，绝对间接地址 m16:64

    * cw 操作码之后跟随2字节代码偏移
    * cd 操作码之后跟随4字节代码偏移，并可能为代码段寄存器指定新的段
    * cp 操作码之后跟随6字节代码偏移，并可能为代码段寄存器指定新的段

    类型    操作数1             操作数2         操作数3     操作数4
    D       Offset              无              无          无
    M       ModRM:r/m (r)       无              无          无

    CALL 调用（非64位模式）
    段内直接调用                            1110 1000 : signed full disp
    段内间接调用                            1111 1111 : 11 010 reg
    段内间接调用                            1111 1111 : mm 010 mem
    跨段直接调用                            1001 1010 : 选择器+无符号偏移
    跨段间接调用                            1111 1111 : mm 011 r/m

    CALL 调用（64位模式）
    段内直接调用                            1110 1000 : disp32
    段内间接调用                0100 WR00↑w 1111 1111 : 11 010 reg
    段内间接调用                0100 W0XB↑w 1111 1111 : mm 010 mem
    跨段间接调用                            1111 1111 : mm 011 r/m
    跨段间接调用                0100 10XB : 1111 1111 : mm 011 r/m

    CALL   (9AH)   (E8H)
          far↑i64 near↑f64
            Ap      Jz

    CALL   (FFH)   mm 010 r/m   mm 011 mem
                    near↑f64     far CALL
                       Ev           Ep

    * ↑w    REX.W 没有效果
    * ↑i64  该指令在 64 位模式下无效或无法编码
    * ↑f64  在 64 位模式下，操作数大小被强制为 64 位操作数大小
    * Ap    操作数大小根据属性决定，32/48/80位指针，位于偏移字段是直接地址
    * Jz    操作数大小对于16位是两字节，对于32/64位是四字节，位于偏移字段，是相对指令指针的偏移
    * Ev    操作数大小根据属性决定，位于r/m字段，是寄存器或内存数据
    * Ep    操作数大小根据属性决定，32/48/80位指针，位于r/m字段，是寄存器或内存数据

**操作**

该指令将过程链接信息保存到栈上，并跳转到目标操作数指定的被调过程位置。目标操作数指定了被
调过程的第一条指令的地址。该操作数可以是立即数、通用寄存器、内存位置。该指令可以执行四种
不同类型的调用：

1. 近调用（near call）- 段内调用，调用当前代码段（由 CS 寄存器指向的段）中的程序
2. 远调用（far call）- 跨段调用，调用另一个代码段中的程序
3. 跨特权级别远调用 - 远调用一个不同特权级别代码段中的程序
4. 任务切换（task switch）- 远调用一个位于不同任务中的程序

跨特权级别远调用和任务切换只能在保护模式下执行。有关近调用、远调用、跨特权级别远调用，参
考卷1第6章使用CALL和RET调用过程。有关使用CALL指令执行任务切换的信息，参考卷3第8章任务管
理。

近调用。在执行近调用时，处理器会将 EIP 寄存器的值推入栈中，然后处理器跳转到当前代码段中
由目标操作数指定的地址。目标操作数指定了代码段内的绝对偏移量（从代码段基地址开始的偏移量）
或相对偏移量（相对于当前指令指针的有符号偏移）。在近调用中，CS 寄存器的值不会被改变。

对于绝对近调用，是通过寄存器或内存位置（r/m16 r/m32 r/m64）间接指定绝对偏移量。操作数
大小属性决定目标操作数的大小（16/32/64）。在64位模式下，近调用以及所有近跳转的操作数大
小被强制为64位。当使用栈指针作为基寄存器指定内存操作数时，使用的基值是指令执行前栈指针
的值。

对于相对近调用（rel16 rel32），通常对应的是汇编语言中的标签。但在机器代码级别，它被编码
为有符号的16位或32位立即值，这个值被加到指令指针上。在64位模式下，相对偏移量始终是一个
32位立即值，在加到RIP之前会符号扩展到64位。

实地址模式或虚拟8086模式下的远调用。处理器会将当前 CS 和 EIP 寄存器的值推入栈中，用作
返回指令指针。然后处理器执行一个远跳转转移到目标操作数指定的代码段被调程序位置。目标操作
数指定了一个绝对地址，可以使用指针（ptr16:16 ptr16:32）直接指定，或通过内存位置（m16:16
m16:32）间接指定。使用指针方式时，被调过程的段和偏移量使用4字节（16位操作数）或6字节（32
位操作数）立即值编码在指令中。使用间接方式时，目标操作数指定了一个包含4字节或6字节的内存
位置。远地址直接加载到 CS 和 EIP 寄存器中。如果操作数大小属性为16，EIP寄存器的高两字节
被清零。

保护模式下的远调用。保护模式下，处理器总是使用远地址的段选择器部分来访问 GDT 或 LDT 中
对应的描述符。描述符的类型（代码段、调用门、任务门、TSS）和访问权限决定了要执行的调用操
作类型。

如果选定的描述符是代码段，执行相同特权级别代码段的远调用。保护模式下，相同特权级的远调用
与实地址或虚拟8086模式下的远调用相似。目标操作数指定了绝对远地址，可以使用直接指针指定，
也可以使用间接内存位置指定。操作数大小属性决定了源地址中编译量的大小（16或32位）。新的代
码段选择器以及描述符被加载到 CS 寄存器中，指令的偏移量被加载到 EIP 寄存器中。调用门（call
gate）也可以用来执行相同特权级别代码段的远调用。使用这种机制提供了额外的间接级别，是进行
16位和32位代码段之间调用的首选方式。

执行跨特权级别远调用时，被调过程的代码段必须通过调用门来访问。目标操作数指定的段选择器标
识的是一个调用门。目标操作数可以使用直接指针或间接内存位置来地中调用门选择器。处理器从调
用门描述符中获取新代码段的段选择器和新的指令指针（偏移量）。当使用调用门时，目标操作数的
偏移量被忽略。

在跨特权级别调用中，处理器切换到被调过程特权级别过程栈中。新栈段的段选择器在当前运行任务
的 TSS 中指定。跳转到新代码段发生在栈切换之后。注意当使用调用门执行相同特权级别的原调用
时，不发生栈切换。在新栈上，处理器推入主调过程栈的段选择器和栈指针，从主调过程复制过程参
数，以及推入主调过程代码段的段选择器和指令指针，这里调用门描述符中的一个值决定了复制多少
参数到新栈。最后，处理器跳转到新代码段被调过程的地址开始执行。

使用CALL指令执行任务切换类似于通过调用门执行调用，目标操作数指定了需要激活的新任务对应
任务门的段选择器，目标操作数中的偏移量被忽略。任务门反过来执行新任务的 TSS，其中包含任务
的代码和栈段的段选择器。注意 TSS 还包含被调任务被挂起之前本应该执行的下一条指令的指令指
针，这个指令指针值被加载到 EIP 寄存器中以重新启动调用任务。

CALL 指令还可以直接指定 TSS 的段选择器，着消除了任务门的间接性。有关任务切换的机制，参
考卷3第8章任务管理。当使用 CALL 指令执行任务切换时，在 EFLAGS 寄存器中设置任务嵌套标志
NT，并将新 TSS 的前一个任务链接字段加载为旧任务的 TSS 选择器。代码预期通过执行 IRET 指
令来挂起次嵌套任务，由于设置了 NT 标志，IRET 指令自动使用前一个任务链接返回到主调任务。
这里使用 CALL 指令切换任务与 JMP 指令不同，JMP 不设置 NT 标志，因此不期望 IRET 指令来
挂起任务。

混合16位和32位调用时，使用调用门执行16位和32位代码段直接的远调用。如果远调用是从32位代码
段到16位代码段，调用应该从32位代码段的前64KB进行，这是因为指令的操作数大小属性设置位16，
因此只能保存16位返回地址偏移量。此外，调用应该使用16位调用门，以便在栈上推入16位值。有关
更多信息，参考卷3第22章混合16位和32位代码。

兼容模式下的远调用。当处理器在兼容模式下运行时，CALL指令可以用来执行以下类型的远调用：

1. 相同特权级别的远调用，保持在兼容模式
2. 相同特权级别的远调用，转换到64位模式
3. 跨特权级别远调用，转换到64位模式

注意在 IA-32e 模式下不支持任务切换，因此不能使用CALL指令在兼容模式下执行任务切换。在兼
容模式下，处理器总是使用远地址的段选择器部分来访问 GDT 或 LDT 中相应的描述符。描述符类型
（代码段、调用门）和访问权限决定了要执行的调用操作类型。

如果选定的描述符是代码段，执行相同特权级别的远调用。在兼容模式下，相同特权级别的远调用与
在保护模式下执行的远调用非常相似。目标操作数使用直接指针或间接内存位置指定绝对远地址。操
作数大小属性决定了远地址中偏移量的大小（16或32位）。新的代码段选择器及其描述符被加载到
CS 寄存器中，指令的偏移量被加载到 EIP 寄存器中。不同之处在于可以进入64位模式，这由新代码
段描述符中的 L 位指定。64位调用门也可以用来执行相同特权级代码段的远调用，但是使用这种机
制要求目标代码段描述符设置 L 位，从而进入 64 位模式。

当执行跨特权级别远调用时，被调过程的代码段必须通过64位调用门来访问。目标操作数指定的段选
择器标识了调用门。目标操作数只能间接用内存位置（m16:16 m16:32 m16:64）指定调用门段选择
器。处理器从16字节调用门描述符中获取新代码段的段选择器和新的指令指针（偏移量）。当使用调
用门时，目标操作数的偏移量被忽略。

在跨特权级别调用中，处理器切换到被调过程特权级过程栈，新栈段的段选择器设置为NULL，新栈指
针在当前运行任务的 TSS 中指定。跳转到新代码段发生在栈切换之后。注意当使用调用门执行相同
特权级别远调用时，由于进入64位模式，会发生隐式栈切换。SS 选择器保持不变，单栈段访问使用
段基址 0x0，忽略限制，并默认栈大小为64位，RSP 的完整值用于偏移量，而它的高32位值未定义。
在新栈上，处理器推入主调过程的栈段选择器和栈指针，以及主调过程的代码段选择器和指令指针，
在IA-32e模式下不支持参数复制。最后，处理器跳转到新代码段被调过程的地址位置。

64位模式下的远调用。当处理器在64位模式下运行时，CALL 指令可以用来执行以下类型的远调用：

1. 相同特权级别的远调用，切换位兼容模式
2. 相同特权级别的远调用，保持在64位模式
3. 跨特权级别远调用，保持在64位模式

注意 IA-32e 模式不支持任务切换，因此 64 位模式 CALL 指令不能用来执行任务切换。在 64 位
模式下，处理器总是使用远地址的段选择器部分来访问 GDT 或 LDT 中相应的描述符。描述符类型
（代码段、调用门）和访问权限决定了要执行的调用操作类型。

如果选定的描述符是代码段，执行相同特权级别的远调用。在64位模式下，相同特权级的远调用与在
兼容模式下执行的远调用非常相似。目标操作数使用间接内存位置（m16:16 m16:32 m16:64）指定
绝对远地址。CALL 指令的直接指定绝对远地址的形式在 64 位模式下未定义。操作数大小属性决定
了远地址中偏移量的大小（16/32/64）。新的代码段选择器及其描述符被加载到 CS 寄存器中，指
令的偏移量被加载到指令寄存器中，新的代码段可以使用 L 位的值指定进入兼容模式或 64 位模式。
64位调用门也可以用来执行相同特权级代码段的远调用，但是使用这种机制要求目标代码段描述符设
置 L 位。

当执行跨特权远调用时，被调过程的代码段必须通过64位调用门来访问。目标操作数指定的段选择器
标识了调用门。目标操作数只能使用间接内存位置指定调用门段选择器。处理器从16字节调用门描述
符中获取新代码段的段选择器和新的指令指针（偏移量）。当使用调用门时，目标操作数的偏移量被
忽略。

在跨特权级别调用中，处理器切换到被调过程特权级过程栈，新栈段的段选择器设置为 NULL，新栈
指针在当前运行任务的 TSS 中指定。跳转到新代码段发生在栈切换之后。注意当使用调用门执行相
同特权级别的远调用时，由于进入64为模式，会发生隐式栈切换。SS 选择器保持不变，但栈段访问
使用段基址 0x0，忽略限制，并默认栈大小为64位，RSP 的完整值用于偏移量。在新栈上，处理器
推入主调过程的栈段选择器和栈指针，以及主调过程代码段选择器和指令指针，在 IA-32e 模式下
不支持参数复制。最后，处理器跳转到新代码段被调过程的地址位置。

有关 CET（控制流执行技术）的详细信息，参考卷1第6章过程调用、中断和异常，以及第17章控制
流执行技术。

指令顺序。在远调用之后的指令可能在较早指令完成执行之前从内存中获取，但它们不会执行，直到
所有远调用之前的指令完成执行，但可能在之前的指令产生的数据完成存储并全局可见之前执行。而
间接近调用之后的指令可能被推测执行，如果软件需要防止这种情况，可以在间接近调用指令之后放
置一个 LFENCE 指令操作码，以阻止推测执行。

**标志位**

如果发生了任务切换所有标志位都受影响，如果没有任务切换不影响标志位。

RET
-----

调用返回： ::

    操作码              指令             操作数  模式    简要描述
    C2 iw               RET imm16           I   V/V     近返回到主调过程并弹出imm16个字节
    C3                  RET                 ZO  V/V     近返回到主调过程
    CA iw               RET imm16           I   V/V     远返回到主调过程并弹出imm16个字节
    CB                  RET                 ZO  V/V     远返回到主调过程

    类型    操作数1             操作数2         操作数3     操作数4
    ZO      无                  无              无          无
    I       imm16               无              无          无

    RET 返回（非64位模式）
    段内返回无参数              1100 0011
    段内返回有参数              1100 0010 : disp16
    跨段返回无参数              1100 1011
    跨段返回有参数              1100 1010 : disp16

    RET 返回（64位模式）
    段内返回无参数              1100 0011
    段内返回有参数              1100 0010 : disp16
    跨段返回无参数              1100 1011
    跨段返回有参数              1100 1010 : disp16

    RET    (C2H)    (C3H)
          near↑f64  near↑f64
            Iw

    RET    (CAH)    (CBH)
          far RET   far RET
            Iw

    * ↑f64  在 64 位模式下，操作数大小被强制为 64 位操作数大小
    * Iw    操作数大小是两字节，位于立即数字段

**操作**

将程序控制权转移到栈顶的返回地址对应的位置，该地址通常是 CALL 指令压入的函数返回地址，
是 CALL 指令之后一条指令的地址。可选的操作数，指定在返回后需要弹出的数据字节数，可以
用于弹出固定数量的函数参数。该指令可以执行三种不同类型的返回：

1. 近返回（近返回）- 返回到当前代码段中的一个过程，CS 段寄存器指向当前代码段
2. 远返回（far return）- 返回到另一个代码段中的一个过程
3. 跨特权级别远返回 - 远返回到一个不同特权级别过程，仅用于保护模式

指令顺序。在远返回指令之后，后续的指令可能在之前的指令完成执行之前就被预取，但它们不会执
行，直到所有远返回之前的指令都已执行完毕，但可以在之前指令产生的数据存储完毕全局可见之前
执行。与近间接调用和近间接跳转不同，处理器不会推测执行近返回指令之后的下一条顺序指令，除
非该指令也是跳转或分支预测的目标。

**标志位**

不影响标志位。

ENTER
------

过程进入： ::

    操作码              指令             操作数  模式    简要描述
    C8 iw 00            ENTER imm16, 0      II  V/V     为过程创建栈帧
    C8 iw 01            ENTER imm16, 1      II  V/V     为过程创建栈帧有一个嵌套帧指针
    C8 iw ib            ENTER imm16, imm8   II  V/V     为过程创建栈帧有多个嵌套帧指针

    类型    操作数1             操作数2         操作数3     操作数4
    II      iw                  imm8            无          无

    ENTER 进入
    1100 1000 : disp16 : imm8

    ENTER (C8H)
          Iw,Ib

    * Iw    操作数大小是两字节，位于偏移量字段
    * Ib    操作数大小是单字节，位于立即数字段

**操作**

该指令为过程创建一个栈帧，包括动态存储空间和 1 ~ 32 个帧指针存储。第一个操作数指定栈帧
动态存储的大小，即在栈上动态分配的字节数。第二个操作数给出过程的语法嵌套级别（0到31），
嵌套级别（imm8 mod 32）和操作数大小属性确定帧指针存储空间的大小。

嵌套级别决定了从前面的栈复制多少帧指针到新栈帧的显示区，默认的帧指针大小是 StackAddrSize，
但可以使用 66H 前缀。因此操作数大小属性决定了将被复制到栈帧中每个帧指针的大小，以及从
SP/ESP/RSP 传输到 BP/EBP/RBP 寄存器的数据大小。

ENTER 指令和 LEAVE 指令配套用于支持块结构化语言。ENTER 指令通常是过程的第一个指令，用于
为过程创建一个栈帧，然后在过程的末尾（就在 RET 指令之前）使用 LEAVE 指令来释放栈帧。

如果嵌套级别为0，则处理器将 BP/EBP/RBP 帧指针压入栈，将当前的栈指针从 SP/ESP/RSP 复制
到 BP/EBP/RBP，并将 SP/ESP/RSP 寄存器加载为当前栈指针减去操作数大小对应的值。对于嵌套
级别为1或更高的情况，处理器在调整栈指针之前将额外的帧指针压入栈。这些额外的帧指针为过程
提供了访问栈上其他嵌套帧的接入点。有关 ENTER 指令操作的更多信息，参考卷1第6章过程调用、
中断和异常中的用于块结构语言的过程调用。

在64位模式下，默认操作数大小为64位，无法编码为32位操作数大小。使用66H前缀可以将帧指针操
作数大小更改为16位。当使用66H前缀导致操作数大小小于 StackAddrSize 时，软件需要确保以下
两点：

1. 对应的 LEAVE 指令也使用 66H 前缀
2. 在执行 66H ENTER 之前，RBP/EBP 中的值必须在当前栈指针（RSP/ESP）16KB 区域内，以便
   在 66H ENTER 之后 RBP/EBP 的值仍然是栈中的有效地址，着确保 66H LEAVE 可以从栈中恢
   复 16 位数据

对应伪代码如下： ::

    AllocSize := imm16
    NestingLevel := imm8 MOD 32
    Push(RBP/EBP/BP)
    Frame := RSP/ESP/SP
    if NestingLevel = 0 {
        goto Continue;
    }
    if NestingLevel > 1 {
        Do NestingLevel - 1 times {
            if OperandSize = 64 {
                RBP -= 8;
                Push([RBP])
            } else if OperandSize = 32 {
                if StackSize = 32 {
                    EBP -= 4;
                    Push([EBP]); 压入四字节
                } else StackSize = 16 {
                    BP -= 4;
                    Push([BP]); 压入四字节
                }
            } else OperandSize = 16 {
                if StackSize = 64 {
                    RBP -= 2;
                    Push([RBP]); 压入两字节
                } else if StackSize = 32 {
                    EBP -= 2;
                    Push([EBP]); 压入两字节
                } else StackSize = 16 {
                    BP -= 2;
                    Push([BP]); 压入两字节
                }
            }
        }
    }
    Push(Frame)
    Continue:
    RBP/EBP/BP := Frame
    RSP/ESP/SP -= AllocSize

**标志位**

不影响标志位。

LEAVE
------

过程退出： ::

    操作码              指令             操作数  模式    简要描述
    C9                  LEAVE               ZO  V/V     将帧指针设置到 SP，并弹出恢复帧指针 BP
    C9                  LEAVE               ZO  N.E./V  将帧指针设置到ESP，并弹出恢复帧指针EBP
    C9                  LEAVE               ZO  V/N.E.  将帧指针设置到RSP，并弹出恢复帧指针RBP

    类型    操作数1             操作数2         操作数3     操作数4
    ZO      无                  无              无          无

    LEAVE 退出
    1100 1001

    LEAVE↑d64 (C9H)

    * ↑d64 在64位模式下，指令默认为64位操作数大小，无法编码为32位操作数大小

**操作**

该指令操作如下： ::

    if StackAddrSize = 32 {
        ESP := EBP;
    } else if StackAddrSize = 64 {
        RSP := RBP;
    } else if StackAddrSize = 16 {
        SP := BP;
    }
    if OperandSize = 32 {
        EBP := Pop();
    } else if OperandSize = 64 {
        RBP := Pop();
    } else if OperandSize = 16 {
        BP := Pop();
    }

**标志位**

不影响标志位。
