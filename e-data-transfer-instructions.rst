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
            B0 +rb ib   MOV r8, imm8        OI  V/V     imm8 => r8
    REX   + B0 +rb ib   MOV r8*, imm8       OI  V/N.E.  imm8 => r8
            B8 +rw iw   MOV r16, imm16      OI  V/V     imm16 => r16
            B8 +rd id   MOV r32, imm32      OI  V/V     imm32 => r32
    REX.W + B8 +rd io   MOV r64, imm64      OI  V/N.E.  imm64 => r64
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
    OI      opcode +rd (w)      imm8/16/32/64   无          无
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
                [80]  [imm8]    imm8 => AL
                [87]  [imm8]    imm8 => BH
          [41]  [80]  [imm8]    imm8 => R8B
          [41]  [87]  [imm8]    imm8 => R15B

    MOV (B8H~BFH)
        reg,Iv

    MOV (C6H)
        Eb,Ib

    MOV (C7H)
        Ev,Iz

LEA
-----

加载地址： ::

    LEA (8DH)
        Gv,M

MOVSX 符号位扩展移动
--------------------

MOVZX 零位扩展移动
------------------

MOVBE
------

XCHG 数据交换
--------------

CMOVcc 条件移动
---------------

BSWAP 字节交换
--------------

XADD 交换并相加
---------------

CMPXCHG 比较交换
----------------

CMPXCHG8B 比较交换8字节
-----------------------


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


栈操作指令
==========

PUSH 入栈
----------

PUSHA 入栈所有寄存器
--------------------

POP 出栈
---------

POPA 出栈所有寄存器
-------------------


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

