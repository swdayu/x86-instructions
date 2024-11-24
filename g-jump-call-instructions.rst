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

    JMP (FFH)   mm 100 r/m      mm 101 r/m
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
4. 任务切换（task switch）- 跳转到另一个任务的一个指令位置

任务切换仅能在保护模式执行，参见第3卷第8章。近跳转的目标操作数可以是一个绝对偏移，或者
相对偏移，相对偏移通常对应为汇编代码中的一个标签，但是在机器代码级别，它是一个8/16/32
位的有符号立即数，然后加到当前的指令指针寄存器上。当前的指令指针的值，表示的是当前JMP
指令之后的那个指令的地址。

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
            E8 cw       CALL rel16          D   N.S./V  Call near, relative, displacement relative to next
    instruction.
            E8 cd       CALL rel32          D   V/V     Call near, relative, displacement relative to next
    instruction. 32-bit displacement sign extended to
    64-bits in 64-bit mode.
            FF /2       CALL r/m16          M   N.E./V  Call near, absolute indirect, address given in r/m16.
            FF /2       CALL r/m32          M   N.E./V  Call near, absolute indirect, address given in r/m32.
            FF /2       CALL r/m64          M   V/N.E.  Call near, absolute indirect, address given in r/m64.
            9A cd       CALL ptr16:16       D   I/V     Call far, absolute, address given in operand.
            9A cp       CALL ptr16:32       D   I/V     Call far, absolute, address given in operand.
            FF /3       CALL m16:16 M V/V Call far, absolute indirect address given in m16:16.
    In 32-bit mode: if selector points to a gate, then RIP
    = 32-bit zero extended displacement taken from
    gate; else RIP = zero extended 16-bit offset from
    far pointer referenced in the instruction.
            FF /3       CALL m16:32         M   V/V     In 64-bit mode: If selector points to a gate, then RIP
    = 64-bit displacement taken from gate; else RIP =
    zero extended 32-bit offset from far pointer
    referenced in the instruction.
    REX.W + FF /3       CALL m16:64         M   V/N.E.  In 64-bit mode: If selector points to a gate, then RIP
    = 64-bit displacement taken from gate; else RIP =
    64-bit offset from far pointer referenced in the
    instruction. 

    类型    操作数1             操作数2         操作数3     操作数4
    D       Offset              无              无          无
    M       ModRM:r/m (r)       无              无          无

    CALL – Call Procedure (in same segment)
    direct 1110 1000 : full displacement
    register indirect 1111 1111 : 11 010 reg
    memory indirect 1111 1111 : mod 010 r/m
    CALL – Call Procedure (in other segment)
    direct 1001 1010 : unsigned full offset, selector
    indirect 1111 1111 : mod 011 r/m

    CALL – Call Procedure (in same segment)
    direct 1110 1000 : displacement32
    register indirect 0100 WR00w 1111 1111 : 11 010 reg
    memory indirect 0100 W0XBw 1111 1111 : mod 010 r/m
    CALL – Call Procedure (in other segment)
    indirect 1111 1111 : mod 011 r/m
    indirect 0100 10XB 0100 1000 1111 1111 : mod 011 r/m



**操作**

该指令可以执行四种不同类型的调用：

1. 近跳转（near jump）- 段内跳转，该段为 CS 段寄存器当前指向的代码段
2. 短跳转（short jump）- 跳转范围限制在 -128 到 127 范围内，相对于当前的指令指针
3. 远跳转（far jump）- 跨段跳转，跳转到拥有相同特权等级的另外一个代码段的一个指令位置
4. 任务切换（task switch）- 跳转到另一个任务的一个指令位置

任务切换仅能在保护模式执行，参见第3卷第8章。近跳转的目标操作数可以是一个绝对偏移，或者
相对偏移，相对偏移通常对应为汇编代码中的一个标签，但是在机器代码级别，它是一个8/16/32
位的有符号立即数，然后加到当前的指令指针寄存器上。当前的指令指针的值，表示的是当前JMP
指令之后的那个指令的地址。

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

    RET – Return from Procedure (to same segment)
    no argument 1100 0011
    adding immediate to SP 1100 0010 : 16-bit displacement
    RET – Return from Procedure (to other segment)
    intersegment 1100 1011
    adding immediate to SP 1100 1010 : 16-bit displacement

    RET – Return from Procedure (to same segment)
    no argument 1100 0011
    adding immediate to SP 1100 0010 : 16-bit displacement
    RET – Return from Procedure (to other segment)
    intersegment 1100 1011
    adding immediate to SP 1100 1010 : 16-bit displacement

    near RET↑f64 (C2H)  (C3H)
                  Iw

    far RET (CAH)   (CBH)
             Iw

    * ↑f64  在 64 位模式下，操作数大小被强制为 64 位操作数大小
    * Iw    操作数大小是两字节，位于立即数字段

**操作**

该指令可以执行三种不同类型的返回：

1. 近返回（近返回）- 返回到当前代码段中的一个过程，CS 段寄存器指向当前代码段
2. 远返回（far return）- 返回到另一个代码段中的一个过程
3. 跨特权级别远返回（inter-privilege-level far return）- 远返回到一个不同特权级别过程

任务切换仅能在保护模式执行，参见第3卷第8章。近跳转的目标操作数可以是一个绝对偏移，或者
相对偏移，相对偏移通常对应为汇编代码中的一个标签，但是在机器代码级别，它是一个8/16/32
位的有符号立即数，然后加到当前的指令指针寄存器上。当前的指令指针的值，表示的是当前JMP
指令之后的那个指令的地址。

**标志位**

不影响标志位。

ENTER
------

过程进入： ::

    操作码              指令             操作数  模式    简要描述
    C8 iw 00            ENTER imm16, 0      II  V/V     Create a stack frame for a procedure.
    C8 iw 01            ENTER imm16, 1      II  V/V     Create a stack frame with a nested pointer for
    a procedure.
    C8 iw ib            ENTER imm16, imm8   II  V/V     Create a stack frame with nested pointers for
    a procedure.

    类型    操作数1             操作数2         操作数3     操作数4
    II      iw                  imm8            无          无

    ENTER – Make Stack Frame for High Level Procedure 1100 1000 : 16-bit displacement : 8-bit level (L)

    ENTER (C8H)
          Iw,Ib

    * Iw    操作数大小是两字节，位于立即数字段
    * Ib    操作数大小是单字节，位于立即数字段

**操作**

**标志位**

不影响标志位。

LEAVE
------

过程退出： ::

    操作码              指令             操作数  模式    简要描述
    C9                  LEAVE               ZO  V/V     Set SP to BP, then pop BP.
    C9                  LEAVE               ZO  N.E./V  Set ESP to EBP, then pop EBP.
    C9                  LEAVE               ZO  V/N.E.  Set RSP to RBP, then pop RBP.

    类型    操作数1             操作数2         操作数3     操作数4
    ZO      无                  无              无          无

    LEAVE – High Level Procedure Exit 1100 1001
    LEAVE – High Level Procedure Exit 1100 1001

    LEAVE↑d64 (C9H)

    * ↑f64  在 64 位模式下，操作数大小被强制为 64 位操作数大小

**操作**

    IF StackAddressSize = 32
    THEN
    ESP := EBP;
    ELSE IF StackAddressSize = 64
    THEN RSP := RBP; FI;
    ELSE IF StackAddressSize = 16
    THEN SP := BP; FI;
    FI;
    IF OperandSize = 32
    THEN EBP := Pop();
    ELSE IF OperandSize = 64
    THEN RBP := Pop(); FI;
    ELSE IF OperandSize = 16
    THEN BP := Pop(); FI;
    FI;

**标志位**

不影响标志位。
