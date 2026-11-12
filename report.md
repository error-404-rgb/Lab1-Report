---
title: "计算机系统基础 II Lab1 - DataLab 实验报告"
author: "姓名：范佳艺"
date: "2026年11月12日"
geometry: margin=2.5cm
fontsize: 12pt
linestretch: 1.5
header-includes:
  - \usepackage{xcolor}
  - \usepackage{tcolorbox}
  - \usepackage{hyperref}
  - \hypersetup{colorlinks=true, linkcolor=blue, urlcolor=blue}
  - \newtcolorbox{mybluebox}{colback=blue!5!white, colframe=blue!50!black, boxrule=0.5pt, arc=3pt, left=5pt, right=5pt, top=5pt, bottom=5pt}
---

\tableofcontents
\newpage

# 一、 实验环境配置与开发工具选择

在 Linux 虚拟机中，首先使用官方脚本完成了 `datalab` 目录的初始化。

![实验环境初始化](1_init.png)

在实验初期，我尝试在虚拟机的终端中直接使用文本编辑器修改代码。但终端操作在处理长文件时不够直观，稍有不慎就会误触或破坏原文件中的注释。为了提高开发效率，我查阅了讲义提示，通过配置 VirtualBox 的 NAT 端口转发，在 Windows 宿主机上使用 VSCode 的 SSH 插件连接到了虚拟机。此后，所有的代码编写、测试与规范检查均在 VSCode 中配合终端完成。

\newpage

# 二、 题目实现与逻辑推导

本次实验共包含 13 个函数，主要考察位运算、补码特性以及浮点数标准。以下是各题的详细推导过程与实现代码。

## 1. 基础整数位运算

### 1.1 bitXor
\begin{mybluebox}
\textbf{题目要求：}仅使用 \textasciitilde{} 和 \& 实现异或运算。\\
\textbf{推导过程：}异或的定义是“相同为 0，不同为 1”，逻辑表示为 (x \& \textasciitilde{}y) | (\textasciitilde{}x \& y)。因禁止使用按位或 |，应用德摩根定律进行转换：A | B = \textasciitilde{}(\textasciitilde{}A \& \textasciitilde{}(\textasciitilde{}B))。将式子代入即可。
\end{mybluebox}

```c
int bitXor(int x, int y) {
  return ~(~(x&~y)&~(~x&y));
}
```

### 1.2 tmin
\begin{mybluebox}
\textbf{题目要求：}返回 32 位补码最小值。\\
\textbf{推导过程：}32 位有符号整数最小值的二进制为 0x80000000（最高位为 1）。直接将整型常量 1 左移 31 位即可。
\end{mybluebox}

```c
int tmin(void) {
  return 1<<31;
}
```

### 1.3 isTmax
\begin{mybluebox}
\textbf{题目要求：}判断 x 是否为 32 位有符号最大值，禁用位移。\\
\textbf{推导过程：}最大值 0x7FFFFFFF 满足 \textasciitilde{}x == x + 1。但特例 -1 也满足此等式。为了排除 -1，利用逻辑非：对 -1 取反为 0，!!\textasciitilde{}x 得 0；对最大值取反非 0，逻辑非得 1。
\end{mybluebox}

```c
int isTmax(int x) {
  return !(~x^(x+1))&!!~x;
}
```

### 1.4 allOddBits
\begin{mybluebox}
\textbf{题目要求：}若所有奇数位都为 1 则返回 1，禁用大常量。\\
\textbf{推导过程：}32 位奇数位全 1 的掩码为 0xAAAAAAAA。用小常量 0xAA 拼接出该掩码。将 x 与掩码相与，判断结果是否等于掩码。
\end{mybluebox}

```c
int allOddBits(int x) {
  int mask = (0xAA<<24)|(0xAA<<16)|(0xAA<<8)|0xAA;
  return !((x&mask)^mask);
}
```

### 1.5 negate
\begin{mybluebox}
\textbf{题目要求：}返回 -x。\\
\textbf{推导过程：}补码系统中，按位取反后加一。
\end{mybluebox}

```c
int negate(int x) {
  return ~x+1;
}
```

## 2. 进阶逻辑与比较

### 2.1 isAsciiDigit
\begin{mybluebox}
\textbf{题目要求：}判断 x 是否在 0x30 到 0x39 之间。\\
\textbf{推导过程：}分别计算 x - 0x30 和 0x39 - x。若两者结果符号位皆为 0，说明在范围内。
\end{mybluebox}

```c
int isAsciiDigit(int x) {
  int lower = x+(~0x30+1);
  int upper = 0x39+(~x+1);
  return !((lower|upper)>>31);
}
```

### 2.2 conditional
\begin{mybluebox}
\textbf{题目要求：}实现 x ? y : z。\\
\textbf{推导过程：}利用 \textasciitilde{}(!!x)+1 构造掩码：x 非 0 时为全 1，为 0 时为全 0。将掩码与 y 相与，反码与 z 相与，合并。
\end{mybluebox}

```c
int conditional(int x, int y, int z) {
  return ((~(!!x)+1)&y)|(~(~(!!x)+1)&z);
}
```

### 2.3 isLessOrEqual
\begin{mybluebox}
\textbf{题目要求：}判断 x <= y。\\
\textbf{推导过程：}防止相减溢出，分情况讨论。提取符号位，异号直接判断；同号再判断 y - x 的符号位。
\end{mybluebox}

```c
int isLessOrEqual(int x, int y) {
  int sx = x>>31;
  int sy = y>>31;
  int diff = y+~x+1;
  int sd = diff>>31;
  return !!((sx&~sy)|(~(sx^sy)&~sd));
}
```

### 2.4 logicalNeg
\begin{mybluebox}
\textbf{题目要求：}实现逻辑非 !。\\
\textbf{推导过程：}若 x 为 0，x | (\textasciitilde{}x+1) 仍为 0；若非 0，正负数必有符号位为 1。右移 31 位后加 1。
\end{mybluebox}

```c
int logicalNeg(int x) {
  int y = x|(~x+1);
  return (y>>31)+1;
}
```

## 3. 复杂算法与浮点数

### 3.1 howManyBits
\begin{mybluebox}
\textbf{题目要求：}求表示 x 所需的最少补码位数。\\
\textbf{推导过程：}二分法，依次判断高 16、8、4 位等是否有 1，累加跨度并加符号位。
\end{mybluebox}

```c
int howManyBits(int x) {
  int sign = x>>31;
  int b16,b8,b4,b2,b1,b0;
  x = x^sign;
  b16 = !!(x>>16)<<4; x = x>>b16;
  b8 = !!(x>>8)<<3; x = x>>b8;
  b4 = !!(x>>4)<<2; x = x>>b4;
  b2 = !!(x>>2)<<1; x = x>>b2;
  b1 = !!(x>>1)<<0; x = x>>b1;
  b0 = x;
  return b16+b8+b4+b2+b1+b0+1;
}
```

### 3.2 floatScale2
\begin{mybluebox}
\textbf{题目要求：}浮点数乘 2。\\
\textbf{推导过程：}阶码为 0xFF 原样返回；阶码为 0 左移尾数；普通规格化数阶码加 1。
\end{mybluebox}

```c
unsigned floatScale2(unsigned uf) {
  unsigned sign = (uf>>31)&1;
  unsigned exp = (uf>>23)&0xFF;
  unsigned frac = uf&0x7FFFFF;
  if(exp==0xFF) { return uf; }
  if(exp==0) { frac = frac<<1; }
  else { exp = exp+1; }
  return (sign<<31)|(exp<<23)|frac;
}
```

### 3.3 floatFloat2Int
\begin{mybluebox}
\textbf{题目要求：}浮点数转整数。\\
\textbf{推导过程：}真实阶数 E = exp - 127。E < 0 返回 0；E > 30 返回 0x80000000。恢复隐藏的 1 并根据 E 移位。
\end{mybluebox}

```c
int floatFloat2Int(unsigned uf) {
  int sign;
  int exp;
  int frac;
  int result;
  sign = uf>>31;
  exp = ((uf>>23)&0xFF)-127;
  frac = uf&0x7FFFFF;
  if(exp<0) { return 0; }
  if(exp>30) { return 1<<31; }
  frac = frac|(1<<23);
  if(exp>23) { result = frac<<(exp-23); }
  else { result = frac>>(23-exp); }
  if(sign) { return -result; }
  return result;
}
```

### 3.4 floatPower2
\begin{mybluebox}
\textbf{题目要求：}计算 2.0 的 x 次方。\\
\textbf{推导过程：}2.0\textasciicircum{}x 只有阶码改变。x+127 为阶码。越界返回特定值，否则阶码左移 23 位。
\end{mybluebox}

```c
unsigned floatPower2(int x) {
  int exp;
  if(x>127) { return (0xFF<<23); }
  if(x<-126) { return 0; }
  exp = x+127;
  return exp<<23;
}
```

\newpage

# 三、 实验中遇到的问题与解决过程

在本地运行 `./btest` 时逻辑完全正确，但在使用 `./dlc` 语法检查器时遇到了多次报错。

## 问题 1：变量声明位置导致编译失败
\begin{mybluebox}
\textbf{现象：}出现大量 parse error 和 undeclared variable。\\
\textbf{原因：} dlc 按照 C89 标准检查，要求变量声明必须在函数最前端。\\
\textbf{解决：}将所有的 int 变量提取到第一行声明，再进行赋值。
\end{mybluebox}

![变量声明报错](2_dlc_19errors.png)

## 问题 2：使用了非法操作符
\begin{mybluebox}
\textbf{现象：} bits.c:165:isTmax: Illegal operator (>>)\\
\textbf{原因：} isTmax 中使用了右移操作 >>，但题目不允许。\\
\textbf{解决：}利用 \textasciitilde{}x == x+1 特性及 !!\textasciitilde{}x 重写判断逻辑。
\end{mybluebox}

![非法操作符报错](3_illegal_op.png)

## 问题 3：边界值测试与大常量解析
\begin{mybluebox}
\textbf{现象：}得分 32/36，且出现 bits.c:283: parse error。\\
\textbf{原因：} howManyBits 处理负数移位出错；floatPower2 的超大常量 0x7F800000 导致老旧解析器崩溃。\\
\textbf{解决：}修正负数逻辑；将超大常量替换为 (0xFF << 23)。
\end{mybluebox}

![部分测试失败](4_32_36.png)

\newpage

# 四、 实验结果与总结

经过逐步排查与修复，最终代码通过了所有限制检查，并在本地测试中拿到了满分。

![btest测试满分](5_btest_36.png)

![dlc无报错](6_dlc_success.png)

\begin{mybluebox}
\textbf{总结：}通过本次实验，我对计算机底层的位运算、补码及 IEEE 754 浮点数格式有了更直观的理解。实验中多次遇到的 dlc 报错，让我了解了 C89 标准规范及老旧编译器的限制，并提升了通过阅读日志解决环境问题的能力。
\end{mybluebox}
