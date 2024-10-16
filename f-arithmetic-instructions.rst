算术运算指令

加法运算
=========

ADD
----

整数加法： ::

    操作码              指令             操作数  模式    简要描述
    04 ib               ADD AL, imm8        I   V/V     imm8加到AL
    05 iw               ADD AX, imm16       I   V/V     imm16加到AX
    05 id               ADD EAX, imm32      I   V/V     imm32加到EAX
    REX.W + 05 id       ADD RAX, imm32      I   V/N.E.  imm32符号扩展加到RAX
    80 /0 ib            ADD r/m8, imm8      MI  V/V     imm8加到r/m8
    REX + 80 /0 ib      ADD r/m8*, imm8     MI  V/N.E.  imm8符号扩展加到r/m8
    81 /0 iw            ADD r/m16, imm16    MI  V/V     imm16加到r/m16
    81 /0 id            ADD r/m32, imm32    MI  V/V     imm32加到r/m32
    REX.W + 81 /0 id    ADD r/m64, imm32    MI  V/N.E.  imm32符号扩展加到r/m64
    83 /0 ib            ADD r/m16, imm8     MI  V/V     imm8符号扩展加到r/m16
    83 /0 ib            ADD r/m32, imm8     MI  V/V     imm8符号扩展加到r/m32
    REX.W + 83 /0 ib    ADD r/m64, imm8     MI  V/N.E.  imm8符号扩展加到r/m64
    00 /r               ADD r/m8, r8        MR  V/V     r8加到r/m8
    REX + 00 /r         ADD r/m8*, r8*      MR  V/N.E.  r8加到r/m8
    01 /r               ADD r/m16, r16      MR  V/V     r16加到r/m16
    01 /r               ADD r/m32, r32      MR  V/V     r32加到r/m32
    REX.W + 01 /r       ADD r/m64, r64      MR  V/N.E.  r64加到r/m64
    02 /r               ADD r8, r/m8        RM  V/V     r/m8加到r8
    REX + 02 /r         ADD r8*, r/m8*      RM  V/N.E.  r/m8加到r8
    03 /r               ADD r16, r/m16      RM  V/V     r/m16加到r16
    03 /r               ADD r32, r/m32      RM  V/V     r/m32加到r32
    REX.W + 03 /r       ADD r64, r/m64      RM  V/N.E.  r/m64加到r64

    * 在64位模式下，当使用了 REX 前缀，r/m8 不能被编码成 AH BH CH DH 寄存器

    类型    操作数1             操作数2         操作数3     操作数4
    RM      ModR/M:reg(r,w)     ModR/M:r/m(r)    无          无
    MR      ModR/M:r/m(r,w)     ModR/M:reg(r)    无          无
    MI      ModR/M:r/m(r,w)     imm8/16/32       无          无
    I       AL/AX/EAX/RAX       imm8/16/32       无          无

将目标操作数（第一个操作数）与源操作数（第二个操作数）相加，然后将结果存储在目标操作数中。
目标操作数可以是寄存器或内存位置；源操作数可以是立即数、寄存器或内存位置（但一条指令中不
能使用两个内存操作数）。当立即数用作操作数时，会将其符号扩展到目标操作数格式的长度。

ADD 指令执行整数加法。它求值有符号和无符号整数操作数的结果，并设置 OF 和 CF 标志以指示
有符号或无符号结果中的进位（溢出）。SF 标志指示有符号结果的符号。可以使用 LOCK 前缀执行
此指令，以允许指令原子地执行。

在 64 位模式下，指令的默认操作大小为 32 位。使用形式为 REX.R 的 REX 前缀允许访问额外的
寄存器（R8-R15）。使用形式为 REX.W 的 REX 前缀将操作提升到 64 位。参见上表中的编码数据
和限制。

**操作**

DEST := DEST + SRC;

**标志位**

影响 OF、SF、ZF、AF、CF、PF 标志。

**异常**

兼容或保护模式：

- #GP(0)：如果目标位于不可写的段中。如果内存操作数的有效地址在 CS、DS、ES、FS 或 GS 段
  的限制之外。如果使用 DS、ES、FS 或 GS 寄存器访问内存，而它包含一个空的段选择器。
- #SS(0)：如果内存操作数的有效地址在 SS 段的限制之外。
- #PF(fault-code)：如果发生页面错误。
- #AC(0)：如果启用了对齐检查，并且在当前特权级别为 3 时进行了未对齐的内存引用。
- #UD：如果使用了 LOCK 前缀，但目标不是内存操作数。

64位模式：

- #SS(0)：如果引用 SS 段的内存地址不是规范形式（canonical form）。在 64 位模式下，规
  范形式的地址必须具有特定的特征，例如，它不能引用到前 64KB 的物理内存。
- #GP(0)：如果内存地址不是规范形式。
- #PF(fault-code)：如果发生页面错误。这可能由于多种原因，例如尝试访问未映射的内存区域或
  访问权限不足。
- #AC(0)：如果启用了对齐检查，并且在当前特权级别为 3 时进行了未对齐的内存引用。在 64 位
  模式下，某些指令要求特定的对齐，例如某些 SSE 指令要求 16 字节对齐。
- #UD：如果使用了 LOCK 前缀，但目标不是内存操作数。

**使用示例**

指令格式，qwreg 表示 64 位寄存器： ::

    ADD – Add
    reg to reg2                             0000 000w : 11 reg reg2
    reg to mem                              0000 000w : mod reg r/m
    reg2 to reg                             0000 001w : 11 reg reg2
    mem to reg                              0000 001w : mod reg r/m
    imm to AL, AX, or EAX                   0000 010w : imm
    imm to reg                              1000 00sw : 11 000 reg : imm
    imm to mem                              1000 00sw : mod 000 r/m : imm

    ADD – Add (64-bit Mode)
    reg to reg2                 0100 0R0B : 0000 000w : 11 reg reg2
    qwreg to qwreg2             0100 1R0B : 0000 0000 : 11 qwreg qwreg2
    qwreg to qwreg2             0100 1R0B : 0000 0010 : 11 qwreg qwreg2
    reg to mem                  0100 0RXB : 0000 000w : mod reg r/m
    qwreg to mem64              0100 1RXB : 0000 0011 : mod qwreg r/m
    reg2 to reg                 0100 0R0B : 0000 001w : 11 reg reg2
    mem to reg                  0100 0RXB : 0000 001w : mod reg r/m
    mem64 to qwreg              0100 1RXB : 0000 0000 : mod qwreg r/m
    imm to AL, AX, or EAX                   0000 010w : imm8
    imm to RAX                  0100 1000 : 0000 0101 : imm32
    imm to reg                  0100 000B : 1000 00sw : 11 000 reg : imm
    imm32 to qwreg              0100 100B : 1000 0001 : 11 010 qwreg : imm
    imm to mem                  0100 00XB : 1000 00sw : mod 000 r/m : imm
    imm32 to mem64              0100 10XB : 1000 0001 : mod 010 r/m : imm32
    imm8 to mem64               0100 10XB : 1000 0011 : mod 010 r/m : imm8

    表-2：
    |    00     |    01     |    02     |    03     |    04     |    05     |
    |ADD                                                                    |
    |   Eb,Gb       Ev,Gv       Gb,Eb       Gv,Ev       AL,Ib       rAX,Iz  |

    |    80     |    81     |           |    83     |
    |Immediate Grp 1↑1A                             |
    | { Eb,Ib       Ev,Iz     Eb,Ib↑i64     Ev,Ib }

    表-6：
    Opcode  |Gr mod pfx | reg/opcode 5,4,3 (r/m 2,1,0 in parenthesis)
            |   7,6     |    000    |
    80 ~ 83 |1  xxB --- |ADD        |

ADD_00_01_02_03: ::

    操作码          指令                操作数          模式        简要描述
    00 /r           ADD r/m8, r8        MR              V/V         r8加到r/m8
    REX + 00 /r     ADD r/m8*, r8*      MR              V/N.E.      r8加到r/m8
    01 /r           ADD r/m16, r16      MR              V/V         r16加到r/m16
    01 /r           ADD r/m32, r32      MR              V/V         r32加到r/m32
    REX.W + 01 /r   ADD r/m64, r64      MR              V/N.E.      r64加到r/m64
    02 /r           ADD r8, r/m8        RM              V/V         r/m8加到r8
    REX + 02 /r     ADD r8*, r/m8*      RM              V/N.E.      r/m8加到r8
    03 /r           ADD r16, r/m16      RM              V/V         r/m16加到r16
    03 /r           ADD r32, r/m32      RM              V/V         r/m32加到r32
    REX.W + 03 /r   ADD r64, r/m64      RM              V/N.E.      r/m64加到r64
    类型            操作数1             操作数2         操作数3     操作数4
    MR              ModR/M:r/m(r,w)     ModR/M:reg(r)    无          无
    RM              ModR/M:reg(r,w)     ModR/M:r/m(r)    无          无
    操作码映射
    ADD (00H)      (01H)               (02H)           (03H)
        Eb,Gb       Ev,Gv               Gb,Eb           Gv,Ev
    * /r 表示 ModR/M 字节包含 reg 操作数和 r/m 操作数
    * r8 r16 r32 r64 表示操作数是 8/16/32/64 位寄存器
    * r/m8 r/m16 r/m32 r/m64 表示操作数是 8/16/32/64 位寄存器或者内存数据
    * 在64位模式下，当使用了 REX 前缀，r/m8 不能被编码成 AH BH CH DH 寄存器
    * Eb,Gb 表示第一个操作数是 Eb，第二个操作数是 Gb，两个操作数的大小都是字节（b），忽略操作数大小属性
    * Ev,Gv 表示第一个操作数是 Ev，第二个操作数是 Gv，两个操作数的大小根据属性决定（v），是字（2B）双字（4B）或四字（8B）
    * E 表示操作数由 ModR/M 字节的 r/m 字段指定，可以是寄存器或内存操作数
    * G 表示操作数由 ModR/M 字节的 reg 字段指定，是一个寄存器操作数

    指令编码
    reg to reg2                             0000 000w : 11 reg reg2
    reg to mem                              0000 000w : mod reg r/m
    reg2 to reg                             0000 001w : 11 reg reg2
    mem to reg                              0000 001w : mod reg r/m
    reg to reg2                 0100 0R0B : 0000 000w : 11 reg reg2
    reg to mem                  0100 0RXB : 0000 000w : mod reg r/m
    reg2 to reg                 0100 0R0B : 0000 001w : 11 reg reg2
    mem to reg                  0100 0RXB : 0000 001w : mod reg r/m
    qwreg to qwreg2             0100 1R0B : 0000 0000 : 11 qwreg qwreg2
    qwreg to qwreg2             0100 1R0B : 0000 0010 : 11 qwreg qwreg2
    qwreg to mem64              0100 1RXB : 0000 0011 : mod qwreg r/m
    mem64 to qwreg              0100 1RXB : 0000 0000 : mod qwreg r/m

    [0100WRXB 76543210 mmregr/m]
              00000000 mmregmem     ADD m8,r8
              00000000 11regreg'    ADD r8',r8
     01000RXB 00000000 mmregmem     ADD m8,r8       r8 - REX.r8
     01000R0B 00000000 11regreg'    ADD r8',r8      r8 - REX.r8
     01001RXB 00000000 mmregmem     ADD m64,r64
     01001R0B 00000000 11regreg'    ADD r64',r64
          66H 00000001 mmregmem     ADD m16,r16
          66H 00000001 11regreg'    ADD r16',r16
              00000001 mmregmem     ADD m32,r32
              00000001 11regreg'    ADD r32',r32
     01000RXB 00000001 mmregmem     ADD m32,r32     r32 - REX.r8
     01000R0B 00000001 11regreg'    ADD r32',r32    r32 - REX.r32
     01001RXB 00000001 mmregmem     ADD m64,r64
     01001R0B 00000001 11regreg'    ADD r64',r64
              00000010 mmregmem     ADD r8,m8
              00000010 11regreg'    ADD r8,r8'
          66H 00000011 mmregmem     ADD r16,m16
          66H 00000011 11regreg'    ADD r16,r16'
              00000011 mmregmem     ADD r32,m32
              00000011 11regreg'    ADD r32,r32'

ADC 带进位加法
==============

对两个整型操作数相加，并且加上 1 如果 CF 被置位。

SUB 整数减法
=============

SBB 带借位减法
==============

对两个整型操作数相减，并且减去 1 如果 CF 被置位。

INC 无符号自加
==============

DEC 无符号自减
==============

在 64 位模式下，INC 和 DEC 指令是支持的。然而，由于操作码被视为 REX 前缀，因此某些形式
的 INC 和 DEC（寄存器操作数使用了 MOD R/M 字节中的寄存器扩展字段来编码）在 64 位模式下
无法编码。

在 64 位模式下，REX 前缀用于访问扩展的通用寄存器（R8 ~ R15）和修改操作数大小。如果 INC
或 DEC 指令的编码与 REX 前缀冲突，那么这些特定形式的指令将无法使用。

CMP 大小比较
=============

NEG 求取负数
=============

MUL 无符号乘法
==============

IMUL 有符号乘法
===============

DIV 无符号除法
==============

IDIV 有符号除法
===============

DAA 十进制加法
==============

DAS 十进制减法
==============

AAA 字节序加法
==============

AAS 字节序减法
==============

AAM 字节序乘法
==============

AAD 字节序除法
==============
