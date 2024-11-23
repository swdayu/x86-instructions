数据移动和交换
==============

MOV 
----

数据移动： ::

    操作码              指令             操作数  模式    简要描述
            88 /r       MOV r/m8, r8        MR  V/V     r8 => r/m8
    REX   + 88 /r       MOV r/m8*, r8*      MR  V/N.E.  r8 => r/m8
            89 /r       MOV r/m16, r16      MR  V/V     r16 => r/m16
            89 /r       MOV r/m32, r32      MR  V/V     r32 => r/m32
    REX.W + 89 /r       MOV r/m64, r64      MR  V/N.E.  r64 => r/m64
            8A /r       MOV r8, r/m8        RM  V/V     r/m8 => r8
    REX   + 8A /r       MOV r8*, r/m8*      RM  V/N.E.  r/m8 => r8
            8B /r       MOV r16, r/m16      RM  V/V     r/m16 => r16
            8B /r       MOV r32, r/m32      RM  V/V     r/m32 => r32
    REX.W + 8B /r       MOV r64, r/m64      RM  V/N.E.  r/m64 => r64
            8C /r       MOV r/m16, Sreg     MR  V/V     段寄存器 => r/m16
            8C /r       MOV r/m16/r32, Sreg MR  V/V     零扩展16位段寄存器 => r16/r32/m16
    REX.W + 8C /r       MOV r64/m16, Sreg   MR  V/V     零扩展16位段寄存器 => r64/m16
            8E /r       MOV Sreg, r/m16     RM  V/V     r/m16 => 段寄存器
    REX.W + 8E /r       MOV Sreg, r/m64     RM  V/V     r/m64 低16位 => 段寄存器
            A0          MOV AL, moffs8      FD  V/V     位于 (seg:offset) 的单字节 => AL
    REX.W + A0          MOV AL, moffs8      FD  V/N.E.  位于 (offset) 的单字节 => AL
            A1          MOV AX, moffs16     FD  V/V     位于 (seg:offset) 的两字节 => AX
            A1          MOV EAX, moffs32    FD  V/V     位于 (seg:offset) 的四字节 => EAX
    REX.W + A1          MOV RAX, moffs64    FD  V/N.E.  位于 (offset) 的八字就 => RAX
            A2          MOV moffs8, AL      TD  V/V     AL => (seg:offset)
    REX.W + A2          MOV moffs8, AL      TD  V/N.E.  AL => (offset)
            A3          MOV moffs16, AX     TD  V/V     AX => (seg:offset)
            A3          MOV moffs32, EAX    TD  V/V     EAX => (seg:offset)
    REX.W + A3          MOV moffs64, RAX    TD  V/N.E.  RAX => (offset)
            B0+rb ib    MOV r8, imm8        OI  V/V     imm8 => r8
    REX   + B0+rb ib    MOV r8*, imm8       OI  V/N.E.  imm8 => r8
            B8+rw iw    MOV r16, imm16      OI  V/V     imm16 => r16
            B8+rd id    MOV r32, imm32      OI  V/V     imm32 => r32
    REX.W + B8+rd io    MOV r64, imm64      OI  V/N.E.  imm64 => r64
            C6 /0 ib    MOV r/m8, imm8      MI  V/V     imm8 => r/m8
    REX   + C6 /0 ib    MOV r/m8*, imm8     MI  V/N.E.  imm8 => r/m8
            C7 /0 iw    MOV r/m16, imm16    MI  V/V     imm16 => r/m16
            C7 /0 id    MOV r/m32, imm32    MI  V/V     imm32 => r/m32
    REX.W + C7 /0 id    MOV r/m64, imm32    MI  V/N.E.  imm32 => r/m64 （符号扩展）

    * 64 位模式下，如果使用了 REX 前缀，r/m8 不能使用使用 AH BH CH DH
    * 32 位模式下，对于 Sreg 汇编器可能插入一个 16 位操作数大小属性前缀，Sreg 表示段寄存器
    * moffs8 moffs16 moffs32 moffs64 操作数指定的是相对于分段的偏移，其中 8/16/32/64 是操作数的大小
    * /r ModR/M字段包含reg寄存器操作数和r/m操作数
    * /0 ModR/M字段仅使用r/m操作数，其中reg字段的值为0
    * ib 包含1个字节的立即数
    * iw 包含2个字节的立即数
    * id 包含4个字节的立即数
    * io 包含8个字节的立即数
    * +rb 用操作码的低三位表示寄存器操作数，操作数的大小是单个字节
    * +rw 用操作码的低三位表示寄存器操作数，操作数的大小是两个字节
    * +rd 用操作码的低三位表示寄存器操作数，操作数的大小是四个字节

    类型    操作数1             操作数2         操作数3     操作数4
    MR      ModRM:r/m (w)       ModRM:reg (r)   无          无
    RM      ModRM:reg (w)       ModRM:r/m (r)   无          无
    FD      AL/AX/EAX/RAX       Moffs           无          无
    TD      Moffs (w)           AL/AX/EAX/RAX   无          无
    OI      opcode+rd (w)       imm8/16/32/64   无          无
    MI      ModRM:r/m (w)       imm8/16/32/64   无          无

    MOV 数据移动（非64位模式）
    reg => rg2                              1000 100w : 11 reg rg2
    rg2 => reg                              1000 101w : 11 reg rg2
    mem => reg                              1000 101w : mm reg r/m
    reg => mem                              1000 100w : mm reg r/m
    imm => reg                              1100 011w : 11 000 reg : imm
    imm => reg                              1011 wreg : imm （替代编码）
    imm => mem                              1100 011w : mm 000 r/m : imm
    mem => AL AX EAX                        1010 000w : full displacement
    AL AX EAX => mem                        1010 001w : full displacement

    MOV 数据移动（64位模式）
    reg => rg2                  0100 0R0B : 1000 100w : 11 reg rg2
    r64 => rx2                  0100 1R0B : 1000 1001 : 11 r64 rx2
    rg2 => reg                  0100 0R0B : 1000 101w : 11 reg rg2
    rx2 => r64                  0100 1R0B : 1000 1011 : 11 r64 rx2
    mem => reg                  0100 0RXB : 1000 101w : mm reg r/m
    m64 => r64                  0100 1RXB : 1000 1011 : mm r64 r/m
    reg => mem                  0100 0RXB : 1000 100w : mm reg r/m
    r64 => m64                  0100 1RXB : 1000 1001 : mm r64 r/m
    imm => reg                  0100 000B : 1100 011w : 11 000 reg : imm
    imm32 => r64                0100 100B : 1100 0111 : 11 000 r64 : imm32 （零扩展）
    imm => reg                  0100 000B : 1011 wreg : imm （替代编码）
    imm64 => r64                0100 100B : 1011 1reg : imm64 （替代编码）
    imm => mem                  0100 00XB : 1100 011w : mm 000 r/m : imm
    imm32 => m64                0100 10XB : 1100 0111 : mm 000 r/m : imm32 （零扩展）
    mem => AL, AX, or EAX       0100 0000 : 1010 000w : disp
    m64 => RAX                  0100 1000 : 1010 0001 : disp64
    AL, AX, or EAX => mem       0100 0000 : 1010 001w : disp
    RAX => m64                  0100 1000 : 1010 0011 : disp64

    MOV (88H)
        Eb,Gb

    MOV (89H)
        Ev,Gv
    
    MOV (8AH)
        Gb,Eb
    
    MOV (8BH)
        Gv,Ev
    
    MOV (8CH)
        Ev,Sw
    
    MOV (8EH)
        Sw,Ew
    
    MOV (A0H)
        AL,Ob
    
    MOV (A1H)
        rAX,Ov
    
    MOV (A2H)
        Ob,AL
    
    MOV (A3H)
        Ov,rAX
    
    MOV (B0H)   源头操作数大小是字节，位于立即数字段
        r8,Ib   目的操作数大小是字节，位于操作码中的reg字段，是一个寄存器
    [0100WRXB] opcode
                [B0]  [imm8]    imm8 => AL
                [B7]  [imm8]    imm8 => BH
          [41]  [B0]  [imm8]    imm8 => R8B
          [41]  [B7]  [imm8]    imm8 => R15B

    MOV (B8H~BFH)
        reg,Iv

    MOV (C6H)
        Eb,Ib

    MOV (C7H)
        Ev,Iz

**操作**

DEST := SRC;

**标志位**

不影响标志位。

LEA
-----

加载地址： ::

    操作码              指令             操作数  模式    简要描述
            8D /r       LEA r16,m           RM  V/V     m的有效地址 => r16
            8D /r       LEA r32,m           RM  V/V     m的有效地址 => r32
    REX.W + 8D /r       LEA r64,m           RM  V/N.E.  m的有效地址 => r64

    * /r ModR/M字段包含reg寄存器操作数和r/m操作数
    * ↑A ModR/M中mod字段的值 11B 是保留的

    类型    操作数1             操作数2         操作数3     操作数4
    RM      ModRM:reg (w)       ModRM:r/m (r)   无          无

    LEA 加载地址
    => reg                              1000 1101 : mod↑A reg r/m
    => r16/r32              0100 0RXB : 1000 1101 : mod↑A reg r/m
    => r64                  0100 1RXB : 1000 1101 : mod↑A r64 r/m

    LEA (8DH)   源头操作数大小根据属性决定，位于r/m字段，只能引用内存
        Gv,M    目的操作数大小根据属性决定，位于reg字段，是一个寄存器
    [0100WRXB] opcode [mm reg r/m]
                [8D]   mm reg mem   [00~BF]         m32 EA => r32
       [40~47]  [8D]   mm reg mem   [00~BF]         m32 EA => REX.r32
       [48~4F]  [8D]   mm reg mem   [00~BF]         m64 EA => REX.r64

**操作**

加载有效地址（load effective address）。

**标志位**

不影响标志位。


MOVSX 符号位扩展移动
--------------------

MOVZX 零位扩展移动
------------------

MOVBE
------

XCHG 数据交换
--------------

BSWAP 字节交换
--------------

XADD 交换并相加
---------------

CMPXCHG 比较交换
----------------

CMPXCHG8B 比较交换8字节
-----------------------


栈操作指令
==========

PUSH
-----

入栈： ::

    操作码              指令             操作数  模式    简要描述
    FF /6               PUSH r/m16          M   V/V     r/m16 => stack
    FF /6               PUSH r/m32          M   N.E./V  r/m32 => stack
    FF /6               PUSH r/m64          M   V/N.E.  r/m64 => stack
    50+rw               PUSH r16            O   V/V     r16 => stack
    50+rd               PUSH r32            O   N.E./V  r32 => stack
    50+rd               PUSH r64            O   V/N.E.  r64 => stack
    6A ib               PUSH imm8           I   V/V     imm8 => stack
    68 iw               PUSH imm16          I   V/V     imm16 => stack
    68 id               PUSH imm32          I   V/V     imm32 => stack
    9C                  PUSHF               ZO  V/V     EFLAGS低16位 => stack
    9C                  PUSHFD              ZO  N.E./V  EFLAGS => stack
    9C                  PUSHFQ              ZO  V/N.E.  RFLAGS => stack

    * /6 ModR/M字段仅使用r/m操作数，其中reg字段的值为6
    * +rw 用操作码的低三位表示寄存器操作数，操作数的大小是两个字节
    * +rd 用操作码的低三位表示寄存器操作数，操作数的大小是四个字节
    * ib 包含1个字节的立即数
    * iw 包含2个字节的立即数
    * id 包含4个字节的立即数

    类型    操作数1             操作数2         操作数3     操作数4
    M       ModRM:r/m (r)       无              无          无
    O       opcode+rd (r)       无              无          无
    I       imm8/16/32          无              无          无
    ZO      无                  无              无          无

    PUSH 入栈（非64位模式）
    reg => stack                                1111 1111 : 11 110 reg
    reg => stack                                0101 0reg（替代编码）
    mem => stack                                1111 1111 : mm 110 r/m
    imm => stack                                0110 10s0 : imm
    EFLAGS => stack                             1001 1100

    PUSH 入栈（64位模式）
    r16 => stack        0110 0110 : 0100 000B : 1111 1111 : 11 110 r16
    r64 => stack                    0100 W00B : 1111 1111 : 11 110 r64
    r16 => stack        0110 0110 : 0100 000B : 0101 0r16（替代编码）
    r64 => stack                    0100 W00B : 0101 0r64（替代编码）
    m16 => stack        0110 0110 : 0100 000B : 1111 1111 : mm 110 r/m
    m64 => stack                    0100 W00B : 1111 1111 : mm 110 r/m
    imm8 => stack                               0110 1010 : imm8
    imm16 => stack                  0110 0110 : 0110 1000 : imm16
    imm64 => stack                              0110 1000 : imm64
    EFLAGS => stack                             1001 1100

    PUSH (50H) rAx/r8   源头操作数大小根据属性决定，不使用ModR/M字段，规定为ax/eax/rax/r8寄存器
    [0100WRXB] opcode
                [50]                    eax => stack
                [57]                    edi => stack
       [40/48]  [50]                    rax => stack
       [41/49]  [50]                    r8  => stack
       [40/48]  [57]                    rdi => stack
       [41/49]  [57]                    r15 => stack

    PUSH (68H) Iz   源头操作数大小根据属性决定，是两个字节（16位）或四个字节（32/64位），位于立即数字段
    [0100WRXB] opcode
                [68]  [imm32/imm64]     imm => stack
          [66]  [68]  [imm16]           imm16 => stack

    PUSH (6AH) Ib   源头操作数大小是字节，位于立即数字段
    [0100WRXB] opcode
                [6A]  [imm8]            imm8 => stack
    
    PUSHF (9CH) Fv  源头操作数大小根据属性决定，操作数是 EFLAGS/RFLAGS 寄存器
    [0100WRXB] opcode
                [9C]                    eflags => stack

    PUSH (FFH) Ev   源头操作数大小根据属性决定，位于r/m字段，是寄存器或内存数据
    [0100WRXB] opcode [mm reg r/m]
                [FF]   mm 110 mem   [30~37|70~77|B0~B7]     m32 => stack
       [40/48]  [FF]   mm 110 m64   [30~37|70~77|B0~B7]     m64 => stack
       [41/49]  [FF]   mm 110 m64   [30~37|70~77|B0~B7]     m64 => stack

**操作**

减少栈指针，并将源操作数存入栈顶。

**标志位**

不影响标志位。

POP
----

出栈： ::

    操作码              指令             操作数  模式    简要描述
    8F /0               POP r/m16           M   V/V     stack => m16
    8F /0               POP r/m32           M   N.E./V  stack => m32
    8F /0               POP r/m64           M   V/N.E.  stack => m64
    58+rw               POP r16             O   V/V     stack => r16
    58+rd               POP r32             O   N.E./V  stack => r32
    58+rd               POP r64             O   V/N.E.  stack => r64
    9D                  POPF                ZO  V/V     stack => EFLAGS低16位
    9D                  POPFD               ZO  N.E./V  stack => EFLAGS
    9D                  POPFQ               ZO  V/N.E.  stack => 零扩展到RFLAGS

    类型    操作数1             操作数2         操作数3     操作数4
    M       ModRM:r/m (w)       无              无          无
    O       opcode+rd (w)       无              无          无
    ZO      无                  无              无          无

    POP 出栈（非64位模式）
    stack => reg                                    1000 1111 : 11 000 reg
    stack => reg                                    0101 1reg （替代编码）
    stack => mem                                    1000 1111 : mm 000 r/m
    stack => EFLAGS                                 1001 1101

    POP 出栈（64位模式）
    stack => r16            0110 0110 : 0100 000B : 1000 1111 : 11 000 r16
    stack => r64                        0100 W00B : 1000 1111 : 11 000 r64
    stack => r16            0110 0110 : 0100 000B : 0101 1r16 （替代编码）
    stack => r64                        0100 W00B : 0101 1r64 （替代编码）
    stack => m64                        0100 W0XB : 1000 1111 : mm 000 r/m
    stack => m16            0110 0110 : 0100 00XB : 1000 1111 : mm 000 r/m
    stack => FLAGS                      0110 0110 : 1001 1101
    stack => RFLAGS                     0100 1000 : 1001 1101

    POP (58H) rAX/r8    目的操作数大小根据属性决定，不使用ModR/M字段，规定为ax/eax/rax/r8寄存器
    [0100WRXB] opcode
                [58]                    stack => eax
                [5F]                    stack => edi
       [40/48]  [58]                    stack => rax
       [41/49]  [58]                    stack => r8
       [40/48]  [5F]                    stack => rdi
       [41/49]  [5F]                    stack => r15

    POPF (9DH) Fv  目的操作数大小根据属性决定，操作数是 EFLAGS/RFLAGS 寄存器
    [0100WRXB] opcode
                [9D]                    stack => eflags
          [66]  [9D]                    stack => flags
          [48]  [9D]                    stack => rflags

    POP (8FH) Ev   目的操作数大小根据属性决定，位于r/m字段，是寄存器或内存数据
    [0100WRXB] opcode [mm reg r/m]
                [8F]   mm 000 mem   [00~07|40~47|80~87]     stack => m32
       [40~43]  [8F]   mm 000 m64   [00~07|40~47|80~87]     stack => m64
       [48~4B]  [8F]   mm 000 m64   [00~07|40~47|80~87]     stack => m64

**操作**

加载栈顶值到目的操作数，并增加栈指针。

**标志位**

POPF/POPFD 对 EFLAGS 寄存器的影响： ::

    模式    操作数大小  CPL IOPL 21  20  19 18 17 16 14 13:12 11 10 09 08 07 06 04 02 00
                                ID VIP VIF AC VM RF NT  IOPL OF DF IF TF SF ZF AF PF CF
    保护模式，   16      0  0-3   N  N  N   N  N  0   S   S   S  S  S  S  S  S  S  S  S
    兼容模式，   16    1-3  <CPL  N  N  N   N  N  0   S   N   S  S  N  S  S  S  S  S  S
    64位模式     16    1-3 >=CPL  N  N  N   N  N  0   S   N   S  S  S  S  S  S  S  S  S
    (CR0.PE=1   32,64   0  0-3   S  N  N   S  N  0   S   S   S  S  S  S  S  S  S  S  S
    EFLAGS.VM   32,64 1-3  <CPL  S  N  N   S  N  0   S   N   S  S  N  S  S  S  S  S  S
    =0)         32,64 1-3 >=CPL  S  N  N   S  N  0   S   N   S  S  S  S  S  S  S  S  S

    * S - 用栈值更新
    * N - 值不变
    * 0 - 值被清0

类型转换指令
============

CBW 字节到字
------------

CWD 字到双字
------------

CWDE 字到双字扩展
-----------------

CDQ 字到四字
------------

段寄存器指令
============

LDS 用DS加载长指针
------------------

LES 用ES加载长指针
------------------

LFS 用FS加载长指针
------------------

LGS 用GS加载长指针
------------------

LSS 用SS加载长指针
------------------

