操作码映射表
============

在 IA-32 和 Intel 64 架构中，操作码映射表用于解释指令代码。这些指令被分为不同的编码组：

* 1 字节、2 字节和 3 字节操作码编码：用于编码整数、系统、MMX 技术、SSE/SSE2/SSE3/
  SSE3/SSE4 和 VMX 指令。这些指令的映射在表-2到表-6中给出。
* 转义操作码（格式为：转义字符、操作码、ModR/M 字节）：用于浮点指令。这些指令的映射在
  表-7到表-22中提供。

这里列出了指令的操作码（包括所需的指令前缀和相关 ModR/M 字节中的操作码扩展）。表格中的
空白单元格表示保留或未定义的操作码。标有 “Reserved-NOP” 的单元格也是保留的，但在某些处
理器上可能表现为 NOP（无操作）。软件不应使用对应于空白单元格或标有 “Reserved-NOP” 的操
作码，也不应依赖这些操作码当前的行为。

操作码映射表按操作码字节的高 4 位和低 4 位的十六进制值组织。对于 1 字节编码（表-2）：使
用操作码的四个高阶位作为操作码表的行索引；使用四个低阶位作为表的列索引。对于以 0FH 开始
的 2 字节操作码（表-3）：跳过任何指令前缀，0FH 字节（0FH 之前可能有 66H、F2H 或 F3H）；
使用下一个操作码字节的高 4 位和低 4 位值来索引表的行和列。类似地，对于以 0F38H 或 0F3AH
开始的 3 字节操作码（表-4）：跳过任何指令前缀，0F38H 或 0F3AH；使用第三个操作码字节的高
4 位和低 4 位值来索引表的行和列。

当 ModR/M 字节提供操作码扩展时，这些信息会限定操作码的执行。有关 ModR/M 字节中操作码扩
展如何修改表-2和表-3中的操作码映射的信息，请参见操作码扩展部分。

浮点指令的转义（ESC）操作码列表在每页的顶部标出操作码的八个高阶位。如果对应的 ModR/M 字
节在 00H-BFH 范围内，顶行中的 bit3~5 以及 ModR/M 的 reg 位确定操作码。超出 00H-BFH 范
围的 ModR/M 字节由每页底部的两个表映射。

缩写词说明
-----------

操作数通常由两个字符标识，形式为 Zz。第一个字符是大写字母，指定寻址方式；第二个字符是小
写字母，指定操作数的类型。

下面这些缩写用于在文档中描述寻址方式：

**A**
    直接地址：指令中没有 ModR/M 字节；操作数的地址编码在指令中。不能应用基寄存器、索引
    寄存器或缩放因子。例如远跳转 JMP (EA)。
**B**
    VEX 前缀的 VEX.vvvv 字段选择一个通用寄存器。
**C**
    ModR/M 字节的 reg 字段选择一个控制寄存器。例如 MOV (0F20, 0F22)。
**D**
    ModR/M 字节的 reg 字段选择一个调试寄存器。例如 MOV (0F21, 0F23)。
**E**
    操作码后跟一个 ModR/M 字节，指定操作数。操作数可以是通用寄存器或内存地址。如果是内
    存地址，地址由段寄存器和以下值计算得出：基寄存器、索引寄存器、缩放因子、偏移。
**F**
    EFLAGS/RFLAGS 寄存器。
**G**
    ModR/M 字节的 reg 字段选择一个通用寄存器。例如 AX (000)。
**H**
    VEX 前缀的 VEX.vvvv 字段选择一个 128 位 XMM 寄存器或 256 位 YMM 寄存器，由操作数
    类型决定。对于传统的 SSE 编码，这个操作数不存在，将指令变为破坏性形式。
**I**
    立即数数据：操作数值编码在指令的后续字节中。
**J**
    指令包含一个相对偏移量，要加到指令指针寄存器上。例如 JMP (0E9)，LOOP。
**L**
    8位立即数的高 4 位选择一个 128 位 XMM 寄存器或 256 位 YMM 寄存器，由操作数类型决
    定（在32位模式中忽略最高位）。
**M**
    ModR/M 字节只能引用内存。例如 BOUND，LES，LDS，LSS，LFS，LGS，CMPXCHG8B。
**N**
    ModR/M 字节的 R/M 字段选择一个打包的四字（quadword），MMX 技术寄存器。
**O**
    指令没有 ModR/M 字节。操作数的偏移量编码为指令中的一个字或双字（取决于地址大小属性）。
    不能应用基寄存器、索引寄存器或缩放因子。例如 MOV (A0–A3)。
**P**
    ModR/M 字节的 reg 字段选择一个打包的四字 MMX 技术寄存器。
**Q**
    操作码后跟一个 ModR/M 字节，指定操作数。操作数可以是 MMX 技术寄存器或内存地址。如果
    是内存地址，地址由段寄存器和以下值计算得出：基寄存器、索引寄存器、缩放因子和偏移。
**R**
    ModR/M 字节的 R/M 字段只能引用通用寄存器。例如 MOV (0F20-0F23)。
**S**
    ModR/M 字节的 reg 字段选择一个段寄存器。例如 MOV (8C, 8E)。
**U**
    ModR/M 字节的 R/M 字段选择一个 128 位 XMM 寄存器或 256 位 YMM 寄存器，由操作数类
    型决定。
**V**
    ModR/M 字节的 reg 字段选择一个 128 位 XMM 寄存器或 256 位 YMM 寄存器，由操作数类
    型决定。
**W**
    操作码后跟一个 ModR/M 字节，指定操作数。操作数可以是 128 位 XMM 寄存器、256 位 YMM
    寄存器（由操作数类型决定）或内存地址。如果是内存地址，地址由段寄存器和以下值计算得出：
    基寄存器、索引寄存器、缩放因子和偏移。
**X**
    由 DS:rSI 寄存器对寻址的内存。例如 MOVS, CMPS, OUTS, 或 LODS。
**Y**
    由 ES:rDI 寄存器对寻址的内存。例如，MOVS, CMPS, INS, STOS, 或 SCAS。

下面这些缩写用于在文档中描述操作数的类型：

**a**
    内存中的两个单字（word）操作数，或根据操作数大小属性，内存中的两个双字（dword）操作
    数（仅由BOUND指令使用）。
**b**
    字节，不考虑操作数大小属性。
**c**
    根据操作数大小属性，字节或字（word）。
**d**
    双字（doubleword），不考虑操作数大小属性。
**dq**
    双四字（double-quadword），不考虑操作数大小属性。
**p**
    根据操作数大小属性，32位、48位或80位指针。
**pd**
    128位或256位打包双精度浮点数据。
**pi**
    四字（quadword）MMX技术寄存器。例如 mm0。
**ps**
    128位或256位打包单精度浮点数据。
**q**
    四字（quadword），不考虑操作数大小属性。
**qq**
    四四字（quad-quadword，256位），不考虑操作数大小属性。
**s**
    6字节或10字节伪描述符。
**sd**
    128位双精度浮点数据的标量元素。
**ss**
    128位单精度浮点数据的标量元素。
**si**
    双字（doubleword）整数寄存器。例如 eax。
**v**
    根据操作数大小属性，字（word）、双字（doubleword）或四字（quadword）（在64位模式下）。
**w**
    字（word），不考虑操作数大小属性。
**x**
    根据操作数大小属性，dq或qq。
**y**
    根据操作数大小属性，双字（doubleword）或四字（quadword）（在64位模式下）。
**z**
    对于16位操作数大小，字（word）；对于32位或64位操作数大小，双字（doubleword）。

当操作码需要特定寄存器作为操作数时，会通过名称来标识这些寄存器（例如 AX、CL 或 ESI）。
寄存器的名称表明了它是 64 位、32 位、16 位还是 8 位宽。

当寄存器宽度取决于操作数大小属性时，使用形式为 eXX 或 rXX 的寄存器标识符。eXX 用于可能
存在 16 位或 32 位大小的情况；rXX 用于可能存在 16 位、32 位或 64 位大小的情况。例如：
eAX 表示当操作数大小属性为 16 位时使用 AX 寄存器，当操作数大小属性为 32 位时使用 EAX
寄存器。rAX 可以表示 AX、EAX 或 RAX。

当使用 REX.B 位修改操作码中 reg 字段指定的寄存器时，这一事实通过在寄存器名称中添加 “/x”
来表示额外的可能性。例如，rCX/r9 用于表示寄存器可以是 rCX 或 r9。请注意，在这种情况下，
r9 的大小由操作数大小属性决定（与 rCX 相同）。

表-1：操作码映射表中使用的上标 ::

    上标符号    符号的含义
    1A          ModR/M 字节的位 5、4 和 3 用作操作码扩展。
    1B          故意尝试生成无效操作码异常（#UD）时，使用 0F0B 操作码（UD2 指令）、
                0FB9H 操作码（UD1 指令）或 0FFFH 操作码（UD0 指令）。
    1C          某些指令使用相同的两字节操作码。如果指令有变化，或者操作码代表不同的指
                令，将使用 ModR/M 字节来区分指令。有关解码指令所需的 ModR/M 字节的值，
                请参阅表-6。
    i64         该指令在 64 位模式下无效或无法编码。40 至 4F（单字节 INC 和 DEC）在
                64 位模式下是 REX 前缀组合（使用 FE/FF Grp 4 和 5 进行 INC 和 DEC）。
    o64         仅在 64 位模式下可用的指令。
    d64         在 64 位模式下，指令默认为 64 位操作数大小，无法编码为 32 位操作数大小。
    f64         在 64 位模式下，操作数大小被强制为 64 位操作数大小（在 64 位模式下，
                更改操作数大小的前缀对此指令被忽略）。
    v           仅存在 VEX 形式。指令没有传统的 SSE 形式。对于整数 GPR 指令，意味着需
                要 VEX 前缀。
    v1          仅存在 VEX128 和 SSE 形式（没有 VEX256），当无法从数据大小推断出来时。

VEX 前缀指令
-------------

包含 VEX 前缀的指令根据 VEX.mmmmm 字段编码相对于 2 字节和 3 字节操作码映射表进行组织，
分别隐含 0F、0F38H、0F3AH。每个 VEX 编码指令的操作码映射条目基于操作码字节的值，类似于
非 VEX 编码指令。

一个 VEX 前缀包括几个比特位字段，编码隐含的 66H、F2H、F3H 前缀功能（VEX.pp）和操作数大
小/操作码信息（VEX.L）。

操作码映射表-2到表-6包括带有 VEX 前缀的指令和没有 VEX 前缀的指令。许多条目只列出一次，
但代表了带 VEX 和不带 VEX 前缀两种形式的指令。如果存在 VEX 前缀，则所有操作数都有效，助
记符通常以 “v” 开头。如果不存在 VEX 前缀，则 VEX.vvvv 操作数不可用，并且从助记符中删除
前缀 “v”。一些指令仅以 VEX 形式存在，这些指令用上标 “v” 标记。

VEX 前缀指令的操作数大小可以由操作数类型代码确定。128 位向量由 'dq' 指示，256 位向量由
'qq' 指示，支持 128 或 256 位（由 VEX.L 确定）的操作数由 'x' 指示。例如条目 "VMOVUPD Vx,Wx"
表示支持 VEX.L=0 和 VEX.L=1。

单字节操作码
-------------

单字节操作码的映射表按行（十六进制值的最低 4 位）和列（十六进制值的最高 4 位）排列。表中
的每个条目列出了以下类型的操作码之一：

- 指令助记符和操作数类型
- 用作指令前缀的操作码

映射表中对应指令的每个条目，主要操作码之后跟随的字节的规则有以下情况之一：

* 需要一个 ModR/M 字节，操作数类型根据上文列出的符号进行解释。
* 需要一个 ModR/M 字节，其 reg 字段包含一个操作码扩展。
* ModR/M 字节的使用是保留的或未定义的。这适用于表示指令前缀的条目或使用 ModR/M 的无操作
  数指令（例如 60H，PUSHA；06H，PUSH ES）。

例如，对于 ADD 指令的操作码 030500000000H，使用单字节操作码映射表（表-2）解释如下：

* 操作码的第一个数字（0）表示表的行，第二个数字（3）表示表的列。这定位了一个带有两个操作
  数的 ADD 操作码。
* 第一个操作数（类型 Gv）表示一个通用寄存器，根据操作数大小属性，可以是字或双字。第二个
  操作数（类型 Ev）表示随后跟随一个 ModR/M 字节，指定操作数是一个字或双字通用寄存器还是
  一个内存地址。
* 这条指令的 ModR/M 字节为 05H，表明随后是一个 32 位位移（00000000H）。ModR/M 字节的
  reg/opcode 部分（位 3-5）为 000，表示 EAX 寄存器。

这条操作码的指令是 ADD EAX, mem_op，mem_op 的偏移量是 00000000H。 一些单和双字节操作
码指向组编号（操作码映射表中的阴影条目）。组编号表明指令使用 ModR/M 字节中的 reg/opcode
位作为操作码扩展。

表-2：单字节操作码映射表，其中灰色单元格表示指令分组（大括号括起） ::

    00H ~ F7H
    l/c     |     0     |     1     |     2     |     3     |     4     |     5     |     6     |     7     |
    0 ----- |ADD                                                                    |PUSH       |POP        |
            |   Eb,Gb       Ev,Gv       Gb,Eb       Gv,Ev       AL,Ib       rAX,Iz      ES↑i64      ES↑i64
    1 ----- |ADC                                                                    |PUSH       |POP        |
            |   Eb,Gb       Ev,Gv       Gb,Eb       Gv,Ev       AL,Ib       rAX,Iz      SS↑i64      SS↑i64
    2 ----- |AND                                                                    |SEG=ES     |DAA↑i64    |
            |   Eb,Gb       Ev,Gv       Gb,Eb       Gv,Ev       AL,Ib       rAX,Iz     (Prefix)
    3 ----- |XOR                                                                    |SEG=SS     |AAA↑i64    |
            |   Eb,Gb       Ev,Gv       Gb,Eb       Gv,Ev       AL,Ib       rAX,Iz     (Prefix)
    4 ----- |INC↑i64 general register / REX↑o64 Prefixes
            |   eAX         eCX         eDX         eBX         eSP         eBP         eSI         eDI
            |   REX         REX.B       REX.X       REX.XB      REX.R       REX.RB      REX.RX      REX.RXB
    5 ----- |PUSH↑d64 general register
            |   rAX/r8      rCX/r9      rDX/r10     rBX/r11     rSP/r12     rBP/r13     rSI/r14     rDI/r15
    6 ----- |PUSHA↑i64/ | POPA↑i64/ |BOUND↑i64  |ARPL↑i64   |SEG=FS     |SEG=GS     |Operand Sz |Address Sz
            |PUSHAD↑i64   POPAD↑i64     Gv,Ma       Ew,Gw      (Prefix)    (Prefix)    (Prefix)    (Prefix)
            |                                    MOVSXD↑o64
            |                                       Gv,Ev
    7 ----- |Jcc↑f64, Jb - Short-displacement jump on condition
            |   O           NO         B/NAE/C    NB/AE/NC      Z/E         NZ/NE       BE/NA       NBE/A
    8 ----- |Immediate Grp 1↑1A                             |TEST                   |XCHG
            | { Eb,Ib       Ev,Iz     Eb,Ib↑i64     Ev,Ib }     Eb,Gb       Ev,Gv       Eb,Gb       Ev,Gv
    9 ----- |NOP        |XCHG word, dword or qword register with rAX
            |PAUSE(F3)      rCX/r9      rDX/r10     rBX/r11     rSP/r12     rBP/r13     rSI/r14     rDI/r15
            |XCHG r8,rAX
    A ----- |MOV                                            |MOVS/B     |MOVS/W/D/Q |CMPS/B     |CMPS/W/D
            |   AL,Ob       rAX,Ov      Ob,AL       Ov,rAX      Yb,Xb       Yv,Xv       Xb,Yb       Xv,Yv
    B ----- |MOV immediate byte into byte regiter
            | AL/R8B,Ib  CL/R9B,Ib   DL/R10B,Ib  BL/R11B,Ib  AH/R12B,Ib  CH/R13B,Ib  DH/R14B,Ib  BH/R15B,Ib
    C ----- |Shift Grp 2↑1A         |near RET↑f64           |LES↑i64    |LDS↑i64    |Grp 11↑1A - MOV
            | { Eb,Ib       Ev,Ib }     Iw                      Gz,Mp       Gz,Mp     { Eb,Ib       Ev,Iz }
            |                                                VEX+2byte   VEX+1byte
    D ----- |Shift Grp 2↑1A                                 |AAM↑i64    |AAD↑i64                |XLAT/XLATB
            | { Eb,1        Ev,1        Eb,CL       Ev,CL }     Ib          Ib
    E ----- |L~PNE↑f64/ |LOOPE↑f64/ |LOOP↑f64   |JrCXZ↑f64  |IN                     |OUT
            |LOOPNZ↑f64  LOOPZ↑f64      Jb          Jb          AL,Ib       eAX,Ib      Ib,AL       Ib,eAX
            |   Jb          Jb
    F ----- |LOCK       |INT1       |REPNE      |REP/REPE   |HLT        |CMC        |Unary Grp 3↑1A
            | (Prefix)               XACQUIRE    XRELEASE                             { Eb          Ev }
            |                        (Prefix)    (Prefix)

    08H ~ FFH
    l/c     |     8     |     9     |     A     |     B     |     C     |     D     |     E     |     F     |
    0 ----- |OR                                                                     |PUSH       |2-byte ESC
            |   Eb,Gb       Ev,Gv       Gb,Eb       Gv,Ev       AL,Ib       rAx,Iz      CS↑i64      表-3
    1 ----- |SBB                                                                    |PUSH       |POP
            |   Eb,Gb       Ev,Gv       Gb,Eb       Gv,Ev       AL,Ib       rAX,Iz      DS↑i64      DS↑i64
    2 ----- |SUB                                                                    |SEG=CS     |DAS↑i64
            |   Eb,Gb       Ev,Gv       Gb,Eb       Gv,Ev       AL,Ib       rAX,Iz     (Prefix)
    3 ----- |CMP                                                                    |SEG=DS     |AAS↑i64
            |   Eb,Gb       Ev,Gv       Gb,Eb       Gv,Ev       AL,Ib       rAX,Iz     (Prefix)
    4 ----- |DEC↑i64 general register / REX↑o64 Prefixes
            |   eAX         eCX         eDX         eBX         eSP         eBP         eSI         eDI
            |   REX.W       REX.WB      REX.WX      REX.WXB     REX.WR      REX.WRB     REX.WRX     REX.WRXB
    5 ----- |POP↑d64 into general register
            |   rAX/r8      rCX/r9      rDX/r10     rBX/r11     rSP/r12     rBP/r13     rSI/r14     rDI/r15
    6 ----- |PUSH↑d64   |IMUL       |PUSH↑d64   |IMUL       |INS/INSB   |INS/INSW/D |OUTS/OUTSB |OUTS/SW/SD
            |   Iz        Gv,Ev,Iz      Ib        Gv,Ev,Ib      Yb,DX       Yz,DX       DX,Xb       DX,Xz
    7 ----- |Jcc↑f64, Jb - Short displacement jump on condition
            |   S           NS          P/PE        NP/PO       L/NGE       NL/GE       LE/NG       NLE/G
    8 ----- |MOV                                            |MOV        |LEA        |MOV        |Grp 1A↑1A POP↑d64
            |   Eb,Gb       Ev,Gv       Gb,Eb       Gv,Ev       Ev,Sw       Gv,M        Sw,Ew      { Ev }
    9 ----- |CBW/CWDE/  |CWD/CDQ/CQO|far CALL↑i64|FWAIT/WAIT|PUSHF/D/   |POPF/D/    |SAHF       |LAHF
            |CDQE                                            Q↑d64       Q↑d64
            |                           Ap                      Fv          Fv
    A ----- |TEST                   |STOS/B     |STOS/W/D/Q |LODS/B     |LODS/W/D/Q |SCAS/B     |SCAS/W/D/Q
            |   AL,Ib       rAX,Iz      Yb,AL       Yv,rAX      AL,Xb       rAX,Xv      AL,Yb       rAX,Yv
    B ----- |MOV immediate word or double into word, double, or quad register
            | rAX/r8,Iv  rCX/r9,Iv   rDX/r10,Iv  rBX/r11,Iv  rSP/r12,Iv  rBP/r13,Iv  rSI/r14,Iv  rDI/r15,Iv
    C ----- |ENTER      |LEAVE↑d64  |far RET    |far RET    |INT3       |INT        |INTO↑i64   |IRET/D/Q
            |   Iw,Ib                   Iw                                  Ib
    D ----- |ESC (Escape to coprocessor instruction set)
    E ----- |near CALL↑f64|JMP                              |IN                     |OUT
            |   Jz         near↑f64   far↑f64   short↑f64       AL,DX       eAX,DX      DX,AL       DX,eAX
            |               Jz          Ap          Jb
    F ----- |CLC        |STC        |CLI        |STI        |CLD        |STD        |INC/DEC    |INC/DEC
            |                                                                        Grp 4↑1A    Grp 5↑1A

双字节操作码
-------------

表-3中显示的双字节操作码映射包括长度为两字节或三字节的主操作码。长度为两字节的主操作码
以转义操作码 0FH 开始。第二个操作码字节的高四位和低四位用于在表-3中索引特定的行和列。长
度为三字节的两字节操作码以强制性前缀（66H、F2H 或 F3H）和转义操作码（0FH）开始。第三个
字节的高四位和低四位用于在表-3中索引特定的行和列（第二个操作码字节是三字节转义操作码 38H
或 3AH 时除外；在这种情况下，参考三字节操作码映射表）。

映射表中对应指令的每个条目，主要操作码之后跟随的字节的规则有以下情况之一：

- 需要一个 ModR/M 字节，操作数类型根据上文列出的符号进行解释。
- 需要一个 ModR/M 字节，其 reg 字段中包含一个操作码扩展。
- ModR/M 字节的使用是保留的或未定义的。这适用于使用 ModR/M 编码的操作数没有编码的指令
  （例如 0F77H，EMMS）。

例如，对于 SHLD 指令的操作码 0FA4050000000003H，使用双字节操作码映射表解释如下：

- 操作码位于 A 行 4 列。该位置指示了一个带有 Ev、Gv 和 Ib 操作数的 SHLD 指令。
- Ev：ModR/M 字节跟在操作码后面，指定一个字或双字操作数。
- Gv：ModR/M 字节的 reg 字段选择一个通用寄存器。
- Ib：立即数数据编码在指令的后续字节中。
- 第三个字节是 ModR/M 字节（05H）。ModR/M 的 mod 和 opcode/reg 字段表明使用了 32 位
  偏移量来定位内存中的第一个操作数，并将 eAX 作为第二个操作数。
- 操作码的下一部分是目标内存操作数的 32 位偏移（00000000H）。最后一个字节存储立即数字
  节（03H）。
- 通过这种分解，该操作码代表指令：SHLD DS:00000000H, EAX, 3。

表-3：双字节操作码映射表（第一个字节为 0FH） ::

    00H ~ F7H
    l/c pfx |     0     |     1     |     2     |     3     |     4     |     5     |     6     |     7     |
    0 ----- |Grp 6↑1A   |Grp 7↑1A   |LAR        |LSL                    |SYSCALL↑o64|CLTS       |SYSRET↑o64
            |                           Gv,Ew       Gv,Ew
    1 ----- |vmovups    |vmovups    |vmovlps    |vmovlps    |vunpcklps  |vunpckhps  |vmovhps↑v1 |vmovhps↑v1
            |  Vps,Wps     Wps,Vps    Vq,Hq,Mq      Mq,Vq     Vx,Hx,Wx    Vx,Hx,Wx    Vdq,Hq,Mq     Mq,Vq
            |                        vmovhlps                                        vmovlphs
            |                         Vq,Hq,Uq                                        Vdq,Hq,Uq
        66  |vmovupd    |vmovupd    |vmovlpd    |vmovlpd    |vunpcklpd  |vunpckhpd  |vmovhpd↑v1 |vmovhpd↑v1
            |  Vpd,Wpd     Wpd,Vpd    Vq,Hq,Mq      Mq,Vq     Vx,Hx,Wx    Vx,Hx,Wx    Vdq,Hq,Mq     Mq,Vq
        F3  |vmovss     |vmovss     |vmovsldup  |                                   |vmovshdup  |
            | Vx,Hx,Wss  Wss,Hx,Vss     Vx,Wx                                           Vx,Wx
        F2  |vmovsd     |vmovsd     |vmovddup   |
            | Vx,Hx,Wsd  Wsd,Hx,Vsd     Vx,Wx
    2 ----- |MOV        |MOV        |MOV        |MOV        |
            |   Rd,Cd       Rd,Dd       Cd,Rd       Dd,Rd
    3 ----- |WRMSR      |RDTSC      |RDMSR      |RDPMC      |SYSENTER   |SYSEXIT    |           |GETSEC
    4 ----- |CMOVcc (Gv,Ev) - Conditioanl Move
            |   O           NO         B/C/NAE    AE/NB/NC      E/Z         NE/NZ       BE/NA       A/NBE
    5 ----- |vmovmskps  |vsqrtps    |vrsqrtps   |vrcpps     |vandps     |vandnps    |vorps      |vxorps
            | Gy,Ups       Vps,Wps     Vps,Wps     Vps,Wps   Vps,Hps,Wps Vps,Hps,Wps Vps,Hps,Wps Vps,Hps,Wps
        66  |vmovmskpd  |vsqrtpd    |                       |vandpd     |vandnpd    |vorpd      |vxorpd
            | Gy,Upd       Vpd,Wpd                           Vpd,Hpd,Wpd Vpd,Hpd,Wpd Vpd,Hpd,Wpd Vpd,Hpd,Wpd
        F3  |           |vsqrtss    |vrsqrtss   |vrcpss     |
            |            Vss,Hss,Wss Vss,Hss,Wss Vss,Hss,Wss
        F2  |           |vsqrtsd    |
            |            Vsd,Hsd,Wsd
    6 ----- |punpcklbw  |punpcklwd  |punpckldq  |packsswb   |pcmpgtb    |pcmpgtw    |pcmpgtd    |packuswb
            |   Pq,Qd       Pq,Qd       Pq,Qd       Pq,Qd       Pq,Qd       Pq,Qd       Pq,Qd       Pq,Qd
        66  |vpunpcklbw |vpunpcklwd |vpunpckldq |vpacksswb  |vpcmpgtb   |vpcmpgtw   |vpcmpgtd   |vpackuswb
            | Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx
        F3  |
    7 ----- |pshufw     |(Grp 12↑1A)|(Grp 13↑1A)|(Grp 14↑1A)|pcmpeqb    |pcmpeqw    |pcmpeqd    |emms
            | Pq,Qq,Ib  |           |           |           |   Pq,Qq       Pq,Qq       Pq,Qq    vzeroupper↑v
            |           |           |           |           |                                    vzeroall↑v
        66  |vpshufd    |           |           |           |vpcmpeqb   |vpcmpeqw   |vpcmpeqd   |
            | Vx,Wx,Ib  |           |           |           | Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx
        F3  |vpshufhw   |           |           |           |
            | Vx,Wx,Ib  |           |           |           |
        F2  |vpshuflw   |           |           |           |
            | Vx,Wx,Ib  |           |           |           |
    8 ----- |Jcc↑f64, Jz - Long displacement jump on condition
            |   O           NO         B/C/NAE    AE/NB/NC      E/Z         NE/NZ       BE/NA       A/NBE
    9 ----- |SETcc, Eb - Byte Set on condition
            |   O           NO         B/C/NAE    AE/NB/NC      E/Z         NE/NZ       BE/NA       A/NBE
    A ----- |PUSH↑d64   |POP↑d64    |CPUID      |BT         |SHLD       |SHLD       |
            |   FS          FS                      Ev,Gv      Ev,Gv,Ib    Ev,Gv,CL
    B ----- |CMPXCHG                |LSS        |BTR        |LFS        |LGS        |MOVZX                  |
            |   Eb,Gb       Ev,Gv       Gv,Mp       Ev,Gv       Gv,Mp       Gv,Mp       Gv,Eb       Gv,Ew
    C ----- |XADD       |XADD       |vcmpps     |movnti     |pinsrw     |pextrw     |vshufps    |Grp 9↑1A   |
            |   Eb,Gb       Ev,Gv   Vps,Hps,Wps,Ib  My,Gy    Pq,Ry/Mw,Ib Gd,Nq,Ib Vps,Hps,Wps,Ib|           |
        66  |                       |vcmppd     |           |vpinsrw    |vpextrw    |vshufpd    |           |
            |                       Vpd,Hpd,Wpd,Ib     Vdq,Hdq,Ry/Mw,Ib Gd,Udq,Ib Vpd,Hpd,Wpd,Ib|           |
        F3  |                       |vcmpss     |                                               |           |
            |                       Vss,Hss,Wss,Ib                                              |           |
        F2  |                       |vcmpsd     |                                               |           |
            |                       Vsd,Hsd,Wsd,Ib                                              |           |
    D ----- |           |psrlw      |psrld      |psrlq      |paddq      |pmullw     |           |pmovmskb
            |               Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq                   Gd,Nq
        66  |vaddsubpd  |vpsrlw     |vpsrld     |vpsrlq     |vpaddq     |vpmullw    |vmovq      |vpmovmskb
            |Vpd,Hpd,Wpd  Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx      Wq,Vq       Gd,Ux
        F3  |                                                                       |movq2dp    |
            |                                                                          Vdq,Nq
        F2  |vaddsubps  |                                                           |movdq2q    |
            |Vps,Hps,Wps                                                                Pq,Uq
    E ----- |pavgb      |psraw      |psrad      |pavgw      |pmulhuw    |pmulhw     |           |movntq
            |   Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq                   Mq,Pq
        66  |vpavgb     |vpsraw     |vpsrad     |vpavgw     |vpmulhuw   |vpmulhw    |vcvttpd2dq |vmovntdq
            | Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx     Vx,Wpd       Mx,Vx
        F3  |                                                                       |vcvtdq2pd  |
            |                                                                          Vx,Wpd
        F2  |                                                                       |vcvtpd2dq  |
            |                                                                          Vx,Wpd
    F ----- |           |psllw      |pslld      |psllq      |pmuludq    |pmaddwd    |psadbw     |maskmovq
            |               Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Nq
        66  |           |vpsllw     |vpslld     |vpsllq     |vpmuludq   |vpmaddwd   |vpsadbw    |vmaskmovdqu
            |             Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx     Vdq,Udq
        F2  |vlddqu     |
            |   Vx,Mx


    08H ~ FFH
    l/c pfx |     8     |     9     |     A     |     B     |     C     |     D     |     E     |     F     |
    0 ----- |INVD       |WBINVD     |           |2-byte illegal|        |prefetchw(/1)|
            |                                   |Opcodes UD2↑1B|        |    Ev     |
    1 ----- |Prefetch↑1C|ReservedNOP|bndldx     |bndstx     |ReservedNOP                        |NOP /0 Ev  |
        66  |(Grp 16↑1A)|           |bndmov     |bndmov     |                                   |           |
        F3  |           |           |bndcl      |bndmk      |                                   |           |
        F2  |           |           |bndcu      |bndcn      |                                   |           |
    2 ----- |vmovaps    |vmovaps    |cvtpi2ps   |vmovntps   |cvttps2pi  |cvtps2pi   |vucomiss   |vcomiss
            |  Vps,Wps     Wps,Vps     Vps,Qpi     Mps,Vps     Ppi,Wps     Ppi,Wps     Vss,Wss     Vss,Wss
        66  |vmovapd    |vmovapd    |cvtpi2pd   |vmovntpd   |cvttpd2pi  |cvtpd2pi   |vucomisd   |vcomisd
            |  Vpd,Wpd    Wpd,Vpd      Vpd,Qpi     Mpd,Vpd     Ppi,Wpd     Qpi,Wpd     Vsd,Wsd     Vsd,Wsd
        F3  |                       |vcvtsi2ss  |           |vcvttss2si |vcvtss2si  |
            |                        Vss,Hss,Ey                Gy,Wss      Gy,Wss
        F2  |                       |vcvtsi2sd  |           |vcvttsd2si |vcvtsd2si  |
            |                        Vsd,Hsd,Ey                Gy,Wsd      Gy,Wsd
    3 ----- |3-byte ESC |           |3-byte ESC |
            |  (表-4)                  (表-5)
    4 ----- |CMOVcc (Gv,Ev) - Conditioanl Move
            |   S           NS          P/PE        NP/PO       L/NGE       NL/GE       LE/NG       NLE/G
    5 ----- |vaddps     |vmulps     |vcvtps2pd  |vcvtdq2ps  |vsubps     |vminps     |vdivps     |vmaxps
            |Vps,Hps,Wps Vps,Hps,Wps   Vpd,Wps     Vps,Wdq   Vps,Hps,Wps Vps,Hps,Wps Vps,Hps,Wps Vps,Hps,Wps
        66  |vaddpd     |vmulpd     |vcvtpd2ps  |vcvtps2dq  |vsubpd     |vminpd     |vdivpd     |vmaxpd
            |Vpd,Hpd,Wpd Vpd,Hpd,Wpd   Vps,Wpd     Vdq,Wps   VPd,Hpd,Wpd Vpd,Hpd,Wpd Vpd,Hpd,Wpd Vpd,Hpd,Wpd
        F3  |vaddss     |vmulss     |vcvtss2sd  |vcvttps2dq |vsubss     |vminss     |vdivss     |vmaxss
            |Vss,Hss,Wss Vss,Hss,Wss  Vsd,Hx,Wss   Vdq,Wps   Vss,Hss,Wss Vss,Hss,Wss Vss,Hss,Wss Vss,Hss,Wss
        F2  |vaddsd     |vmulsd     |vcvtsd2ss  |           |vsubsd     |vminsd     |vdivsd     |vmaxsd
            |Vsd,Hsd,Wsd Vsd,Hsd,Wsd  Vss,Hx,Wsd             Vsd,Hsd,Wsd Vsd,Hsd,Wsd Vsd,Hsd,Wsd Vsd,Hsd,Wsd
    6 ----- |punpckhbw  |punpckhwd  |punpckhdq  |packssdw   |                       |movd/q     |movq
            |   Pq,Qd       Pq,Qd       Pq,Qd       Pq,Qd                               Pd,Ey       Pq,Qq
        66  |vpunpckhbw |vpunpckhwd |vpunpckhdq |vpackssdw  |vpunpcklqdq|vpunpckhqdq|vmovd/q    |vmovdqa
            | Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx      Vy,Ey       Vx,Wx
        F3  |                                                                                   |vmovdqu
            |                                                                                       Vx,Wx
    7 ----- |VMREAD     |VMWRITE    |                                               |movd/q     |movq
            |   Ey,Gy       Gy,Ey                                                       Ey,Pd       Qq,Pq
        66  |                                               |vhaddpd    |vhsubpd    |vmovd/q    |vmovdqa
            |                                                Vpd,Hpd,Wpd Vpd,Hpd,Wpd    Ey,Vy       Wx,Vx
        F3  |                                                                       |vmovq      |vmovdqu
            |                                                                           Vq,Wq       Wx,Vx
        F2  |                                               |vhaddps    |vhsubps    |
            |                                                Vps,Hps,Wps Vps,Hps,Wps
    8 ----- |Jcc↑f64, Jz - Long displacement jump on condition
            |   S           NS          P/PE        NP/PO       L/NGE       NL/GE       LE/NG       NLE/G
    9 ----- |SETcc, Eb - Byte Set on condition
            |   S           NS          P/PE        NP/PO       L/NGE       NL/GE       LE/NG       NLE/G
    A ----- |PUSH↑d64   |POP↑d64    |RSM        |BTS        |SHRD       |SHRD      |(Grp 15↑1A)↑1C|IMUL
            |   GS          GS                      Ev,Gv     Ev,Gv,Ib    Ev,Gv,CL                  Gv,Ev
    B ----- |JMPE(RSVfor|Grp 10↑1A  |Grp 8↑1A   |BTC        |BSF        |BSR        |MOVSX                  |
            |emu on IPF)|INV OPCD↑1B|   Ev,Ib   |   Ev,Gv       Gv,Ev       Gv,Ev   |   Ev,Eb       Gv,Ew   |
        F3  |POPCNT     |           |           |           |TZCNT      |LZCNT      |                       |
            |   Gv,Ev   |           |           |               Gv,Ev       Gv,Ev   |                       |
    C ----- |BSWAP
            |RAX/EAX/    RCX/ECX/    RDX/EDX/    RBX/EBX/    RSP/ESP/    RBP/EBP/    RSI/ESI/    RDI/EDI/
            | R8/R8D      R9/R9D      R10/R10D    R11/R11D    R12/R12D    R13/R13D    R14/R14D    R15/R15D
    D ----- |psubusb    |psubusw    |pminub     |pand       |paddusb    |paddusw    |pmaxub     |pandn
            |   Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq
        66  |vpsubusb   |vpsubusw   |vpminub    |vpand      |vpaddusb   |vpaddusw   |vpmaxub    |vpandn
            | Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx
        F3  |
        F2  |
    E ----- |psubsb     |psubsw     |pminsw     |por        |paddsb     |paddsw     |pmaxsw     |pxor
            |   Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq
        66  |vpsubsb    |vpsubsw    |vpminsw    |vpor       |vpaddsb    |vpaddsw    |vpmaxsw    |vpxor
            | Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx
        F3  |
        F2  |
    F ----- |psubb      |psubw      |psubd      |psubq      |paddb      |paddw      |paddd      |UD0        |
            |   Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq   |           |
        66  |vpsubb     |vpsubw     |vpsubd     |vpsubq     |vpaddb     |vpaddw     |vpaddd     |           |
            | Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx  |           |
        F2  |                                                                                   |           |

三字节操作码
------------

表-4和表-5中显示的三字节操作码映射包括长度为 3 或 4 字节的主操作码。长度为三字节的主操
作码以两个转义字节 0F38H 或 0F3AH 开始。第三个操作码字节的高四位和低四位用于在表-4或
表-5中索引特定的行和列。长度为四字节的三字节操作码以强制性前缀（66H、F2H 或 F3H）和两个
转义字节（0F38H 或 0F3AH）开始。第四个字节的高四位和低四位用于在表-4或表-5中索引特定的
行和列。

映射表中对应指令的每个条目，主要操作码之后跟随的字节的规则如下：

- 需要一个 ModR/M 字节，操作数类型根据上文中列出的符号进行解释。

例如，对于 PALIGNR 指令的操作码 660F3A0FC108H，使用三字节操作码映射表解释如下：

- 66H 是前缀，0F3AH 表示使用表-5。操作码位于 0 行，F 列，指示了一个带有 Vdq、Wdq 和 Ib
  操作数的 PALIGNR 指令。
- Vdq：ModR/M 字节的 reg 字段选择一个 128 位 XMM 寄存器。
- Wdq：ModR/M 字节的 r/m 字段选择一个 128 位 XMM 寄存器或内存位置。
- Ib：立即数数据编码在指令的后续字节中。
- 下一个字节是 ModR/M 字节（C1H）。reg 字段表明第一个操作数是 XMM0。mod 显示 r/m 字段
  指定一个寄存器，r/m 表明第二个操作数是 XMM1。
- 最后一个字节是立即数字节（08H）。
- 通过这种分解，该操作码代表指令：PALIGNR XMM0, XMM1, 8。

表-4：三字节操作码映射表（前两个字节为 0FH 38H） ::

    00H ~ F7H
    l/c pfx |     0     |     1     |     2     |     3     |     4     |     5     |     6     |     7     |
    0 ----- |pshufb     |phaddw     |phaddd     |phaddsw    |pmaddubsw  |phsubw     |phsubd     |phsubsw
            |   Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq
        66  |vpshufb    |vphaddw    |vphaddd    |vphaddsw   |vpmaddubsw |vphsubw    |vphsubd    |vphsubsw
            | Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx
    1   66  |pblendvb                           |vcvtph2ps↑v|blendvps   |blendvpd   |vpermps↑v  |vptest
            | Vdq,Wdq                             Vx,Wx,Ib    Vdq,Wdq     Vdq,Wdq    Vqq,Hqq,Wqq    Vx,Wx
    2   66  |vpmovsxbw  |vpmovsxbd  |vpmovsxbq  |vpmovsxwd  |vpmovsxwq  |vpmovsxdq
            | Vx,Ux/Mq    Vx,Ux/Md    Vx,Ux/Mw    Vx,Ux/Mq    Vx,Ux/Md    Vx,Ux/Mq
    3   66  |vpmovzxbw  |vpmovzxbd  |vpmovzxbq  |vpmovzxwd  |vpmovzxwq  |vpmovzxdq  |vpermd↑v   |vpcmpgtq
            | Vx,Ux/Mq    Vx,Ux/Md    Vx,Ux/Mw    Vx,Ux/Mq    Vx,Ux/Md    Vx,Ux/Mq   Vqq,Hqq,Wqq  Vx,Hx,Wx
    4   66  |vpmulld    |vphminposuw                                    |vpsrlvd/q↑v|vpsravd↑v  |vpsllvd/q↑v
            | Vx,Hx,Wx    Vdq,Wdq                                         Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx
    5 ----- |
    6 ----- |
    7 ----- |
    8   66  |INVEPT     |INVVPID    |INVPCID
            |   Gy,Mdq      Gy,Mdq      Gy,Mdq
    9   66  |vgatherdd/q↑v|~qd/q↑v  |~dps/d↑v   |~qps/d↑v                           |vfmaddsub132ps/d↑v|vfmsubadd132ps/d↑v
            |   Vx,Hx,Wx    Vx,Hx,Wx  Vx,Hx,Wx    Vx,Hx,Wx                              Vx,Hx,Wx         Vx,Hx,Wx
    A   66  |                                                                       |vfmaddsub213ps/d↑v|vfmsubadd213ps/d↑v
            |                                                                           Vx,Hx,Wx         Vx,Hx,Wx
    B   66  |                                                                       |vfmaddsub231ps/d↑v|vfmsubadd231ps/d↑v
            |                                                                           Vx,Hx,Wx         Vx,Hx,Wx
    C ----- |
    D ----- |
    E ----- |
    F ----- |MOVBE      |MOVBE      |ANDN↑v     |Grp 17↑1A  |           |BZHI↑v                 |BEXTR↑v
            |   Gy,My       My,Gy     Gy,By,Ey  |           |             Gy,Ey,By                Gy,Ey,By
        66  |MOVBE      |MOVBE                  |           |                       |ADCX       |SHLX↑v
            |   Gw,Mw       Mw,Gw               |           |                           Gy,Ey     Gy,Ey,By
        F3  |                                   |           |           |PEXT↑v     |ADOX       |SARX↑v
            |                                   |           |             Gy,By,Ey      Gy,Ey     Gy,Ey,By
        F2  |CRC32      |CRC32                  |           |           |PDEP↑v     |MULX↑v     |SHRX↑v
            |   Gd,Eb       Gd,Ey               |           |             Gy,By,Ey   By,Gy,rDX,Ey Gy,Ey,By
        66& |CRC32      |CRC32                  |           |
        F2  |   Gd,Eb       Gd,Ew               |           |

    08H ~ FFH
    l/c pfx |     8     |     9     |     A     |     B     |     C     |     D     |     E     |     F     |
    0 ----- |psignb     |psignw     |psignd     |pmulhrsw
            |   Pq,Qq       Pq,Qq       Pq,Qq       Pq,Qq
        66  |vpsignb    |vpsignw    |vpsignd    |vpmulhrsw  |vpermilps↑v|vpermilpd↑v|vtestps↑v  |vtestpd↑v
            | Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx      Vx,Wx       Vx,Wx
    1 ----- |                                               |pabsb      |pabsw      |pabsd
            |                                                   Pq,Qq       Pq,Qq       Pq,Qq
        66  |vbroadcastss↑v|~sd↑v   |~f128↑v                |vpabsb     |vpabsw     |vpabsd
            |   Vx,Wd       Vqq,Wq    Vqq,Mdq                   Vx,Wx       Vx,Wx       Vx,Wx
    2   66  |vpmuldq    |vpcmpeqq   |vmovntdqa  |vpackusdw  |vmaskmovps↑v|~pd↑v     |vmaskmovps↑v|~pd↑v
            | Vx,Hx,Wx    Vx,Hx,Wx    Vx,Wx       Vx,Hx,Wx    Vx,Hx,Mx     Vx,Hx,Mx   Mx,Hx,Vx     Mx,Hx,Vx
    3   66  |vpminsb    |vpminsd    |vpminuw    |vpminud    |vpmaxsb    |vpmaxsd    |vpmaxuw    |vpmaxud
            | Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx    Vx,Hx,Wx
    4 ----- |
    5   66  |vpbroadcastd↑v|~q↑v    |vbroadcasti128↑v
            |   Vx,Wx       Vx,Wx       Vqq,Mdq
    6 ----- |
    7   66  |vpbroadcastb↑v|~w↑v
            |   Vx,Wx       Vx,Wx
    8   66  |                                               |vpmaskmovd/q↑v         |vpmaskmovd/q↑v
            |                                                 Vx,Hx,Mx                Mx,Vx,Hx
    9   66  |vfmadd132ps/d↑v|~ss/d↑v|vfmsub132ps/d↑v|~ss/d↑v|vfnmadd132ps/d↑v|~ss/d↑v|vfnmsub132ps/d↑v|~ss/d↑v
            | Vx,Hx,Wx      Vx,Hx,Wx  Vx,Hx,Wx      Vx,Hx,Wx  Vx,Hx,Wx       Vx,Hx,Wx  Vx,Hx,Wx       Vx,Hx,Wx
    A   66  |vfmadd213ps/d↑v|~ss/d↑v|vfmsub213ps/d↑v|~ss/d↑v|vfnmadd213ps/d↑v|~ss/d↑v|vfnmsub213ps/d↑v|~ss/d↑v
            | Vx,Hx,Wx      Vx,Hx,Wx  Vx,Hx,Wx      Vx,Hx,Wx  Vx,Hx,Wx       Vx,Hx,Wx  Vx,Hx,Wx       Vx,Hx,Wx
    B   66  |vfmadd231ps/d↑v|~ss/d↑v|vfmsub231ps/d↑v|~ss/d↑v|vfnmadd231ps/d↑v|~ss/d↑v|vfnmsub231ps/d↑v|~ss/d↑v
            | Vx,Hx,Wx      Vx,Hx,Wx  Vx,Hx,Wx      Vx,Hx,Wx  Vx,Hx,Wx       Vx,Hx,Wx  Vx,Hx,Wx       Vx,Hx,Wx
    C ----- |sha1nexte  |sha1msg1   |sha1msg2   |sha256rnds2|sha256msg1 |sha256msg2
            |   Vdq,Wdq     Vdq,Wdq     Vdq,Wdq     Vdq,Wdq     Vdq,Wdq     Vdq,Wdq
        66  |
    D   66  |                                   |VAESIMC    |VAESENC    |VAESENCLAST|VAESDEC    |VAESDECLAST
            |                                       Vdq,Wdq  Vdq,Hdq,Wdq Vdq,Hdq,Wdq Vdq,Hdq,Wdq Vdq,Hdq,Wdq
    E ----- |
    F ----- |
        66  |
        F3  |
        F2  |
     66 & F2|

表-5：三字节操作码映射表（前两个字节为 0FH 3AH） ::

    00H ~ F7H
    l/c pfx |     0     |     1     |     2     |     3     |     4     |     5     |     6     |     7     |
    0   66  |vpermq↑v   |vpermpd↑v  |vpblendd↑v             |vpermilps↑v|vpermilpd↑v|vperm2f128↑v
            | Vqq,Wqq,Ib  Vqq,Wqq,Ib  Vx,Hx,Wx,Ib             Vx,Wx,Ib    Vx,Wx,Ib    Vqq,Hqq,Wqq,Ib
    1   66  |                                               |vpextrb    |vpextrw    |vpextrd/q  |vextractps
            |                                               Rd/Mb,Vdq,Ib Rd/Mw,Vdq,Ib Ey,Vdq,Ib   Ed,Vdq,Ib
    2   66  |vpinsrb    |vinsertps  |vpinsrd/q
            | Vdq,Hdq,    Vdq,Hdq,    Vdq,Hdq,
            | Ry/Mb,Ib    Udq/Md,Ib   Ey,Ib
    3 ----- |
    4   66  |vdpps      |vdppd      |vmpsadbw               |vpclmulqdq             |vperm2i128↑v
            |Vx,Hx,Wx,Ib Vdq,Hdq,Wdq,Ib Vx,Hx,Wx,Ib           Vdq,Hdq,Wdq,Ib          Vqq,Hqq,Wqq,Ib
    5 ----- |
    6   66  |vpcmpestrm |vpcmpestri |vpcmpistrm |vpcmpistri
            | Vdq,Wdq,Ib  Vdq,Wdq,Ib  Vdq,Wdq,Ib  Vdq,Wdq,Ib
    7 ----- |
    8 ----- |
    9 ----- |
    A ----- |
    B ----- |
    C ----- |
    D ----- |
    E ----- |
    F   F2  |RORX↑v
            | Gy,Ey,Ib

    08H ~ FFH
    l/c pfx |     8     |     9     |     A     |     B     |     C     |     D     |     E     |     F     |
    0 ----- |                                                                                   |palignr
            |                                                                                     Pq,Qq,Ib
        66  |vroundps   |vroundpd   |vroundss   |vroundsd   |vblendps   |vblendpd   |vpblendw   |vpalignr
            | Vx,Wx,Ib    Vx,Wx,Ib    Vss,Wss,Ib  Vsd,Wsd,Ib  Vx,Hx,Wx,Ib Vx,Hx,Wx,Ib Vx,Hx,Wx,Ib Vx,Hx,Wx,Ib
    1   66  |vinsertf128↑v|vextractf128↑v                               |vcvtps2ph↑v
            |Vqq,Hqq,Wqq,Ib Wdq,Vqq,Ib                                    Wx,Vx,Ib
    2 ----- |
    3   66  |vinserti128↑v|vextracti128↑v
            |Vqq,Hqq,Wqq,Ib Wdq,Vqq,Ib
    4   66  |                       |vblendvps↑v|vblendvpd↑v|vpblendvb↑v
            |                         Vx,Hx,Wx,Lx Vx,Hx,Wx,Lx Vx,Hx,Wx,Lx
    5 ----- |
    6 ----- |
    7 ----- |
    8 ----- |
    9 ----- |
    A ----- |
    B ----- |
    C ----- |                                               |sha1rnds4
            |                                                 Vdq,Wdq,Ib
    D   66  |                                                                                   |VAESKEYGEN
            |                                                                                     Vdq,Wdq,Ib
    E ----- |
    F ----- |

操作码扩展
----------

一些一字节和双字节操作码使用 ModR/M 字节中的 reg/opcode（比特 5 ~ 3）字段进行扩展。这
些操作码在表-6中展示并按组编号。组编号（第二列中的 1 到 16）提供了一个表格入口点。每个指
令的 r/m 字段编码可以使用表格第三列确定。

例如操作码为 80H 的 ADD 指令，它是一个 Grp 1 指令。表-6可以查到 ModR/M 字节中用于此指
令的操作码扩展字段（reg/opcode）为 000B，r/m（比特 2 ~ 0）字段可以编码访问寄存器（11B）
或使用指定的寻址模式访问内存地址（mem = 00B, 01B, 10B）。

使用表-2、表-3、表-6查找 VMRESUME 指令的操作码 0F01C3：

- 0F 表示此指令在双字节操作码映射中
- 01（表-3 第 0 行的第 1 列）显式此操作码在 表-6 的 Grp 7
- C3 是 ModR/M 字节，C3 的前两位是 11B，表示查看 表-6 中 Grp 7 的第二项
- reg/opcode 字段 [5,4,3] 是 000B，表示在 Grp 7 中的 000 列查找
- 最后，r/m 字段 [2,1,0] 是 011B，对应操作码为 VMRESUME 指令

表-6：组编号的一字节和双字节操作码扩展 ::

    Opcode  |Gr mod pfx | reg/opcode 5,4,3 (r/m 2,1,0 in parenthesis)                                                   |
            |   7,6     |    000    |    001    |    010    |    011    |    100    |    101    |    110    |    111    |
    80 ~ 83 |1  xxB --- |ADD        |OR         |ADC        |SBB        |AND        |SUB        |XOR        |CMP
            |           |
    8F      |1A xxB --- |POP        |
            |           |
    C0,     |2  xxB --- |ROL        |ROR        |RCL        |RCR        |SHL/SAL    |SHR        |           |SAR
    C1 reg, |
    imm D0, |
    D1 reg, |
    1 D2,   |
    D3 reg, |
    CL      |
            |
    F6, F7  |3  xxB --- |TEST       |           |NOT        |NEG        |MUL        |IMUL       |DIV        |IDIV
            |           |   Ib/Iz                                           AL/rAX      AL/rAX      AL/rAX      AL/rAX
    FE      |4  xxB --- |INC        |DEC        |
            |           |   Eb          Eb
    FF      |5  xxB --- |INC        |DEC        |near CALL↑f64|far CALL |near JMP↑f64|far JMP   |PUSH↑d64   |
            |           |   Ev          Ev          Ev            Ep        Ev           Mp         Ev
    0F 00   |6  xxB --- |SLDT       |STR        |LLDT       |LTR        |VERR       |VERW       |
            |           |   Rv/Mw       Rv/Mw       Ew          Ew          Ew          Ew
    0F 01   |7  mem --- |SGDT       |SIDT       |LGDT       |LIDT       |SMSW       |           |LMSW       |INVLPG
            |           |   Ms          Ms          Ms          Ms      |   Mw/Rv   |           |   Ew          Mb
            |   11B     |VMCALL(001)  |MONITOR(000)|XGETBV(000)         |           |           |           |SWAPGS↑o64
            |           |VMLAUNCH(010)|MWAIT(001)  |XSETBV(001)         |           |           |           |  (000)
            |           |VMRESUME(011)|CLAC(010)   |VMFUNC(100)         |           |           |           |RDTSCP(001)
            |           |VMXOFF(100)  |STAC(011)   |XEND(101)           |           |           |           |
            |           |             |ENCLS(111)  |XTEST(110)          |           |           |           |
            |           |                          |ENCLU(111)          |           |           |           |
    0F BA   |8  xxB --- |                                               |BT         |BTS        |BTR        |BTC
            |           |
    0F C7   |9  mem --- |           |CMPXCHG8B  |                                               |VMPTRLD    |VMPTRST
            |           |               Mq                                                          Mq          Mq
            |           |           |CMPXCHG16B |
            |           |               Mdq
            |       66  |                                                                       |VMCLEAR    |
            |           |                                                                           Mq
            |       F3  |                                                                       |VMXON      |
            |           |                                                                           Mq
            |   11B     |                                                                       |RDRAND     |RDSEED
            |           |                                                                           Rv          Rv
            |       F3  |                                                                                   |RDPID
            |           |                                                                                       Rd/q
    0F B9   |10 mem --- |UD1                                                                                            |
            |   11B     |                                                                                               |
    C6      |11 mem --- |MOV        |
            |           |   Eb,Ib   |
            |   11B     |           |                                                                       |XABORT(000)
            |           |           |                                                                           Ib
    C7      |11 mem --- |MOV        |
            |           |   Ev,Iz   |
            |   11B     |           |                                                                       |XBEGIN(000)
            |           |           |                                                                           Jz
    0F 71   |12 mem --- |
            |   11B     |                       |psrlw      |           |psraw      |           |psllw      |
            |           |                           Nq,Ib                   Na,Ib                   Nq,Ib
            |       66  |                       |vpsrlw     |           |vpsraw     |           |vpsllw     |
            |           |                         Hx,Ux,Ib                Hx,Ux,Ib                Hx,Ux,Ib
    0F 72   |13 mem --- |
            |   11B     |                       |psrld      |           |psrad      |           |pslld      |
            |           |                           Nq,Ib                   Nq,Ib                   Nq,Ib
            |       66  |                       |vpsrld     |           |vpsrad     |           |vpslld     |
            |           |                         Hx,Ux,Ib                Hx,Ux,Ib                Hx,Ux,Ib
    0F 73   |14 mem --- |
            |   11B     |                       |psrlq      |                                   |psllq      |
            |           |                           Nq,Ib                                           Nq,Ib
            |       66  |                       |vpsrlq     |vpsrldq    |                       |vpsllq     |vpslldq
            |           |                         Hx,Ux,Ib    Hx,Ux,Ib                            Hx,Ux,Ib    Hx,Ux,Ib
    0F AE   |15 mem --- |fxsave     |fxrstor    |ldmxcsr    |stmxcsr    |XSAVE      |XRSTOR     |XSAVEOPT   |clflush
            |   11B     |                                                           |lfence     |mfence     |sfence
            |       F3  |RDFSBASE   |RDGSBASE   |WRFSBASE   |WRGSBASE   |
            |           |   Ry          Ry          Ry          Ry
    0F 18   |16 mem --- |prefetch   |prefetch   |prefetch   |prefetch   |Reserved NOP                                   |
            |           |   NTA         T0          T1          T2      |                                               |
            |   11B     |Reserved NOP                                                                                   |
    VEX.0F38|17 mem --- |           |BLSR↑v     |BLSMSK↑v   |BLSI↑v     |
    F3      |   11B     |           |   By,Ey   |   By,Ey   |   By,Ey   |
