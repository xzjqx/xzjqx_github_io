---
title: 深入理解计算机系统（CSAPP）实验一 Data Lab
date: 2018-04-13 19:18:03
tags: 
	- C/C++
	- CSAPP
categories: 计算机系统
---

## bitAnd - x&y using only ~ and |

目标：不使用&运算符计算按位与运算
方法：运用德·摩根定律：非(P 且 Q) = (非 P) 或 (非 Q)
```
int bitAnd(int x, int y) {
  return ~((~x) | (~y));
}
```
<!-- more -->
## getByte - Extract byte n from word x

目标：提取一个字中的特定一个字节
方法：将字向右移位到最低字节，再与0xff按位与得到目标字节

```
int getByte(int x, int n) {
  return (x >> (n << 3)) & 0xff;
}
```
## logicalShift - shift x to the right by n, using a logical shift

目标：将x逻辑右移n位
方法：>>操作符是算术右移，故使用>>右移后，将高n位置零即可

```
int logicalShift(int x, int n) {
  int mask = ~(1 << 31);   // mask = 0x7fffffff
  mask = mask >> n;
  mask = mask << 1;
  mask = mask + 1;         // mask高n位为0，低位为1
  return (x >> n) & mask;
}
```
## bitCount - returns count of number of 1's in word

目标：计算x中有多少位1
方法：将x分为四个字节，分别计算1的数量（共计算八次），最后将结果分为四个字节计算总和即为最终答案

```
int bitCount(int x) {
  int result = 0;
  int mask = 1 | (1 << 8);
  mask = mask | (mask << 16);         // mask = 0x01010101
  result = result + (x & mask);
  result = result + ((x>>1) & mask);
  result = result + ((x>>2) & mask);
  result = result + ((x>>3) & mask);
  result = result + ((x>>4) & mask);
  result = result + ((x>>5) & mask);
  result = result + ((x>>6) & mask);
  result = result + ((x>>7) & mask);  // result的每一个字节为x在这个字节上1的数量
  return (result & 0xff) + ((result>>8) &0xff) + ((result>>16) &0xff) + ((result>>24) &0xff);
}
```
## bang - Compute !x without using !

目标：不使用!运算符计算!x
方法：将x与-x进行或运算，若x为0，最高位为0，反之最高位为1；将结果右移31位得到0xffffffff或0x00000000，加上1即为最终结果

```
int bang(int x) {
  int tmp = ~x + 1;       // tmp = -x;
  tmp = x | tmp;          // tmp = x | -x; 若x=0，最高位为0，反之最高位为1
  return (tmp >> 31) + 1;
}
```
## tmin - return minimum two's complement integer

目标：求得二进制整数的最小值
方法：1左移31位即可

```
int tmin(void) {
  return 1<<31;
}
```
## fitsBits - return 1 if x can be represented as an n-bit, two's complement integer.

目标：如果x能被n位二进制补码表示，则返回1
方法：如果x能被n位二进制表示，则x前(32-n)位都是相同的

```
int fitsBits(int x, int n) {
  return !((x << (33 + ~n) >> (33 + ~n)) ^ x);
}
```
## divpwr2 - Compute x/(2^n), for 0 ~ n ~ 30

目标：计算 x/(2^n)
方法：对于正数，直接右移n位即可；对于负数，需要加上偏置量bias后右移

```
int divpwr2(int x, int n) {
  int sign = x >> 31;        // 若x为负数，sign = 0xffffffff
  int mask = (1 << n) + ~0; // mask为0x0000ffff形式
  int bias = sign & mask;   // 若x为负数，bias有值
  return (x + bias) >> n;
}
```
## negate - return -x

目标：求相反数
方法：补码取相反数，包含符号位一起取反再加一

```
int negate(int x) {
  return ~x + 1;
}
```
## isPositive - return 1 if x > 0, return 0 otherwise

目标：判断x是否为正数
方法：直接通过x的符号位可以分出负数，再对x两次取非可以判断是否为0

```
int isPositive(int x) {
  return (!((x >> 31) & 1) & !!x);
}
```
## isLessOrEqual - if x is less or euqal y then return 1, else return 0

目标：判断是否x <= y
方法：当x与y异号时，只要x为负数，x<=y；当x与y同号时，利用isPositive函数的方法判断y-x是否为正数或0即可

```
int isLessOrEqual(int x, int y) {
  int signx = (x >> 31) & 1;
  int signy = (y >> 31) & 1;
  int sign = (signx ^ signy) & signx;               	 // 异号
  int tmp = y + ~x + 1;
  tmp = (!((tmp >> 31) & 1)) & (!(signx ^ signy));  // 同号
  return (sign | tmp);
}
```
## ilog2 - return floor(log base 2 of x), where x > 0

目标：求floor(log2x)
方法：通过二分法找到x中最高位1的位数，这个位数就是要求出的结果

```
int ilog2(int x) {
  int bits=0;
  bits = (!!(x>>16))<<4;
  bits = bits + ((!!(x>>(bits+8)))<<3);
  bits = bits + ((!!(x>>(bits+4)))<<2);
  bits = bits + ((!!(x>>(bits+2)))<<1);
  bits = bits + (!!(x>>(bits+1)));
  bits = bits + (!!bits)+(~0)+(!(1^x));
  return bits;
}
```
## float_neg - Return bit-level equivalent of expression -f for floating point argument f

目标：将浮点数uf取反
方法：若uf不是NaN，直接将符号位取反，否则返回uf

```
unsigned float_neg(unsigned uf) {
  unsigned result;
  unsigned tmp;
  result=uf ^ 0x80000000; //将符号位改反
  tmp=uf & (0x7fffffff);
  if(tmp > 0x7f800000)    //此时是NaN
    result = uf;
  return result;
}
```
## float_i2f - Return bit-level equivalent of expression (float) x

目标：返回整数x的对应的浮点数的二进制表示形式
方法：这是一个求阶码与尾数的过程
		
由于整数x一定是规格化数，故按照规格化标准转化；因为规格化的值尾数定义为M=1+f，故将x左移到最高一位1超出32位时，剩下的即为尾数；阶码是32减去上述转移的位数，故阶码字段值为127+32-shift

```
unsigned float_i2f(int x) {
  unsigned sign = 0;         // 符号位
  unsigned exp;              // 阶码
  unsigned frac;             // 尾数
  unsigned shift = 0, tmp, rounding = 0;

  if(x == 0) return x;       // 为0时直接返回0
  if(x < 0) {
    x = -x;
    sign = 1;
  }                          // x取绝对值，记录符号位
  frac = x;
  while(1) {
    tmp = frac;
    shift ++;
    frac <<= 1;
    if(tmp & 0x80000000) break;
  }                          
  exp = (159 - shift) << 23; // 计算阶码
  if((frac & 0x1ff) > 0x100) rounding = 1;        // 向上舍入
  else if((frac & 0x3ff) == 0x300) rounding = 1;  // 向上舍入
  frac = (frac >> 9) + rounding; // 计算尾数

  return (sign << 31) + exp + frac;
}
```
## float_twice - Return bit-level equivalent of expression 2*f for

目标：返回浮点uf的两倍（以unsigned的形式）
方法：对于非规格化的数，uf的两倍等价于尾数左移一位，即小数字段左移移位；对于规格化的数，uf的两倍等价于阶码加一，即阶码字段加一

```
unsigned float_twice(unsigned uf) {
  unsigned f = uf;
  if ((f & 0x7F800000) == 0) {                  // 非规格化的
    f = ((f & 0x007FFFFF) << 1) | (0x80000000 & f);
  }
  else if ((f & 0x7F800000) != 0x7F800000) {    // 规格化的
    f = f + 0x00800000;
  }
  return f;
}
```

