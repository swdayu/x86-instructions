数据移动和交换
==============

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
    => reg                              1000 1101 : mod↑A reg mem
    => r16/r32              0100 0RXB : 1000 1101 : mod↑A reg mem
    => r64                  0100 1RXB : 1000 1101 : mod↑A r64 mem

    LEA (8DH)   源头操作数大小根据属性决定，位于r/m字段，只能引用内存
        Gv,M    目的操作数大小根据属性决定，位于reg字段，是一个寄存器
    [0100WRXB] opcode [mm reg r/m]
                [8D]   mm reg mem   [00~BF]         m32 EA => r32
       [40~47]  [8D]   mm reg mem   [00~BF]         m32 EA => REX.r32
       [48~4F]  [8D]   mm reg mem   [00~BF]         m64 EA => REX.r64

    * Gv 操作数大小根据属性决定，位于reg字段，是一个寄存器
    * M  ModR/M 字节只能引用内存

**操作**

加载有效地址（load effective address）。

**标志位**

不影响标志位。

LODS
-----

加载字符串

STOS
-----

保存字符串

SCAS
-----

字符串扫描

MOVS
-----

字符串移动： ::

    操作码              指令             操作数  模式    简要描述
            A4          MOVS m8, m8         ZO  V/V     单字节移动 DS:ESI => ES:EDI，64位模式 R/ESI => R/EDI
            A5          MOVS m16, m16       ZO  V/V     双字节移动 DS:ESI => ES:EDI，64位模式 R/ESI => R/EDI
            A5          MOVS m32, m32       ZO  V/V     四字节移动 DS:ESI => ES:EDI，64位模式 R/ESI => R/EDI
    REX.W + A5          MOVS m64, m64       ZO  V/N.E.  八字节移动 R/ESI => R/EDI
            A4          MOVSB               ZO  V/V     单字节移动 DS:ESI => ES:EDI，64位模式 R/ESI => R/EDI
            A5          MOVSW               ZO  V/V     双字节移动 DS:ESI => ES:EDI，64位模式 R/ESI => R/EDI
            A5          MOVSD               ZO  V/V     四字节移动 DS:ESI => ES:EDI，64位模式 R/ESI => R/EDI
    REX.W + A5          MOVSQ               ZO  V/N.E.  八字节移动 R/ESI => R/EDI

    类型    操作数1             操作数2         操作数3     操作数4
    ZO      无                  无              无          无

    MOVS 字符串移动
                1010 010w
    0100 1000 : 1010 0101

    MOVS/MOVSB (A4H)
        Yb,Xb

    MOVS/MOVSW/MOVSD/MOVSQ (A5H)
        Yv,Xv

    * Xb 操作数大小是字节，通过 DS:rSI 寄存器进行内存寻址
    * Xv 操作数大小根据属性决定，通过 DS:rSI 寄存器进行内存寻址
    * Yb 操作数大小是字节，通过 ES:rDI 寄存器进行内存寻址
    * Yv 操作数大小根据属性决定，通过 ES:rDI 寄存器进行内存寻址

**操作**

将 rSI 指定的内存内容移动到 rDI 指定的内存位置，在移动操作之后，rSI 和 rDI 寄存器会根
据 EFLAGS 寄存器中的 DF 标志的设置自动增加或减少。如果 DF 标志为 0 则增加，如果 DF 标
志为 1 则减少。对于字节操作，寄存器增加或减少 1；对于字操作，增加或减少 2；对于双字操作，
增加或减少 4。MOVS、MOVSB、MOVSW 和 MOVSD 指令可以使用 REP 前缀，可以移动 ECX 个字节、
字或双字的数据块。

为了提高性能，处理器支持在 MOVS 和 MOVSB 移动字符串期间修改处理器操作。有关快速字符串操
作的详细信息，参考卷 1 第 7.3.9.3 节，以及卷 3 第 9.2.4 节快速字符串操作和无序存储。

**标志位**

不影响标志位。

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
    REX.W + A1          MOV RAX, moffs64    FD  V/N.E.  位于 (offset) 的八字节 => RAX
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
    mem => AL/AX/EAX                        1010 000w : full displacement
    AL/AX/EAX => mem                        1010 001w : full displacement

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
    mem => AL/AX/EAX            0100 0000 : 1010 000w : disp
    m64 => RAX                  0100 1000 : 1010 0001 : disp64
    AL/AX/EAX => mem            0100 0000 : 1010 001w : disp
    RAX => m64                  0100 1000 : 1010 0011 : disp64

    MOV (88H)   (89H)   (8AH)   (8BH)   (8CH)   (8EH)
        Eb,Gb   Ev,Gv   Gb,Eb   Gv,Ev   Ev,Sw   Sw,Ew

    MOV (A0H)   (A1H)   (A2H)   (A3H)
        AL,Ob  rAX,Ov   Ob,AL   Ov,rAX

    MOV (B0H)   (B1H)   (B2H)   (B3H)   (B4H)   (B5H)   (B6H)   (B7H)
      AL/R8B,Ib CL/R9B DL/R10B BL/R11B AH/R12B CH/R13B DH/R14B BH/R15B
        (B8H)   (B9H)   (BAH)   (BBH)   (BCH)   (BDH)   (BEH)   (BFH)
      rAX/r8,Iv rCX/r9 rDX/r10 rBX/r11 rSP/r12 rBP/r13 rSI/r14 rDI/r15

    MOV (C6H) mm 000 r/m imm8
        Eb,Ib
        (C7H) mm 000 r/m immz
        Ev,Iz

    * Eb 操作数大小是字节，位于r/m字段，是寄存器或内存数据
    * Ew 操作数大小是双字节，位于r/m字段，是寄存器或内存数据
    * Ev 操作数大小根据属性决定，位于r/m字段，是寄存器或内存数据
    * Gb 操作数大小是字节，位于reg字段，是一个寄存器
    * Gv 操作数大小根据属性决定，位于reg字段，是一个寄存器
    * Sw 操作数大小是双字节，ModR/M 的 reg 字段选择一个段寄存器
    * Ob 操作数大小是字节，不使用ModR/M字段，位于偏移量字段
    * Ov 操作数大小根据属性决定，不使用ModR/M字段，位于偏移量字段
    * Ib 操作数大小是字节，位于立即数字段
    * Iv 操作数大小根据属性决定，是两字节或四字节或八字节，位于立即数字段
    * Iz 操作数大小根据属性决定，是两个字节（16位）或四个字节（32/64位），位于立即数字段
    * AL/R8B 操作数大小是字节，不使用ModR/M字段，规定为对应寄存器
    * rAX/r8 操作数大小根据属性决定，不使用ModR/M字段，规定为对应寄存器

**操作**

DEST := SRC;

**标志位**

不影响标志位。

MOVSX
-------

符号扩展移动： ::

    操作码              指令             操作数  模式    简要描述
            63 /r       MOVSXD r16, r/m16   RM  V/N.E.  r/m16 => r16
            63 /r       MOVSXD r32, r/m32   RM  V/N.E.  r/m32 => r32
    REX.W + 63 /r       MOVSXD r64, r/m32   RM  V/N.E.  r/m32 符号扩展到 r64
            0F BE /r    MOVSX r16, r/m8     RM  V/V     r/m8 符号扩展到 r16
            0F BE /r    MOVSX r32, r/m8     RM  V/V     r/m8 符号扩展到 r32
      REX + 0F BE /r    MOVSX r16/32, r/m8* RM  V/N.E.  r/m8 符号扩展到 r16 或 r32
    REX.W + 0F BE /r    MOVSX r64, r/m8*    RM  V/N.E.  r/m8 符号扩展到 r64
            0F BF /r    MOVSX r32, r/m16    RM  V/V     r/m16 符号扩展到 r32
    REX.W + 0F BF /r    MOVSX r64, r/m16    RM  V/N.E.  r/m16 符号扩展到 r64

    * 不推荐在64位模式下使用不带 REX.W 的 MOVSXD 指令，而应该使用 MOV 指令

    类型    操作数1             操作数2         操作数3     操作数4
    RM      ModRM:reg (w)       ModRM:r/m (r)   无          无

    MOVSX 符号扩展移动 （非64位模式）
    mem => reg                      0000 1111 : 1011 111w : mm reg r/m

    MOVSX/MOVSXD 符号扩展移动 （64位模式）
    rg2 => reg          0100 0R0B : 0000 1111 : 1011 111w : 11 reg rg2
    r8 => r64           0100 1R0B : 0000 1111 : 1011 1110 : 11 r64 r8
    r16 => r64          0100 1R0B : 0000 1111 : 1011 1111 : 11 r64 r16
    r32 => r64                      0100 1R0B : 0110 0011 : 11 r64 r32
    mem => reg          0100 0RXB : 0000 1111 : 1011 111w : mm reg mem
    m8 => r64           0100 1RXB : 0000 1111 : 1011 1110 : mm r64 mem
    m16 => r64          0100 1RXB : 0000 1111 : 1011 1111 : mm r64 mem
    m32 => r64                      0100 1RXB : 0110 0011 : mm r64 mem

    MOVSXD↑o64 (63H)
               Gv,Ev

    MOVSX (BEH)   (BFH)
          Ev,Eb   Gv,Ew

    * Eb 操作数大小是字节，位于r/m字段，是寄存器或内存数据
    * Ew 操作数大小是双字节，位于r/m字段，是寄存器或内存数据
    * Ev 操作数大小根据属性决定，位于r/m字段，是寄存器或内存数据
    * Gv 操作数大小根据属性决定，位于reg字段，是一个寄存器

**操作**

DEST := SignExtend(SRC);

**标志位**

不影响标志位。

MOVZX
-------

零扩展移动： ::

    操作码              指令             操作数  模式    简要描述
            0F B6 /r    MOVZX r16, r/m8     RM  V/V     r/m8 零扩展到 r16
            0F B6 /r    MOVZX r32, r/m8     RM  V/V     r/m8 零扩展到 r32
      REX + 0F B6 /r    MOVZX r16/32, r/m8* RM  V/N.E.  r/m8 零扩展到 r16 或 r32
    REX.W + 0F B6 /r    MOVZX r64, r/m8*    RM  V/N.E.  r/m8 零扩展到 r64
            0F B7 /r    MOVZX r32, r/m16    RM  V/V     r/m16 零扩展到 r32
    REX.W + 0F B7 /r    MOVZX r64, r/m16    RM  V/N.E.  r/m16 零扩展到 r64

    类型    操作数1             操作数2         操作数3     操作数4
    RM      ModRM:reg (w)       ModRM:r/m (r)   无          无

    MOVZX 零扩展移动 （非64位模式）
    rg2 => reg                      0000 1111 : 1011 011w : 11 reg rg2
    mem => reg                      0000 1111 : 1011 011w : mm reg r/m

    MOVZX 零扩展移动 （64位模式）
    rg2 => reg          0100 0R0B : 0000 1111 : 1011 011w : 11 reg rg2
    mem => reg          0100 0RXB : 0000 1111 : 1011 011w : mm reg mem
    r32 => r64          0100 1R0B : 0000 1111 : 1011 0111 : 11 r64 r32
    m32 => r64          0100 1RXB : 0000 1111 : 1011 0111 : mm r64 mem

    MOVZX (B6H)   (B7H)
          Gv,Eb   Gv,Ew

    * Eb 操作数大小是字节，位于r/m字段，是寄存器或内存数据
    * Ew 操作数大小是双字节，位于r/m字段，是寄存器或内存数据
    * Gv 操作数大小根据属性决定，位于reg字段，是一个寄存器

**操作**

DEST := ZeroExtend(SRC);

**标志位**

不影响标志位。

MOVBE
------

交换字节后移动： ::

    操作码              指令             操作数  模式    简要描述
            0F 38 F0 /r MOVBE r16, m16      RM  V/V     m16反转字节 => r16
            0F 38 F0 /r MOVBE r32, m32      RM  V/V     m32反转字节 => r32
    REX.W + 0F 38 F0 /r MOVBE r64, m64      RM  V/N.E.  m64反转字节 => r64
            0F 38 F1 /r MOVBE m16, r16      MR  V/V     r16反转字节 => m16
            0F 38 F1 /r MOVBE m32, r32      MR  V/V     r32反转字节 => m32
    REX.W + 0F 38 F1 /r MOVBE m64, r64      MR  V/N.E.  r64反转字节 => m64

    类型    操作数1             操作数2         操作数3     操作数4
    RM      ModRM:reg (w)       ModRM:r/m (r)   无          无
    MR      ModRM:r/m (w)       ModRM:reg (r)   无          无

    MOVBE 反转字节后移动 （非64位模式）
    mem => reg                      0000 1111 : 0011 1000 : 1111 0000 : mod reg r/m
    reg => mem                      0000 1111 : 0011 1000 : 1111 0001 : mod reg r/m

    MOVBE 反转字节后移动 （64位模式）
    mem => reg          0100 0RXB : 0000 1111 : 0011 1000 : 1111 0000 : mod reg r/m
    m64 => r64          0100 1RXB : 0000 1111 : 0011 1000 : 1111 0000 : mod reg r/m
    reg => mem          0100 0RXB : 0000 1111 : 0011 1000 : 1111 0001 : mod reg r/m
    r64 => m64          0100 1RXB : 0000 1111 : 0011 1000 : 1111 0001 : mod reg r/m

    MOVBE   (F0H)   (F1H)
            Gy,My   My,Gy
        66  Gw,Mw   Mw,Gw

    * Gw 操作数大小是双字节，位于reg字段，是一个寄存器
    * Gy 操作数大小是四字节或八字节（64位模式），位于reg字段，是一个寄存器
    * Mw 操作数大小是双字节，ModR/M 字节只能引用内存
    * My 操作数大小是四字节或八字节（64位模式），ModR/M 字节只能引用内存

**标志位**

不影响标志位。

XLAT
-----

字节表查找： ::

    操作码              指令             操作数  模式    简要描述
            D7          XLAT m8             ZO  V/V     DS:[(E)BX + unsigned AL] => AL
            D7          XLATB               ZO  V/V     DS:[(E)BX + unsigned AL] => AL
    REX.W + D7          XLATB               ZO  V/N.E.  [RBX + unsigned AL] => AL

    类型    操作数1             操作数2         操作数3     操作数4
    ZO      无                  无              无          无

**操作**

该指令的操作如下： ::

    if AddressSize = 16 {
        AL := (DS:BX + ZeroExtend(AL));
    } else if AddressSize = 32 {
        AL := (DS:EBX + ZeroExtend(AL));
    } else AddressSize = 64 {
        AL := (RBX + ZeroExtend(AL));
    }

**标志位**

不影响标志位。

XCHG
-----

数据交换： ::

    操作码              指令             操作数  模式    简要描述
            86 /r       XCHG r/m8, r8       MR  V/V     交换 r8 和 r/m8
      REX + 86 /r       XCHG r/m8*, r8*     MR  V/N.E.  交换 r8 和 r/m8
            86 /r       XCHG r8, r/m8       RM  V/V     交换 r/m8 和 r8
      REX + 86 /r       XCHG r8*, r/m8*     RM  V/N.E.  交换 r/m8 和 r8
            87 /r       XCHG r/m16, r16     MR  V/V     交换 r16 和 r/m16
            87 /r       XCHG r16, r/m16     RM  V/V     交换 r/m16 和 r16
            87 /r       XCHG r/m32, r32     MR  V/V     交换 r32 和 r/m32
    REX.W + 87 /r       XCHG r/m64, r64     MR  V/N.E.  交换 r64 和 r/m64
            87 /r       XCHG r32, r/m32     RM  V/V     交换 r/m32 和 r32
    REX.W + 87 /r       XCHG r64, r/m64     RM  V/N.E.  交换 r/m64 和 r64
            90+rw       XCHG AX, r16        O   V/V     交换 r16 和 AX
            90+rw       XCHG r16, AX        O   V/V     交换 AX 和 r16
            90+rd       XCHG EAX, r32       O   V/V     交换 r32 和 EAX
    REX.W + 90+rd       XCHG RAX, r64       O   V/N.E.  交换 r64 和 RAX
            90+rd       XCHG r32, EAX       O   V/V     交换 EAX 和 r32
    REX.W + 90+rd       XCHG r64, RAX       O   V/N.E.  交换 RAX 和 r64

    类型    操作数1             操作数2         操作数3     操作数4
    O       AX/EAX/RAX (r,w)    opcode+rd (r,w)  无         无
    O       opcode+rd (r,w)     AX/EAX/RAX (r,w) 无         无
    MR      ModRM:r/m (r,w)     ModRM:reg (r)    无         无
    RM      ModRM:reg (w)       ModRM:r/m (r)    无         无

    XCHG 交换数据
    reg <-> rg2         1000 011w : 11 reg rg2
    mem <-> reg         1000 011w : mm reg mem
    AX/EAX <-> reg      1001 0reg

    XCHG (86H)   (87H)
         Eb,Gb   Ev,Gv

    XCHG (90H)   (91H)   (92H)   (93H)   (94H)   (95H)   (96H)   (97H)
         r8,rAX  rCX/r9  rDX/r10 rBX/r11 rSP/r12 rBP/r13 rSI/r14 rDI/r15

    * Eb 操作数大小是字节，位于r/m字段，是寄存器或内存数据
    * Ev 操作数大小根据属性决定，位于r/m字段，是寄存器或内存数据
    * Gb 操作数大小是字节，位于reg字段，是一个寄存器
    * Gv 操作数大小根据属性决定，位于reg字段，是一个寄存器
    * r8 rCX/r9 操作数大小根据属性决定，不使用ModR/M字段，规定为对应寄存器
    * rAX 操作数大小根据属性决定，不使用ModR/M字段，规定为对应寄存器

**操作**

该指令的操作如下： ::

    TEMP := DEST;
    DEST := SRC;
    SRC := TEMP;

**标志位**

不影响标志位。

XADD
------

交换并相加： ::

    操作码              指令             操作数  模式    简要描述
            0F C0 /r    XADD r/m8, r8       MR  V/V     交换 r8 和 r/m8，将求和加载到 r/m8
      REX + 0F C0 /r    XADD r/m8*, r8*     MR  V/N.E.  交换 r8 和 r/m8，将求和加载到 r/m8
            0F C1 /r    XADD r/m16, r16     MR  V/V     交换 r16 和 r/m16，将求和加载到 r/m16
            0F C1 /r    XADD r/m32, r32     MR  V/V     交换 r32 和 r/m32，将求和加载到 r/m32
    REX.W + 0F C1 /r    XADD r/m64, r64     MR  V/N.E.  交换 r64 和 r/m64，将求和加载到 r/m64

    类型    操作数1             操作数2         操作数3     操作数4
    MR      ModRM:r/m (r,w)     ModRM:reg (r,w)  无         无

    XADD 交换求和 （非64位模式）
    rg2 => reg                      0000 1111 : 1100 000w : 11 rg2 reg
    reg => mem                      0000 1111 : 1100 000w : mm reg r/m

    XADD 交换求和 （64位模式）
    rg2 => reg          0100 0R0B : 0000 1111 : 1100 000w : 11 rg2 reg
    rb2 => r8           0100 0R0B : 0000 1111 : 1100 0000 : 11 rb2 r8
    rx2 => r64          0100 0R0B : 0000 1111 : 1100 0001 : 11 rx2 r64
    reg => mem          0100 0RXB : 0000 1111 : 1100 000w : mm reg r/m
    r8 => m8            0100 1RXB : 0000 1111 : 1100 0000 : mm r8  r/m
    r64 => m64          0100 1RXB : 0000 1111 : 1100 0001 : mm r64 r/m

    XADD (C0H)  (C1H)
         Eb,Gb  Ev,Gv

    * Eb 操作数大小是字节，位于r/m字段，是寄存器或内存数据
    * Ev 操作数大小根据属性决定，位于r/m字段，是寄存器或内存数据
    * Gb 操作数大小是字节，位于reg字段，是一个寄存器
    * Gv 操作数大小根据属性决定，位于reg字段，是一个寄存器

**操作**

该指令的操作如下： ::

    TEMP := SRC + DEST;
    SRC := DEST;
    DEST := TEMP;

**标志位**

根据结果会设置 CF PF AF SF ZF OF 标志位。

BSWAP
------

字节交换： ::

    操作码              指令             操作数  模式    简要描述
            0F C8+rd    BSWAP r32           O   V*/V    r32反转字节
    REX.W + 0F C8+rd    BSWAP r64           O   V/N.E.  r64反转字节

    * BSWAP 在 Intel486 系列之前的处理器不支持

    类型    操作数1             操作数2         操作数3     操作数4
    O       opcode+rd (r,w)     无              无          无

    BSWAP 反转字节
                0000 1111 : 1100 1reg
    0100 100B : 0000 1111 : 1100 1r64

    BSWAP (C8H)               (C9H)               (CAH)               (CBH)
          RAX/EAX/R8/R8D      RCX/ECX/R9/R9D      RDX/EDX/R10/R10D    RBX/EBX/R11/R11D
          (CCH)               (CDH)               (CEH)               (CFH)
          RSP/ESP/R12/R12D    RBP/EBP/R13/R13D    RSI/ESI/R14/R14D    RDI/EDI/R15/R15D

    * RAX/EAX/R8/R8D 操作数大小根据属性决定，不使用ModR/M字段，规定为对应寄存器

**标志位**

不影响标志位。

CMPXCHG
--------

比较交换： ::

    操作码              指令             操作数  模式    简要描述
            0F B0 /r    CMPXCHG r/m8, r8    MR  V/V*    比较 AL r/m8，相等置 ZF r8 => r/m8，否则清 ZF r/m8 => AL
      REX + 0F B0 /r    CMPXCHG r/m8*, r8   MR  V/N.E.  比较 AL r/m8，相等置 ZF r8 => /m8，否则清 ZF r/m8 => AL
            0F B1 /r    CMPXCHG r/m16, r16  MR  V/V*    比较 AX r/m16，相等置 ZF r16 => r/m16，否则清 ZF r/m16 => AX
            0F B1 /r    CMPXCHG r/m32, r32  MR  V/V*    比较 EAX r/m32，相等置 ZF r32 => r/m32，否则清 ZF r/m32 => EAX
    REX.W + 0F B1 /r    CMPXCHG r/m64, r64  MR  V/N.E.  比较 RAX r/m64，相等置 ZF r64 => r/m64，否则清 ZF r/m64 => RAX
            0F C7 /1    CMPXCHG8B m64       M   V/V*    比较 EDX:EAX m64，相等置 ZF ECX:EBX => m64，否则清 ZF m64 => EDX:EAX
    REX.W + 0F C7 /1    CMPXCHG16B m128     M   V/N.E.  比较 RDX:RAX m128，相等置 ZF RCX:RBX => m128，否则清 ZF m128 => RDX:RAX

    * CMPXCHG 在 Intel486 系列以前的处理器上不支持
    * CMPXCHG8B/CMPXCHG16B 在 Pentium 系列以前的处理器上不支持

    类型    操作数1             操作数2         操作数3     操作数4
    MR      ModRM:r/m (r,w)     ModRM:reg (r)   无          无
    M       ModRM:r/m (r,w)     无              无          无

    CMPXCHG 比较后交换 （非64位模式）
    reg rg2                     0000 1111 : 1011 000w : 11 rg2 reg
    mem reg                     0000 1111 : 1011 000w : mm reg mem

    CMPXCHG 比较后交换 （64位模式）
    reg rg2                     0000 1111 : 1011 000w : 11 rg2 reg
    r8  rb2         0100 000B : 0000 1111 : 1011 0000 : 11 rb2 r8
    r64 rx2         0100 100B : 0000 1111 : 1011 0001 : 11 rx2 r64
    mem reg                     0000 1111 : 1011 000w : mm reg r/m
    m8  r8          0100 00XB : 0000 1111 : 1011 0000 : mm r8  m8
    m64 r64         0100 10XB : 0000 1111 : 1011 0001 : mm r64 m64
    EDX:EAX m64                 0000 1111 : 1100 0111 : mm 001 m64
    RDX:RAX m128    0100 10XB : 0000 1111 : 1100 0111 : mm 001 m128

    CMPXCHG (B0H)   (B1H)
            Eb,Gb   Ev,Gv

    CMPXCHG8B (C7H) mm 001 mem
               Mq

    CMPXCHG16B (C7H) mm 001 mem
                Mdq

    * Eb 操作数大小是字节，位于r/m字段，是寄存器或内存数据
    * Ev 操作数大小根据属性决定，位于r/m字段，是寄存器或内存数据
    * Gb 操作数大小是字节，位于reg字段，是一个寄存器
    * Gv 操作数大小根据属性决定，位于reg字段，是一个寄存器
    * Mq 操作数大小是八字节，ModR/M 字节只能引用内存
    * Mdq 操作数大小是双八字节，ModR/M 字节只能引用内存

**标志位**

根据比较结果会设置 ZF CF PF AF SF OF 标志位。

CBW
-----

字节到字： ::

    操作码              指令             操作数  模式    简要描述
            98          CBW                 ZO  V/V     AL 符号扩展到 AX
            98          CWDE                ZO  V/V     AX 符号扩展到 EAX
    REX.W + 98          CDQE                ZO  V/N.E.  EAX 符号扩展到 RAX

    类型    操作数1             操作数2         操作数3     操作数4
    ZO      无                  无              无          无

    CBW 单字节符号扩展到双字节
                    1001 1000
    CWDE 双字节符号扩展到四字节
                    1001 1000
    CDQE EAX符号扩展到RAX
        0100 1000 : 1001 1000

    CBW/CWDE/CDQE (98H)

**操作**

该指令对应的操作： ::

    if OperandSize = 16 **CBW** {
        AX := SignExtend(AL)
    } else if OperandSize = 32 **CWDE** {
        EAX := SignExtend(AX)
    } else 64-bit Mode OperandSize = 64 **CDQE** {
        RAX := SignExtend(EAX)
    }

**标志位**

不影响标志位。

CWD
-----

字到双字： ::

    操作码              指令             操作数  模式    简要描述
            99          CWD                 ZO  V/V     AX 符号扩展到 DX:AX
            99          CDQ                 ZO  V/V     EAX 符号扩展到 EDX:EAX
    REX.W + 99          CQO                 ZO  V/N.E.  RAX 符号扩展到 RDX:RAX

    类型    操作数1             操作数2         操作数3     操作数4
    ZO      无                  无              无          无

    CWD 双字节符号扩展到四字节
                    1001 1001
    CDQ 四字节符号扩展到八字节
                    1001 1001
    CQO RAX符号扩展到RDX:RAX
        0100 1000 : 1001 1001

    CWD/CDQ/CQO (98H)

**操作**

该指令对应的操作： ::

    if OperandSize = 16 **CWD** {
        DX := SignExtend(AX)
    } else if OperandSize = 32 **CDQ** {
        EDX := SignExtend(EAX)
    } else if 64-bit Mode and OperandSize = 64 **CQO** {
        RDX := SignExtend(RAX)
    }

**标志位**

不影响标志位。

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
