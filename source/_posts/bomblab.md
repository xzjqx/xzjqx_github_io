---
title: 深入理解计算机系统（CSAPP）实验二 Bomb Lab
date: 2018-04-26 11:52:57
tags: 
	- C/C++
	- CSAPP
categories: 计算机系统
---
深入理解计算机系统（CSAPP）的实验二是Bomb Lab。实验中有六道关卡，我们的任务是通过查看反汇编代码，在程序运行时，从键盘输入六条正确的字符串，才能通过这六道关卡。
<!-- more -->
## 第一关：phase_1
在Ubuntu终端下通过`gdb bomb`指令开始调试bomb程序。
使用disas指令查看phase_1程序的反汇编代码：
```
Dump of assembler code for function phase_1:
   0x0000000000400ee0 <+0>:	sub    $0x8,%rsp
   0x0000000000400ee4 <+4>:	mov    $0x402400,%esi
   0x0000000000400ee9 <+9>:	callq  0x401338 <strings_not_equal>
   0x0000000000400eee <+14>:	test   %eax,%eax
   0x0000000000400ef0 <+16>:	je     0x400ef7 <phase_1+23>
   0x0000000000400ef2 <+18>:	callq  0x40143a <explode_bomb>
   0x0000000000400ef7 <+23>:	add    $0x8,%rsp
   0x0000000000400efb <+27>:	retq   
End of assembler dump.
```
从上述反汇编代码中看出`phase_1`函数调用了`strings_not_equal`函数和`explode_bomb`，使用disas指令分辨查看这两个函数的作用，反汇编代码如下：

strings_not_euqal：
```
Dump of assembler code for function strings_not_equal:
   0x0000000000401338 <+0>:	push   %r12
   0x000000000040133a <+2>:	push   %rbp
   0x000000000040133b <+3>:	push   %rbx
   0x000000000040133c <+4>:	mov    %rdi,%rbx
   0x000000000040133f <+7>:	mov    %rsi,%rbp
   0x0000000000401342 <+10>:	callq  0x40131b <string_length>
   0x0000000000401347 <+15>:	mov    %eax,%r12d
   0x000000000040134a <+18>:	mov    %rbp,%rdi
   0x000000000040134d <+21>:	callq  0x40131b <string_length>
   0x0000000000401352 <+26>:	mov    $0x1,%edx
   0x0000000000401357 <+31>:	cmp    %eax,%r12d
   0x000000000040135a <+34>:	jne    0x40139b <strings_not_equal+99>
   0x000000000040135c <+36>:	movzbl (%rbx),%eax
   0x000000000040135f <+39>:	test   %al,%al
   0x0000000000401361 <+41>:	je     0x401388 <strings_not_equal+80>
   0x0000000000401363 <+43>:	cmp    0x0(%rbp),%al
   0x0000000000401366 <+46>:	je     0x401372 <strings_not_equal+58>
   0x0000000000401368 <+48>:	jmp    0x40138f <strings_not_equal+87>
   0x000000000040136a <+50>:	cmp    0x0(%rbp),%al
   0x000000000040136d <+53>:	nopl   (%rax)
   0x0000000000401370 <+56>:	jne    0x401396 <strings_not_equal+94>
   0x0000000000401372 <+58>:	add    $0x1,%rbx
   0x0000000000401376 <+62>:	add    $0x1,%rbp
   0x000000000040137a <+66>:	movzbl (%rbx),%eax
   0x000000000040137d <+69>:	test   %al,%al
   0x000000000040137f <+71>:	jne    0x40136a <strings_not_equal+50>
   0x0000000000401381 <+73>:	mov    $0x0,%edx
   0x0000000000401386 <+78>:	jmp    0x40139b <strings_not_equal+99>
   0x0000000000401388 <+80>:	mov    $0x0,%edx
   0x000000000040138d <+85>:	jmp    0x40139b <strings_not_equal+99>
   0x000000000040138f <+87>:	mov    $0x1,%edx
   0x0000000000401394 <+92>:	jmp    0x40139b <strings_not_equal+99>
   0x0000000000401396 <+94>:	mov    $0x1,%edx
   0x000000000040139b <+99>:	mov    %edx,%eax
   0x000000000040139d <+101>:	pop    %rbx
   0x000000000040139e <+102>:	pop    %rbp
   0x000000000040139f <+103>:	pop    %r12
   0x00000000004013a1 <+105>:	retq   
End of assembler dump.
```
从strings_not_equal函数的反汇编代码可以看出，这个函数有两个输入参数，分别存在rdi和rsi寄存器中，这两个寄存器分别存的是两个字符串的起始地址。函数判断两个字符串是否相等：若相等，则返回参数寄存器eax为0，否则寄存器eax值为1。

explode_bomb：
```
Dump of assembler code for function explode_bomb:
   0x000000000040143a <+0>:	sub    $0x8,%rsp
   0x000000000040143e <+4>:	mov    $0x4025a3,%edi
   0x0000000000401443 <+9>:	callq  0x400b10 <puts@plt>
   0x0000000000401448 <+14>:	mov    $0x4025ac,%edi
   0x000000000040144d <+19>:	callq  0x400b10 <puts@plt>
   0x0000000000401452 <+24>:	mov    $0x8,%edi
   0x0000000000401457 <+29>:	callq  0x400c20 <exit@plt>
End of assembler dump.
```
从explode_bomb的反汇编代码不难看出这个函数就是产生boom的函数了，这样我们就知道，后续任务的主要目的就是不让程序进入这个函数即可。

重新回到`phase_1`函数的反汇编代码，我们就能很容易知道这个函数的任务就是输入一个字符串，与存在地址($0x402400)中的字符串比较，若两个字符串相等则不会产生爆炸（将炸弹拆除）。在gdb中使用`x/s 0x402400`指令查看该地址对应的字符串：
```
(gdb) x/s 0x402400
0x402400:	"Border relations with Canada have never been better."
```
退出gdb，重新运行`bomb`，第一条输入`Border relations with Canada have never been better.`得到结果如下，表示第一关已经通过。
```
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Border relations with Canada have never been better.
Phase 1 defused. How about the next one?
```
## 第二关：phase_2
同样的方法：首先进入gdb，使用disas指令查看`phase_2`的反汇编代码：
```
Dump of assembler code for function phase_2:
   0x0000000000400efc <+0>:	push   %rbp
   0x0000000000400efd <+1>:	push   %rbx
   0x0000000000400efe <+2>:	sub    $0x28,%rsp
   0x0000000000400f02 <+6>:	mov    %rsp,%rsi
   0x0000000000400f05 <+9>:	callq  0x40145c <read_six_numbers>
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
```
从函数`read_six_numbers`不难看出phase_2需要输入六个数字，存放于栈帧的前六个32位字中，之后程序运行分为下面几个步骤：
1. 从`cmpl $0x1,(%rsp)`中看出第一个输入的数字是1，若不为1，会跳转到`explode_bomb`函数产生爆炸；
2. 程序依次运行到`<+52>:`，执行指令`lea 0x4(%rsp),%rbx`把第二个参数的地址传给rbx寄存器，指令`lea 0x18(%rsp),%rbp`把最后一个参数的下一位地址传给rbp寄存器；
3. 之后程序运行到`jmp 0x400f17 <phase_2+27>`，跳转回`<+27>`开始进行新的运算；
4. 函数段`<+27>`到`<+32>`将上一个参数乘以2与下一个参数比较，若不等则产生爆炸；
5. 函数段`<+41>`到`<+48>`判断六个参数是否全部读完，若读完则函数返回，否则跳转回第四步继续执行；
6. 若前五步没有产生爆炸，那么这一关成功通过。

由上述分析可以看出，我们只要输入一个以1开始，之后参数每次乘2的六个参数即可，也就是`1 2 4 8 16 32`
第二关phase_2通关结果如下：
```
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Border relations with Canada have never been better.
Phase 1 defused. How about the next one?
1 2 4 8 16 32
That's number 2.  Keep going!
```
## 第三关：phase_3
使用disas查看`phase_3`函数的反汇编代码：
```
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
这个代码有点长，我们一步步来看。指令`<+14>`将一个立即数传给了esi寄存器，接下来就调用了`sscanf`函数，所以我们可以使用`x/s`查看$0x4025cf这个地址上的内容：
```
(gdb) x/s 0x4025cf
0x4025cf:	"%d %d"
```
可以看出这一关需要输入两个整数。可是之后就没有头绪了，我们可以在gdb中调试运行，查看各阶段寄存器的值，来判断这一关该怎样通过。
首先在phase_3位置设置断点：
```
(gdb) break phase_3
Breakpoint 1 at 0x400f43
```
之后运行`run`，依次输入头两关的正确答案，到第三关时，通过上述的分析，我们先随便输入两个整数：
```
(gdb) run
Starting program: /mnt/c/Users/xzjqx/Desktop/Study/计算机系统/CMU实验_2018/bomblab/bomb/bomb 
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Border relations with Canada have never been better.
Phase 1 defused. How about the next one?
1 2 4 8 16 32
That's number 2.  Keep going!
1 2

Breakpoint 1, 0x0000000000400f43 in phase_3 ()
```
之后使用`disas`指令可以查看断点后的一部分汇编代码，就是phase_3函数的反汇编代码。
使用`stepi`指令可以单步调试，由于我们已经不需要知道`<__isoc99_sscanf@plt>`函数的内容，我们直接在`0x0000000000400f60 <+29>:`处设置断点，再`continue`即可：
```
(gdb) break *0x400f60
Breakpoint 2 at 0x400f60
(gdb) continue
Continuing.

Breakpoint 2, 0x0000000000400f60 in phase_3 ()
```
此时我们查看后续代码可以知道主要使用的(rsp+0x8)和(rsp+0xc)地址的数据，可以怀疑这里存的就是第三关输入的两个数。使用下面的一系列指令查看这两个位置的值：
```
(gdb) print &rsp
No symbol "rsp" in current context.
(gdb) print $rsp
$1 = (void *) 0x7fffffffdd10
(gdb) x/2wd 0x7fffffffdd18
0x7fffffffdd18:	1	2
```
可以看出我们的猜测是正确的。接下来单步运行，从以下几条指令可以看出，输入的第一个数必须小于7。
```
   0x0000000000400f6a <+39>:	cmpl   $0x7,0x8(%rsp)
   0x0000000000400f6f <+44>:	ja     0x400fad <phase_3+106>
   ...
   0x0000000000400fad <+106>:	callq  0x40143a <explode_bomb>
```
继续单步运行，由于我第一个数输入的是一，`<+50>:	jmpq   *0x402470(,%rax,8)`这条指令使程序运行到了`<+118>`，从下面的汇编代码可以看出，输入的第二个数必须等于0x137(也就是311)。
```
=> 0x0000000000400fb9 <+118>:	mov    $0x137,%eax
   0x0000000000400fbe <+123>:	cmp    0xc(%rsp),%eax
   0x0000000000400fc2 <+127>:	je     0x400fc9 <phase_3+134>
   0x0000000000400fc4 <+129>:	callq  0x40143a <explode_bomb>
   0x0000000000400fc9 <+134>:	add    $0x18,%rsp
```
所以我们就得到了一个答案`1 311`，重新运行`bomb`，得到结果如下，表示第三关通过：
```
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Border relations with Canada have never been better.
Phase 1 defused. How about the next one?
1 2 4 8 16 32
That's number 2.  Keep going!
1 311
Halfway there!
```
这一关应该还有很多答案，我这是通过单步调试找出来的答案，其实仔细回头看`phase_3`函数，可以发现指令`<+46>`到`<+104>`类似于C++中的switch结构，分别第一个输入数的对应八种情况，表示当第一个输入数为`0，1，2，3，4，5，6，7`时，第二个输入数应该分别为`207，311，707，256，389，206，682，327`

## 第四关：phase_4
使用disas查看`phase_4`函数的反汇编代码：
```
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
```
这一关输入部分很像第三关，同样是输入两个整数。但是第四关主要的运算过程在一个新的函数`func4`中，使用disas查看这个的反汇编代码：
```
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
从反汇编代码中看出函数自己调用了自己，应该是一个递归函数，但是查看下列反汇编代码，发现好像可以投机不需要进入递归，就能安全完成这个函数：
```
   0x0000000000400fe2 <+20>:	cmp    %edi,%ecx
   0x0000000000400fe4 <+22>:	jle    0x400ff2 <func4+36>
   ...
   0x0000000000400ff2 <+36>:	mov    $0x0,%eax
   0x0000000000400ff7 <+41>:	cmp    %edi,%ecx
   0x0000000000400ff9 <+43>:	jge    0x401007 <func4+57>
   ...
   0x0000000000401007 <+57>:	add    $0x8,%rsp
   0x000000000040100b <+61>:	retq   
```
这一段代码的意思其实就是，如果%ecx==%edi，就直接返回phase_4，通过分析func4函数前面的赋值过程，可以很容易知道edi寄存器存的是输入的第一个整数，ecx寄存器得出的值0xe除以2，也就是7，所以我们就确定了第一个输入的整数应该为7。
再回到`phase_4`的最后一部分代码，如下：
```
   0x0000000000401051 <+69>:	cmpl   $0x0,0xc(%rsp)
   0x0000000000401056 <+74>:	je     0x40105d <phase_4+81>
   0x0000000000401058 <+76>:	callq  0x40143a <explode_bomb>
   0x000000000040105d <+81>:	add    $0x18,%rsp
   0x0000000000401061 <+85>:	retq 
```
可以看出第二个输入数应该为0，所以`phase_4`的一个答案为`7 0`，由于我没有具体查看递归函数`func`，所以应该还有其他正确答案可以通关。第四关通关结果如下：
```
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Border relations with Canada have never been better.
Phase 1 defused. How about the next one?
1 2 4 8 16 32
That's number 2.  Keep going!
1 311
Halfway there!
7 0
So you got that one.  Try this one.
```
## 第五关：phase_5
使用disas查看`phase_5`函数的反汇编代码：
```
Dump of assembler code for function phase_5:
   0x0000000000401062 <+0>:     push   %rbx
   0x0000000000401063 <+1>:     sub    $0x20,%rsp
   0x0000000000401067 <+5>:     mov    %rdi,%rbx
   0x000000000040106a <+8>:     mov    %fs:0x28,%rax
   0x0000000000401073 <+17>:    mov    %rax,0x18(%rsp)
   0x0000000000401078 <+22>:    xor    %eax,%eax
   0x000000000040107a <+24>:    callq  0x40131b <string_length>
   0x000000000040107f <+29>:    cmp    $0x6,%eax
   0x0000000000401082 <+32>:    je     0x4010d2 <phase_5+112>
   0x0000000000401084 <+34>:    callq  0x40143a <explode_bomb>
   0x0000000000401089 <+39>:    jmp    0x4010d2 <phase_5+112>
   0x000000000040108b <+41>:    movzbl (%rbx,%rax,1),%ecx
   0x000000000040108f <+45>:    mov    %cl,(%rsp)
   0x0000000000401092 <+48>:    mov    (%rsp),%rdx
   0x0000000000401096 <+52>:    and    $0xf,%edx
   0x0000000000401099 <+55>:    movzbl 0x4024b0(%rdx),%edx
   0x00000000004010a0 <+62>:    mov    %dl,0x10(%rsp,%rax,1)
   0x00000000004010a4 <+66>:    add    $0x1,%rax
   0x00000000004010a8 <+70>:    cmp    $0x6,%rax
   0x00000000004010ac <+74>:    jne    0x40108b <phase_5+41>
   0x00000000004010ae <+76>:    movb   $0x0,0x16(%rsp)
   0x00000000004010b3 <+81>:    mov    $0x40245e,%esi
   0x00000000004010b8 <+86>:    lea    0x10(%rsp),%rdi
   0x00000000004010bd <+91>:    callq  0x401338 <strings_not_equal>
   0x00000000004010c2 <+96>:    test   %eax,%eax
   0x00000000004010c4 <+98>:    je     0x4010d9 <phase_5+119>
   0x00000000004010c6 <+100>:   callq  0x40143a <explode_bomb>
   0x00000000004010cb <+105>:   nopl   0x0(%rax,%rax,1)
   0x00000000004010d0 <+110>:   jmp    0x4010d9 <phase_5+119>
   0x00000000004010d2 <+112>:   mov    $0x0,%eax
   0x00000000004010d7 <+117>:   jmp    0x40108b <phase_5+41>
   0x00000000004010d9 <+119>:   mov    0x18(%rsp),%rax
   0x00000000004010de <+124>:   xor    %fs:0x28,%rax
   0x00000000004010e7 <+133>:   je     0x4010ee <phase_5+140>
   0x00000000004010e9 <+135>:   callq  0x400b30 <__stack_chk_fail@plt>
   0x00000000004010ee <+140>:   add    $0x20,%rsp
   0x00000000004010f2 <+144>:   pop    %rbx
   0x00000000004010f3 <+145>:   retq
End of assembler dump.
```
从下面一段代码可以看出，这一关的输入应该是一个长为6的字符串：
```
   0x000000000040107a <+24>:    callq  0x40131b <string_length>
   0x000000000040107f <+29>:    cmp    $0x6,%eax
   0x0000000000401082 <+32>:    je     0x4010d2 <phase_5+112>
   0x0000000000401084 <+34>:    callq  0x40143a <explode_bomb>
```
这样我们就可以先随便输入一个字符串：`abcdef`，方便后面单步调试运行：
首先设置断点`phase_5`，在运行`run`，输入前四关的正确答案和第五关的`abcdef`：
```
(gdb) break phase_5
Breakpoint 1 at 0x401062
(gdb) run
Starting program: /mnt/c/Users/xzjqx/Desktop/Study/计算机系统/CMU实验_2018/bomblab/bomb/bomb
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Border relations with Canada have never been better.
Phase 1 defused. How about the next one?
1 2 4 8 16 32
That's number 2.  Keep going!
1 311
Halfway there!
7 0
So you got that one.  Try this one.
abcdef

Breakpoint 1, 0x0000000000401062 in phase_5 ()
```
之后的代码中，下面这一段是一个loop：
```
   0x000000000040108b <+41>:    movzbl (%rbx,%rax,1),%ecx
   0x000000000040108f <+45>:    mov    %cl,(%rsp)
   0x0000000000401092 <+48>:    mov    (%rsp),%rdx
   0x0000000000401096 <+52>:    and    $0xf,%edx
   0x0000000000401099 <+55>:    movzbl 0x4024b0(%rdx),%edx
   0x00000000004010a0 <+62>:    mov    %dl,0x10(%rsp,%rax,1)
   0x00000000004010a4 <+66>:    add    $0x1,%rax
   0x00000000004010a8 <+70>:    cmp    $0x6,%rax
   0x00000000004010ac <+74>:    jne    0x40108b <phase_5+41>
```
我们在`0x40108b`处设置断点，然后输入`continue`继续运行程序，再单步运行，并查看寄存器的值，全部操作如下所示：
```
(gdb) print $rbx
$1 = 6305984
(gdb) print $rax
$2 = 0
(gdb) stepi
0x000000000040108f in phase_5 ()
(gdb) print $ecx
$3 = 97
(gdb) print $cl
$4 = 97
(gdb) stepi
0x0000000000401092 in phase_5 ()
(gdb) stepi
0x0000000000401096 in phase_5 ()
(gdb) print $rdx
$5 = 4203105
(gdb) stepi
0x0000000000401099 in phase_5 ()
(gdb) print $rdx
$6 = 1
(gdb) x/s 6305984
0x6038c0 <input_strings+320>:   "abcdef"
```
上述测试结果可以看出：在这个循环中rbx寄存器是输入的字符串的基址，循环依次取一个字符，获取该字符ASCII码的低四位存放在edx寄存器中。继续单步运行，出现一个地址`0x4024b0`，查看如下：
```
(gdb) x/s 0x4024b0
0x4024b0 <array.3449>:  "maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?"
```
可以看出这也是一个字符串（也可以看作字符数组）。

这样就能理解这一段循环的具体意义：按照输入的六个字符的ASCII码低四位为索引，从一个字符串中取出六个新的字符，存入栈帧`(rsp+0x0)~(rsp+0x5)`处。

继续运行会发现一个新的地址，通过命令查看存放的是一个六位的字符串：

```
(gdb) x/s 0x40245e
0x40245e:       "flyers"
```
接下来要比较这个字符串和上述索引所得的新字符串，也就是说我们需要输入一个字符串，通过一个索引，或者一个新的字符串"flyers"，这个通过计算可以知道输入的字符串是`ionefg`，我们重新运行`bomb`程序测试第五关是否通过：
```
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Border relations with Canada have never been better.
Phase 1 defused. How about the next one?
1 2 4 8 16 32
That's number 2.  Keep going!
1 311
Halfway there!
7 0
So you got that one.  Try this one.
ionefg
Good work!  On to the next...
```
第五关也正确通过。
## 第六关：phase_6
这一关的反汇编代码有点长，我们分段来看：

```
   0x00000000004010fc <+8>:     sub    $0x50,%rsp
   0x0000000000401100 <+12>:    mov    %rsp,%r13
   0x0000000000401103 <+15>:    mov    %rsp,%rsi
   0x0000000000401106 <+18>:    callq  0x40145c <read_six_numbers>
```

最开始调用了一个`read_six_numbers`代表输入是六个数字，这样就可以采用单步调试并查看寄存器内存的方式，来理解反汇编代码（先输入`1 2 3 4 5 6`）：

```
(gdb) break phase_6
Breakpoint 1 at 0x4010f4
(gdb) run
Starting program: /mnt/c/Users/xzjqx/Desktop/Study/计算机系统/CMU实验_2018/bomblab/bomb/bomb
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Border relations with Canada have never been better.
Phase 1 defused. How about the next one?
1 2 4 8 16 32
That's number 2.  Keep going!
1 311
Halfway there!
7 0
So you got that one.  Try this one.
ionefg
Good work!  On to the next...
1 2 3 4 5 6

Breakpoint 1, 0x00000000004010f4 in phase_6 ()
(gdb) break *0x40110b
Breakpoint 2 at 0x40110b
(gdb) continue
Continuing.

Breakpoint 2, 0x000000000040110b in phase_6 ()
(gdb) x/6d $rsp
0x7ffffffedf00: 1       2       3       4
0x7ffffffedf10: 5       6
```
可以看出输入为六个32位数，且存入栈帧`(rsp+0x0)~(rsp+0x14)`处。

之后一段是一个循环，我把每一句都换成类似C语言的注释，能更好的理解这段代码的作用：

```
   0x000000000040110b <+23>:    mov    %rsp,%r14			# r14 = rsp 
   0x000000000040110e <+26>:    mov    $0x0,%r12d			# r12 = 0
   0x0000000000401114 <+32>:    mov    %r13,%rbp			# rbp = r13
   0x0000000000401117 <+35>:    mov    0x0(%r13),%eax		# eax = [r13]
   0x000000000040111b <+39>:    sub    $0x1,%eax			# eax = eax -1
   0x000000000040111e <+42>:    cmp    $0x5,%eax			# if(eax > 5)
   0x0000000000401121 <+45>:    jbe    0x401128 <phase_6+52>	#     return "boom"
   0x0000000000401123 <+47>:    callq  0x40143a <explode_bomb>
   0x0000000000401128 <+52>:    add    $0x1,%r12d			# r12 = r12 + 1
   0x000000000040112c <+56>:    cmp    $0x6,%r12d			# if(r12 == 6)
   0x0000000000401130 <+60>:    je     0x401153 <phase_6+95>	#     jump <phase_6+95>
   0x0000000000401132 <+62>:    mov    %r12d,%ebx			# ebx = r12
   0x0000000000401135 <+65>:    movslq %ebx,%rax			# rax = ebx
   0x0000000000401138 <+68>:    mov    (%rsp,%rax,4),%eax	# eax = [rsp+rax*4]
   0x000000000040113b <+71>:    cmp    %eax,0x0(%rbp)		# if(eax == [rbp])
   0x000000000040113e <+74>:    jne    0x401145 <phase_6+81>	#     return "boom"
   0x0000000000401140 <+76>:    callq  0x40143a <explode_bomb>
   0x0000000000401145 <+81>:    add    $0x1,%ebx			# ebx = ebx + 1
   0x0000000000401148 <+84>:    cmp    $0x5,%ebx			# if(ebx <= 5)
   0x000000000040114b <+87>:    jle    0x401135 <phase_6+65>	#     jump <phase_6+65>
   0x000000000040114d <+89>:    add    $0x4,%r13			# r13 = r13 + 4
   0x0000000000401151 <+93>:    jmp    0x401114 <phase_6+32>	# jump <phase_6+32>
```

根据翻译来的注释可以看出，这是一个二重循环：第一重循环确定每个输入都不大于6；第二重循环确定所有输入两两不相等。

接下来也是一段循环：

```
   0x0000000000401153 <+95>:    lea    0x18(%rsp),%rsi
   0x0000000000401158 <+100>:   mov    %r14,%rax
   0x000000000040115b <+103>:   mov    $0x7,%ecx
   0x0000000000401160 <+108>:   mov    %ecx,%edx
   0x0000000000401162 <+110>:   sub    (%rax),%edx
   0x0000000000401164 <+112>:   mov    %edx,(%rax)
   0x0000000000401166 <+114>:   add    $0x4,%rax
   0x000000000040116a <+118>:   cmp    %rsi,%rax
   0x000000000040116d <+121>:   jne    0x401160 <phase_6+108>
```

这段代码用7减去每一个输入，作为新的输入，运行程序再查看rsp寄存器地址存的六个32位数即可确认：

```
(gdb) break *0x40116f
Breakpoint 3 at 0x40116f
(gdb) continue
Continuing.

Breakpoint 3, 0x000000000040116f in phase_6 ()
(gdb) x/6d $rsp
0x7ffffffedf00: 6       5       4       3
0x7ffffffedf10: 2       1
```

下一段代码中有一个地址`0x6032d0` ，我们先查看其中的内容：

```
(gdb) x/24d 0x6032d0
0x6032d0 <node1>:       332     1       6304480 0
0x6032e0 <node2>:       168     2       6304496 0
0x6032f0 <node3>:       924     3       6304512 0
0x603300 <node4>:       691     4       6304528 0
0x603310 <node5>:       477     5       6304544 0
0x603320 <node6>:       443     6       0       0
```

结果类似一个结构体，下一段代码的作用就是使用我们输入的六位数对这个结构体重新排序，存入(rsp+0x18)处，可以设置断点查看重新排序的结果：

```
(gdb) x/12d $rsp+0x18
0x7ffffffedf18: 0       0       6304544 0
0x7ffffffedf28: 6304528 0       6304512 0
0x7ffffffedf38: 6304496 0       6304480 0
```

可以看出，是按照`6 5 4 3 2 1`的顺序重新排列了，原因是之前经过了7-input的过程。

最后一段代码判断结构体中的第一个数字是否是降序排列，要排序的六个数是`332,168,924,691,477,443`要达到降序排列，需要把第三个结点放在第一位，第四个放在第二位......这样得出的输入序列为`3 4 5 6 1 2`，由于之前有一个7-input的过程，故正确答案应该是`4 3 2 1 6 5`。

最终结果为：

```
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Border relations with Canada have never been better.
Phase 1 defused. How about the next one?
1 2 4 8 16 32
That's number 2.  Keep going!
1 311
Halfway there!
7 0
So you got that one.  Try this one.
ionefg
Good work!  On to the next...
4 3 2 1 6 5
Congratulations! You've defused the bomb!
```

所有炸弹都被拆除！