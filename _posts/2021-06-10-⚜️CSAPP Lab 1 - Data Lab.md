---
layout: post
author: syea
title: CSAPP Lab 1 - Data Lab
# subtitle:
# header-img: 
# header-mask:  
# catalog: 
# multilingual: 
tags:
    - OS
---

# CSAPP Lab 1 - Data Lab

## 前言

夫天地者，万物之逆旅也；光阴者，百代之过客也。而浮生若梦，为欢几何.    
如果对 `敲下代码` 到 `应用运行` 再到 `网络通信` 整个过程都有了一个比较完整的认知,对人生中当`程序员`的这些年也算有个交代.

## 实验前提    
1. 看完《CSAPP》书本第二章
2. 看完作者第二章讲课视频(非必要,但是建议看看)

## 学习后的收获    
1. 了解整数在计算机中的表示
2. 了解位运算 `| & ^ ~ << >> `
3. **了解浮点数在计算机中是如何实现的**
4. 了解浮点数的运算在计算机中是如何实现的
5. **了解浮点数与整数的转换逻辑,什么情况下会失去精度**
6. **了解 > < ! 在计算机中是如何实现的**

## 实验详解

```
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 *
 * 思路: x & y 可以得出两者都为 1 的位
 *      ~x & ~y 可以得出两者都为 0 的位
 *      两个结果 | 一下可以得出两者所有相同的位
 *      取反
 */
int bitXor(int x, int y) {
        return ~((x & y) | (~x & ~y));
}
```

```
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 *
 * 思路: 没什么好说的 = =
 */
int tmin(void) {
        return 1 << 31;
}
```

```
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 *
 * 思路: tmax = 0111 1111 1111 1111 1111 1111 1111 1111
 *      1. 它的最高位是 0 =>  x >> 31 == 0
 *      2. 它跟 1 << 31 | 运算 == -1, ~ == 0
 *      3. 满足以上两个条件则 return 1, => !(a | b) 
 */
int isTmax(int x) {
  return !(~(x | (1 << 31)) | (x >> 31));
}
```

```
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 *
 * 思路: 如果 x 奇数位上都是 1, 则 x 与偶数位上都是 1 的数 | 运算, 结果为 -1
 */
int allOddBits(int x) {
  int y = 0x55;
  int z = y + (y << 8) + (y << 16) + (y << 24);
  return !~(x | z);
}
```

```
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 * 思路: 标准解法
 */
int negate(int x) {
  return ~x + 1;
}
```

```
/* 
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 *
 * 思路: 区间两端的值跟 x 做差,最高位需要等于 0
 */
int isAsciiDigit(int x) {
        int y = (x + ~0x30 + 1) >> 31;
        int z = (0x39 + ~x + 1) >> 31;
        return !y & !z;
}
```

```
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 *
 * 思路: 固定公式 (t & a) | (~t & b),  当 t == 0 时, 表示取 b, t == -1 时, 表示取 a
 */
int conditional(int x, int y, int z) {
        int t = !!x;
        t = ~t + 1;
        return (t & y) | (~t & z);
}
```

```
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 *
 * 思路: 通过将两个数 >> 31 可以判断他们的符号是否相同
 *      1. 相同 => 相减最高位为 0 则大于等于
 *      2. 不同 => y 为正则大于等于
 */
int isLessOrEqual(int x, int y) {
        int t = ((x >> 31) ^ (y >> 31));
        int a = (y + ~x + 1) >> 31;
        int b = y >> 31;
        return !((~t & a) | (t & b));
}
```

```
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 *
 * 思路: x | (~x + 1) 如果最高位为 1 则不为0,  反之为 0
 */
int logicalNeg(int x) {
        return ((x | (~x + 1)) >> 31) + 1;
}
```

```
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 *
 * 思路: 右移后判断当前数值是否为 0, 为0则表示右移的最高位都为 0
 */
int howManyBits(int x) {
        int b16,b8,b4,b2,b1,b0;
        int sign = x >> 31;
        x = (~sign & x) | (sign & ~x);
        b16 = !!(x >> 16) << 4;
        x = x >> b16;
        b8 = !!(x >> 8) << 3;
        x = x >> b8;
        b4 = !!(x >> 4) << 2;
        x = x >> b4;
        b2 = !!(x >> 2) << 1;
        x = x >> b2;
        b1 = !!(x >> 1);
        x = x >> b1;
        b0 = x;
        return b16 + b8 + b4 + b2 + b1 + b0 + 1;
}
```

剩余 3 题浮点数相关的未完成.

## 总结






