程序执行寄存器
==============

通用寄存器，段寄存器，指令指针，以及处理器状态寄存器： ::

    通用寄存器
    64位: rax rbx rcx rdx rbp rsi rdi rsp r8  r9  r10  r11  r12  r13  r14  r15
    32位: eax ebx ecx edx ebp esi edi esp r8d r9d r10d r11d r12d r13d r14d r15d
    16位:  ax  bx  cx  dx  bp  si  di  sp r8w r9w r10w r11w r12w r13w r14w r15w
     8位:  al  bl  cl  dl bpl sil dil spl r8b r9b r10b r11b r12b r13b r14b r15b
     8位:  ah  bh  ch  dh

    段寄存器
    16位: cs ds es fs gs ss
    其中64位机器模式会忽略 ds es ss

    指令指针
    64位: rip
    32位: eip
    16位:  ip

    处理器状态寄存器
    64位: rflags
    32位: eflags
    16位:  flags

    包括状态标记（S），控制标记（C），系统标记（X）：
    bit0  S cf  进位  CY  无符号算术运算结果相对目标寄存器有溢出
    bit2  S pf  奇偶  PE  结果最低字节有偶数个1
    bit4  S af  辅助  AC  辅助进位，用于4位BCD码计算，BCD码计算溢出
    bit6  S zf  零值  ZR  计算结果是零
    bit7  S sf  符号  PL  计算结果是负数
    bit8  X tf  陷阱      启用单步调试
    bit9  X if  中断  EI  中断使能标记，开启时允许响应可掩码中断
    bit10 C df  方向  UP  用于串操作，索引寄存器（si、di）递减，否则递增
    bit11 S of  溢出  OV  有符号算术运算结果相对目标寄存器有溢出
    bit12 X io  特权      iopl（privilege level）
    bit13 X pl  级别      iopl，输入输出特权级别，控制对输入输出指令的访问
    bit14 X nt  嵌套      嵌套任务标记，用于控制被中断和被调用任务的嵌套链
    bit16 X rf  恢复      恢复调制器执行，控制处理器对调试异常的响应
    bit17 X vm  虚拟      启用8086虚拟模式
    bit18 X ac  对齐      启用对齐检查
    bit19 X vif 虚拟      虚拟中断使能标记
    bit20 X vip 虚拟      虚拟中断待处理
    bit21 X id  标识      启用cpuid指令

在32位机器模式下，不能使用以 r 开头的寄存器，包括：rax rbx rcx rdx rbp rsi rdi rsp
r8 ~ r15 r8d ~ r15d r8w ~ r15w r8l ~ r15l rip rflags；也不能使用这四个64位机器才
引入的寄存器：bpl sil dil spl。寄存器以及标记名称如下： ::

    ax      累加寄存器（accumulator）
    bx      基址寄存器（base）
    cx      计数寄存器（counter）
    dx      数据寄存器（data）
    bp      栈基寄存器（stack-frame base pointer）
    si      变址寄存器（source index）
    di      变址寄存器（destination index）
    sp      栈顶寄存器（stack pointer），指向栈顶元素
    cs      代码段寄存器（code segment）
    ds      数据段寄存器（data segment）
    es      附加数据段（data segment）
    fs      附加数据段（data segment）
    gs      附加数据段（data segment）
    ss      栈段寄存器（stack segment）
    ip      指令指针（instruction pointer），指向下一条要执行的指令
    cf      进位标记（carry flag）
    pf      奇偶标记（parity flag）
    af      辅助进位（auxiliary carry flag）
    zf      零值标记（zero flag）
    sf      符号标记（sign flag）
    tf      陷阱标记（trap flag）
    if      中断标记（interrupt enable flag）
    df      方向标记（direction flag）
    of      溢出标记（overflow flag）
    iopl    输入输出特权等级（i/o privilege level）
    nt      嵌套标记（nested task flag）
    rf      恢复标记（resume flag）
    vm      虚拟模式（virtual-8086 mode flag）
    ac      对齐检查（alignment check or access control flag）
    vif     虚拟中断标记（virtual interrupt flag）
    vip     虚拟中断待处理（virtual interrupt pending）
    id      标识标记（identification flag）

浮点寄存器
-----------

浮点寄存器包含数据寄存器、状态寄存器、控制寄存器、标签寄存器、操作码寄存器、浮点指令指针、
浮点数据指针等，其中多媒体扩展寄存器（MMX）共享浮点数据寄存器的低64位： ::

    浮点数据寄存器
    80位:  st0 ~  st7 (FPU data register - register stack)
    64位: mmx0 ~ mmx7 (MMX multimedia extensions)

    浮点状态寄存器（status word register）
    16位: fsw

    浮点控制寄存器（control word register）
    16位: fcw

    浮点标签寄存器（tag word register）
    16位：ftw

    浮点操作码寄存器（opcode register）
    11位: fop

    浮点指令指针
    48位: fip

    浮点数据指针
    48位: fdp

单指多码寄存器
--------------

单指多码寄存器（SIMD, single instruction multiple data）包含 SSE 寄存器、AVX 寄存器、
AVX-512 扩展寄存器等等，其中 xmm0 ~ xmm15 共享 ymm0 ~ ymm15 的低128位，ymm0 ~ ymm15
共享 zmm0 ~ zmm15 的低256位。 ::

    SIMD寄存器
    256位: ymm0 ~ ymm15 (AVX advanced vector extensions)
    128位: xmm0 ~ xmm15 (SSE streaming simd extensions)

    SSE状态寄存器
    32位: mxcsr

    边界寄存器
    128位: bnd0 ~ bnd3

    边界状态寄存器
    128位: bndcsr
     64位: bndstatus bndcfgu

    AVX-512扩展寄存器
    512位: zmm0 ~ zmm15 zmm16 ~ zmm31
    256位: ymm0 ~ ymm15
    128位: xmm0 ~ xmm15

    向量掩码寄存器（AVX-512）
    64位: k0 ~ k7

其中32位机器模式只能使用 xmm0 ~ xmm7 以及 ymm0 ~ ymm7。

系统级寄存器
-------------

系统级寄存器包含控制寄存器、扩展控制寄存器、调试寄存器、内存管理寄存器等等，其中控制寄存
器 cr8 又称为任务优先寄存器 tpr（task priority register）： ::

    控制寄存器
    64位: cr0 ~ cr4 tpr/cr8 cr9 ~ cr15
    32位: cr0 ~ cr4

    扩展控制寄存器
    64位: xcr0

    调试寄存器
    64位: dr0 ~ dr7 dr8 ~ dr15
    32位: dr0 ~ dr7

    内存管理寄存器：全局描述符表、中断描述符表
    80位: gdtr idtr
    48位: gdtr idtr

    内存管理寄存器：局部描述符表、任务寄存器
    64位: ldtr tr
    32位: ldtr tr

其中32位机器模式只能使用32位的 cr0 ~ cr4，64位的 xcr0，32位的 dr0 ~ dr7，48位的 gdtr
以及 idtr，32位的 ldtr 和 tr。
