---
layout: post
author: syea
title: CSAPP Lab 2 - Bomb Lab
# subtitle:
# header-img: 
# header-mask:  
# catalog: 
# multilingual: 
tags:
    - OS
---

# CSAPP Lab 2 - Bomb Lab

## 前言

夫天地者，万物之逆旅也；光阴者，百代之过客也。而浮生若梦，为欢几何.    
如果对 `敲下代码` 到 `应用运行` 再到 `网络通信` 整个过程都有了一个比较完整的认知,对人生中当`程序员`的这些年也算有个交代.

## 实验前提    
1. 看完《CSAPP》书本第三章
2. 看完作者第三章讲课视频(非必要,但是建议看看)

## 学习后的收获    
1. 了解汇编语言
2. 了解计算机对控制的实现(`if`, `switch`, `loop`, `cpu分支预测`)
3. **了解计算机对过程的实现(`栈帧的概念`, `转移控制`, `递归`)**
4. **可以使用 `gdb` 反汇编程序, 将汇编转换为高级语言**
5. **可以使用 `gdb` 调试程序(查看寄存器数值,打断点,逐步调试,打印某个地址的数值)**
6. 了解数组/结构体的分配和访问

## 实验详解

`bomb lab`共6关加一个彩蛋关,实力不佳,只做出了前4关。    

简单列一下`gdb`常用的一些命令:    
`gdb xxx`: 进入 `gdb` 调试环境;    
`disassmeble function_name`: 反汇编函数;   
`info registers`:  打印寄存器信息;
`info frame`: 打印栈帧信息;
`break * 0x0000000000400ee0`: 在目标位置打断点;
`step`: 执行下一步;

`lldb`相对应的命令: [http://lldb.llvm.org/lldb-gdb.html](http://lldb.llvm.org/lldb-gdb.html)

1) phase_1    
首先反汇编`phase_1`，很容易看出题目是要我们输入一个字符串，如果跟内置的相等就通过，打印入参即可得到答案。    
```    
(gdb) disassemble phase_1
Dump of assembler code for function phase_1:
   0x0000000000400ee0 <+0>:	sub    $0x8,%rsp        // 开辟栈空间
   0x0000000000400ee4 <+4>:	mov    $0x402400,%esi   // 设置第一个参数
   0x0000000000400ee9 <+9>:	callq  0x401338 <strings_not_equal>   // 调用函数
   0x0000000000400eee <+14>:	test   %eax,%eax      // 判断返回值
   0x0000000000400ef0 <+16>:	je     0x400ef7 <phase_1+23>
   0x0000000000400ef2 <+18>:	callq  0x40143a <explode_bomb>
   0x0000000000400ef7 <+23>:	add    $0x8,%rsp
   0x0000000000400efb <+27>:	retq   
End of assembler dump.

(gdb) x/s 0x402400   // 打印该地址的字符串即答案
0x402400:	"Border relations with Canada have never been better."

```    


2) phase_2    
依然是反汇编目标函数
```
(gdb) disassemble phase_2
Dump of assembler code for function phase_2:
   0x0000000000400efc <+0>:	push   %rbp
   0x0000000000400efd <+1>:	push   %rbx
   0x0000000000400efe <+2>:	sub    $0x28,%rsp
   0x0000000000400f02 <+6>:	mov    %rsp,%rsi
   0x0000000000400f05 <+9>:	callq  0x40145c <read_six_numbers>   // 获取6个数字
   0x0000000000400f0a <+14>:	cmpl   $0x1,(%rsp)
   0x0000000000400f0e <+18>:	je     0x400f30 <phase_2+52>
   0x0000000000400f10 <+20>:	callq  0x40143a <explode_bomb>
   0x0000000000400f15 <+25>:	jmp    0x400f30 <phase_2+52>
   0x0000000000400f17 <+27>:	mov    -0x4(%rbx),%eax
   0x0000000000400f1a <+30>:	add    %eax,%eax
   0x0000000000400f1c <+32>:	cmp    %eax,(%rbx)
   0x0000000000400f1e <+34>:	je     0x400f25 <phase_2+41>
   0x0000000000400f20 <+36>:	callq  0x40143a <explode_bomb>
   0x0000000000400f25 <+41>:	add    $0x4,%rbx
   0x0000000000400f29 <+45>:	cmp    %rbp,%rbx
   0x0000000000400f2c <+48>:	jne    0x400f17 <phase_2+27>
   0x0000000000400f2e <+50>:	jmp    0x400f3c <phase_2+64>
   0x0000000000400f30 <+52>:	lea    0x4(%rsp),%rbx
   0x0000000000400f35 <+57>:	lea    0x18(%rsp),%rbp
   0x0000000000400f3a <+62>:	jmp    0x400f17 <phase_2+27>
   0x0000000000400f3c <+64>:	add    $0x28,%rsp
   0x0000000000400f40 <+68>:	pop    %rbx
   0x0000000000400f41 <+69>:	pop    %rbp
   0x0000000000400f42 <+70>:	retq   
End of assembler dump.

(gdb) disassemble read_six_numbers 
Dump of assembler code for function read_six_numbers:   // 将获取到的分别存入对应地址
   0x000000000040145c <+0>:	sub    $0x18,%rsp
   0x0000000000401460 <+4>:	mov    %rsi,%rdx
   0x0000000000401463 <+7>:	lea    0x4(%rsi),%rcx
   0x0000000000401467 <+11>:	lea    0x14(%rsi),%rax
   0x000000000040146b <+15>:	mov    %rax,0x8(%rsp)
   0x0000000000401470 <+20>:	lea    0x10(%rsi),%rax
   0x0000000000401474 <+24>:	mov    %rax,(%rsp)
   0x0000000000401478 <+28>:	lea    0xc(%rsi),%r9
   0x000000000040147c <+32>:	lea    0x8(%rsi),%r8
   0x0000000000401480 <+36>:	mov    $0x4025c3,%esi
   0x0000000000401485 <+41>:	mov    $0x0,%eax
   0x000000000040148a <+46>:	callq  0x400bf0 <__isoc99_sscanf@plt>
   0x000000000040148f <+51>:	cmp    $0x5,%eax
   0x0000000000401492 <+54>:	jg     0x401499 <read_six_numbers+61>
   0x0000000000401494 <+56>:	callq  0x40143a <explode_bomb>
   0x0000000000401499 <+61>:	add    $0x18,%rsp
   0x000000000040149d <+65>:	retq   
End of assembler dump.
```

`phase_2` 翻译后的代码如下，第一个数字必须是1，不然直接爆炸，然后走入一个循环，%rsp每次向后偏移4获取下一个数字，值需要是前一个的2倍，否则爆炸
```
	%rsi = %rsp
	call  read_six_numbers
	if (%rsp) == 1 {
		%rbx = 4(%rsp)   
		%rbp = 24(%rsp)  
		while (%rbp != %rbx) {
			%rax = -4(%rbx)
			%rax = %rax + %rax
			if %rax == (%rbx) {
				%rbx = %rbx + 4
			} else {
				explode_bomb()
			}
		}
	} else {
		explode_bomb()
	}
```

3) phase_3
还是`disassemble`    
```
(gdb) disassemble phase_3
Dump of assembler code for function phase_3:
   0x0000000000400f43 <+0>:	sub    $0x18,%rsp
   0x0000000000400f47 <+4>:	lea    0xc(%rsp),%rcx
   0x0000000000400f4c <+9>:	lea    0x8(%rsp),%rdx
   0x0000000000400f51 <+14>:	mov    $0x4025cf,%esi
   0x0000000000400f56 <+19>:	mov    $0x0,%eax
   0x0000000000400f5b <+24>:	callq  0x400bf0 <__isoc99_sscanf@plt>
   0x0000000000400f60 <+29>:	cmp    $0x1,%eax
   0x0000000000400f63 <+32>:	jg     0x400f6a <phase_3+39>
   0x0000000000400f65 <+34>:	callq  0x40143a <explode_bomb>
   0x0000000000400f6a <+39>:	cmpl   $0x7,0x8(%rsp)
   0x0000000000400f6f <+44>:	ja     0x400fad <phase_3+106>
   0x0000000000400f71 <+46>:	mov    0x8(%rsp),%eax
   0x0000000000400f75 <+50>:	jmpq   *0x402470(,%rax,8)
   0x0000000000400f7c <+57>:	mov    $0xcf,%eax
   0x0000000000400f81 <+62>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400f83 <+64>:	mov    $0x2c3,%eax
   0x0000000000400f88 <+69>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400f8a <+71>:	mov    $0x100,%eax
   0x0000000000400f8f <+76>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400f91 <+78>:	mov    $0x185,%eax
   0x0000000000400f96 <+83>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400f98 <+85>:	mov    $0xce,%eax
   0x0000000000400f9d <+90>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400f9f <+92>:	mov    $0x2aa,%eax
   0x0000000000400fa4 <+97>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400fa6 <+99>:	mov    $0x147,%eax
   0x0000000000400fab <+104>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400fad <+106>:	callq  0x40143a <explode_bomb>
   0x0000000000400fb2 <+111>:	mov    $0x0,%eax
   0x0000000000400fb7 <+116>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400fb9 <+118>:	mov    $0x137,%eax
   0x0000000000400fbe <+123>:	cmp    0xc(%rsp),%eax
   0x0000000000400fc2 <+127>:	je     0x400fc9 <phase_3+134>
   0x0000000000400fc4 <+129>:	callq  0x40143a <explode_bomb>
   0x0000000000400fc9 <+134>:	add    $0x18,%rsp
   0x0000000000400fcd <+138>:	retq   
End of assembler dump.
```
翻译成代码如下所示，需要我们输入两个数字，第一个数字小于7，然后根据第一个数，jump到一个位置执行
```
	%rsp - 24                 // 申请 24 字节栈空间
	%rcx = 12(%rsp)
	%rdx = 8(%rsp)
	%rsi = 0x4025cf = 4203983
	%rax = 0
	call sanf    // 获取输入值
	if %rax <= 1 {
		explode_bomb();
		return;
	}
	if 8(%rsp) > 7 {
		explode_bomb();
		return;
	} 
	%rax = 8(%rsp)
	jump 0x402470 + 8(%rax)
	if (%rax != 0xc(%rsp)) {
		explode_bomb();
		return;
	}
```
但是当时我一直不知道该jump到哪一行，不知道`*0x402470`到底意味着什么，想着跟前面一排的栈地址都没对上啊，思索再三还是没有头绪，无奈去搜了答案，结果是需要用16进制打印一下数值。
```
(gdb) x/x 0x402470
0x402470:	0x7c
```
这样一来就很简单了，如果第一个数为0，则第二个应为`$0xcf`；如果第一个数为1，则第二个数为`$0x2c3`，以此类推。

4. phase_4
仍然是反汇编
```
(gdb) disassemble phase_4
Dump of assembler code for function phase_4:
   0x000000000040100c <+0>:	sub    $0x18,%rsp
   0x0000000000401010 <+4>:	lea    0xc(%rsp),%rcx
   0x0000000000401015 <+9>:	lea    0x8(%rsp),%rdx
   0x000000000040101a <+14>:	mov    $0x4025cf,%esi
   0x000000000040101f <+19>:	mov    $0x0,%eax
   0x0000000000401024 <+24>:	callq  0x400bf0 <__isoc99_sscanf@plt>
   0x0000000000401029 <+29>:	cmp    $0x2,%eax
   0x000000000040102c <+32>:	jne    0x401035 <phase_4+41>
   0x000000000040102e <+34>:	cmpl   $0xe,0x8(%rsp)
   0x0000000000401033 <+39>:	jbe    0x40103a <phase_4+46>
   0x0000000000401035 <+41>:	callq  0x40143a <explode_bomb>
   0x000000000040103a <+46>:	mov    $0xe,%edx
   0x000000000040103f <+51>:	mov    $0x0,%esi
   0x0000000000401044 <+56>:	mov    0x8(%rsp),%edi
   0x0000000000401048 <+60>:	callq  0x400fce <func4>
   0x000000000040104d <+65>:	test   %eax,%eax
   0x000000000040104f <+67>:	jne    0x401058 <phase_4+76>
   0x0000000000401051 <+69>:	cmpl   $0x0,0xc(%rsp)
   0x0000000000401056 <+74>:	je     0x40105d <phase_4+81>
   0x0000000000401058 <+76>:	callq  0x40143a <explode_bomb>
   0x000000000040105d <+81>:	add    $0x18,%rsp
   0x0000000000401061 <+85>:	retq   
End of assembler dump.
(gdb) disassemble func4
Dump of assembler code for function func4:
   0x0000000000400fce <+0>:	sub    $0x8,%rsp
   0x0000000000400fd2 <+4>:	mov    %edx,%eax
   0x0000000000400fd4 <+6>:	sub    %esi,%eax
   0x0000000000400fd6 <+8>:	mov    %eax,%ecx
   0x0000000000400fd8 <+10>:	shr    $0x1f,%ecx
   0x0000000000400fdb <+13>:	add    %ecx,%eax
   0x0000000000400fdd <+15>:	sar    %eax
   0x0000000000400fdf <+17>:	lea    (%rax,%rsi,1),%ecx
   0x0000000000400fe2 <+20>:	cmp    %edi,%ecx
   0x0000000000400fe4 <+22>:	jle    0x400ff2 <func4+36>
   0x0000000000400fe6 <+24>:	lea    -0x1(%rcx),%edx
   0x0000000000400fe9 <+27>:	callq  0x400fce <func4>
   0x0000000000400fee <+32>:	add    %eax,%eax
   0x0000000000400ff0 <+34>:	jmp    0x401007 <func4+57>
   0x0000000000400ff2 <+36>:	mov    $0x0,%eax
   0x0000000000400ff7 <+41>:	cmp    %edi,%ecx
   0x0000000000400ff9 <+43>:	jge    0x401007 <func4+57>
   0x0000000000400ffb <+45>:	lea    0x1(%rcx),%esi
   0x0000000000400ffe <+48>:	callq  0x400fce <func4>
   0x0000000000401003 <+53>:	lea    0x1(%rax,%rax,1),%eax
   0x0000000000401007 <+57>:	add    $0x8,%rsp
   0x000000000040100b <+61>:	retq   
End of assembler dump.
```
将两个关键函数翻译之后如下所示：
```
function phase_4
	%rsp - 24
	%rcx = 0xc(%rsp)
	%rdx = 0x8(%rsp)
	%rsi = 0x4025cf
	%rax = 0
	call scan
	if %rax != 2 {
		explode_bomb();
		return;
	}
	if 8(%rsp) > 14 {
		explode_bomb();
		return;
	}
	%rdx = 14
	%rsi = 0
	%rdi = 8(%rsp)
	call func4
	if %rax != 0 {
		explode_bomb();
		return;
	}
	if 12(%rsp) != 0 {
		explode_bomb();
		return;
	}
--------------------------------

func4  

	// $rdi = 输入的第一个数 
	// %rsi = 0 
	// %rdx = 14

	%rsp - 8
	%rax = %rdx  // 14
	%rax = %rax - %rsi  // 14 - 0 = 14
	%rcx = %rax  // 14
	%rcx = 0
	%rax = %rax + %rcx  // 14 + 0 = 14
	%rax = %rax >> 1    // 7
	%rcx = (%rax) + (%rsi) * 1 // 7

	if %rcx <= %rdi {  // 输入的第一个数 <= 14
		%rax = 0
		if %rcx >= %rdi {
			return   // 输入的第一个数 == 14, 返回0
		} else {
			%rsi = %rcx + 1
			func4
		}
	} else {
		%rdx = %rcx - 1
		func4
		%rax = %rax + %rax
		return
	}

```
实验要求在调用完`func4`之后，返回值为0，`12(%rsp)`也为0，这个代码比较复杂，我决定翻译成java运行开来得出结果
```
    // x <=14
    public static int func4(int x, int a, int b) {

        // %rax = b
        int result = b;

        // %rax = %rax - a
        result = result - a;

        // %rcx = %rax  // y
        int temp = result;

        // %rcx = %rcx >>> 31
        temp = temp >>> 31;

        // %rax = %rax + %rcx
        result = result + temp;

        // %rax = %rax >> 1
        result = result >> 1;

        // %rcx = %rax + a * 1
        temp = result + a;

        if (temp <= x) {
            result = 0;
            if (temp >= x) {
                return result;
            } else {
                a = temp + 1;
                int r = func4(x, a, b);
                return r + r + 1;
            }
        } else {
            b = temp - 1;
            int r = func4(x, a, b);
            return r + r;
        }
    }
```
从0到14for循环输出返回值，为0的即为答案，一共有三组。

## 总结
之后的题目都尝试过一二，死磕到底还是妥协我纠结了很久，最终还是妥协了，不一定是错的选择，但肯定不是能让人开心的选择。
