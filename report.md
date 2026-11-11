---
title: "计算机系统基础Ⅱ Lab1 - DataLab 实验报告"
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
  - \renewenvironment{Shaded}{\begin{tcolorbox}[colback=gray!10, colframe=gray!60, boxrule=0.5pt, arc=3pt, left=5pt, right=5pt, top=5pt, bottom=5pt]}{\end{tcolorbox}}
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
**题目要求**：仅使用 `~` 和 `&` 实现异或运算，最大操作符数量 14。
**推导过程**：
异或的基本定义是“相同为 0，不同为 1”，用集合的逻辑表示即 `(x & ~y) | (~x & y)`。由于题目禁止使用按位或 `|`，这里需要应用德摩根定律：`A | B = ~(~A & ~(~B))`。
将 `(x & ~y)` 视为 A，`(~x & y)` 视为 B，代入公式得到：`~(~(x & ~y) & ~(~x & y))`。
**代码实现**：
\begin{mybluebox}
\begin{verbatim}
int bitXor(int x, int y) {
  return ~(~(x&~y)&~(~x&y));
}
\end{verbatim}
\end{mybluebox}

### 1.2 tmin
**题目要求**：返回 32 位补码最小值，最大操作符数量 4。
**推导过程**：
32 位有符号整数的最小值为 `-2147483648`，其二进制补码表示为 `0x80000000`，即最高位（符号位）为 1，其余 31 位为 0。
直接将整型常量 1 左移 31 位即可得到该值。
**代码实现**：
\begin{mybluebox}
\begin{verbatim}
int tmin(void) {
  return 1<<31;
}
\end{verbatim}
\end{mybluebox}

### 1.3 isTmax
**题目要求**：判断 x 是否为 32 位有符号最大值，不能使用位移操作，最大操作符数量 10。
**推导过程**：
最大值为 `0x7FFFFFFF`。将其取反得到 `0x80000000`（即最小值），而最大值加 1 也是 `0x80000000`。因此，满足 `~x == x + 1` 的数大概率是最大值。
但存在一个特例：当 `x = -1`（即 `0xFFFFFFFF`）时，`~x` 为 0，`x+1` 也为 0，同样满足上述等式。因此必须将 `-1` 排除。
由于不能使用位移，利用逻辑非 `!` 的特性：对 `-1` 取反得到 0，再次取逻辑非 `!!~x` 得到 0；对最大值取反得到非 0 值，逻辑非后得到 1。
**代码实现**：
\begin{mybluebox}
\begin{verbatim}
int isTmax(int x) {
  return !(~x^(x+1))&!!~x;
}
\end{verbatim}
\end{mybluebox}

### 1.4 allOddBits
**题目要求**：若所有奇数位都为 1 则返回 1，最大操作符数量 12。不能使用大于 255 的常量。
**推导过程**：
32 位整数的奇数位全为 1，其掩码为 `0xAAAAAAAA`。由于禁止使用大常量，需要用小常量 `0xAA` 通过位移和或运算拼接：`(0xAA<<24)|(0xAA<<16)|(0xAA<<8)|0xAA`。
将 x 与该掩码进行与运算，若奇数位全为 1，结果必定等于该掩码。通过异或运算判断两数是否相等，最后用逻辑非返回结果。
**代码实现**：
\begin{mybluebox}
\begin{verbatim}
int allOddBits(int x) {
  int mask = (0xAA<<24)|(0xAA<<16)|(0xAA<<8)|0xAA;
  return !((x&mask)^mask);
}
\end{verbatim}
\end{mybluebox}

### 1.5 negate
**题目要求**：返回 -x，最大操作符数量 5。
**推导过程**：
在补码系统中，求一个数的相反数的方法是按位取反后加一。
**代码实现**：
\begin{mybluebox}
\begin{verbatim}
int negate(int x) {
  return ~x+1;
}
\end{verbatim}
\end{mybluebox}

## 2. 进阶逻辑与比较

### 2.1 isAsciiDigit
**题目要求**：判断 x 是否在 `0x30` 到 `0x39` 之间（即字符 '0'-'9'），最大操作符数量 15。
**推导过程**：
比较大小可通过减法判断符号位。计算 `x - 0x30`（即 `x + (~0x30+1)`），若结果最高位为 1，则 x 小于 0x30。
计算 `0x39 - x`（即 `0x39 + (~x+1)`），若结果最高位为 1，则 x 大于 0x39。
将两个结果相或，右移 31 位提取符号位。若符号位为 0，说明两者皆满足，在范围内。
**代码实现**：
\begin{mybluebox}
\begin{verbatim}
int isAsciiDigit(int x) {
  int lower = x+(~0x30+1);
  int upper = 0x39+(~x+1);
  return !((lower|upper)>>31);
}
\end{verbatim}
\end{mybluebox}

### 2.2 conditional
**题目要求**：实现 `x ? y : z`，最大操作符数量 16。
**推导过程**：
核心是构造一个掩码。若 x 非 0，需构造全 1 掩码；若 x 为 0，需构造全 0 掩码。
利用两次逻辑非将 x 归一化为 0 或 1，取反加一，即 `~(!!x)+1`。当 x 非 0 时，该式为全 1；当 x 为 0 时，该式为 0。
将掩码与 y 相与，其反码与 z 相与，最后合并。
**代码实现**：
\begin{mybluebox}
\begin{verbatim}
int conditional(int x, int y, int z) {
  return ((~(!!x)+1)&y)|(~(~(!!x)+1)&z);
}
\end{verbatim}
\end{mybluebox}

### 2.3 isLessOrEqual
**题目要求**：判断 `x <= y`，最大操作符数量 24。
**推导过程**：
若直接计算 `y - x`，在异号相减时极易发生溢出，导致符号位判断错误。
必须分情况讨论：
1. 提取两者符号位 `sx` 和 `sy`。
2. 若异号（`sx` 为 1 且 `sy` 为 0），必然 `x <= y`。
3. 若同号（`sx ^ sy == 0`），此时不会溢出，可直接判断 `y - x` 的符号位（最高位是否为 0）。
将上述逻辑转换为位运算组合，并使用 `!!` 规范化为 1 或 0 返回。
**代码实现**：
\begin{mybluebox}
\begin{verbatim}
int isLessOrEqual(int x, int y) {
  int sx = x>>31;
  int sy = y>>31;
  int diff = y+~x+1;
  int sd = diff>>31;
  return !!((sx&~sy)|(~(sx^sy)&~sd));
}
\end{verbatim}
\end{mybluebox}

### 2.4 logicalNeg
**题目要求**：实现逻辑非 `!`，最大操作符数量 12。
**推导过程**：
如果 x 为 0，`x | (~x+1)` 的结果仍为 0。如果 x 非 0，它和它的相反数必然有一个是负数（符号位为 1）。
因此，`x | (~x+1)` 的结果符号位可以作为判断依据。将其右移 31 位，若是 0 则结果为 0，非 0 则结果为 `0xFFFFFFFF`（即 -1）。对其加 1 即可得到 1 或 0。
**代码实现**：
\begin{mybluebox}
\begin{verbatim}
int logicalNeg(int x) {
  int y = x|(~x+1);
  return (y>>31)+1;
}
\end{verbatim}
\end{mybluebox}

## 3. 复杂算法与浮点数

### 3.1 howManyBits
**题目要求**：求表示 x 所需的最少补码位数，最大操作符数量 90。
**推导过程**：
若 x 为正数，需要找到最高位的 1；若为负数，需要找到最高位的 0。可以先将负数转化为正数统一处理。
采用二分法思想：判断高 16 位是否存在有效位，若有则至少需要 16 位，将 x 右移 16 位继续判断；再判断高 8 位、4 位、2 位、1 位。最后累加所有跨度，加上符号位。
**代码实现**：
\begin{mybluebox}
\begin{verbatim}
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
\end{verbatim}
\end{mybluebox}

### 3.2 floatScale2
**题目要求**：浮点数乘 2，最大操作符数量 30。
**推导过程**：
提取符号位、8 位阶码和 23 位尾数。
若阶码为 0xFF（无穷大或 NaN），直接返回原值。
若阶码为 0（非规格化数），直接左移尾数一位即可完成乘 2 操作。
若为普通规格化数，阶码加 1 即可。
**代码实现**：
\begin{mybluebox}
\begin{verbatim}
unsigned floatScale2(unsigned uf) {
  unsigned sign = (uf>>31)&1;
  unsigned exp = (uf>>23)&0xFF;
  unsigned frac = uf&0x7FFFFF;
  if(exp==0xFF) { return uf; }
  if(exp==0) { frac = frac<<1; }
  else { exp = exp+1; }
  return (sign<<31)|(exp<<23)|frac;
}
\end{verbatim}
\end{mybluebox}

### 3.3 floatFloat2Int
**题目要求**：浮点数转整数，最大操作符数量 30。
**推导过程**：
提取符号、阶码和尾数。计算真实阶数 `E = exp - 127`。
若 `E < 0`，表示浮点数值绝对值小于 1，转整数为 0。
若 `E > 30`，超出 32 位整数表示范围，返回 `0x80000000`。
若在合法范围内，需恢复隐含的整数位 1（`frac | (1<<23)`），并根据 E 的大小进行左移或右移，最后根据符号位补上负号。
**代码实现**：
\begin{mybluebox}
\begin{verbatim}
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
\end{verbatim}
\end{mybluebox}

### 3.4 floatPower2
**题目要求**：计算 2.0 的 x 次方，最大操作符数量 30。
**推导过程**：
2.0^x 的浮点表示中，尾数全为 0，仅改变阶码。
真实阶数 x 加上偏移量 127 即为阶码。若计算出的阶码大于 254（即 x > 127），返回无穷大；若阶码小于等于 0（即 x < -126），返回 0。
将计算出的阶码直接左移 23 位即可得到最终结果。
**代码实现**：
\begin{mybluebox}
\begin{verbatim}
unsigned floatPower2(int x) {
  int exp;
  if(x>127) { return (0xFF<<23); }
  if(x<-126) { return 0; }
  exp = x+127;
  return exp<<23;
}
\end{verbatim}
\end{mybluebox}

\newpage

# 三、 实验中遇到的问题与解决过程

在本地运行 `./btest` 时，逻辑功能完全正确，但在使用 `./dlc` 语法检查器时，遇到了多次报错。

## 问题 1：变量声明位置导致的编译失败
\begin{mybluebox}
\textbf{现象：} 运行时出现大量 `parse error` 和 `undeclared variable`，共计 19 个报错。
\textbf{原因：} `dlc` 按照 C89 标准进行语法检查，要求所有的局部变量声明必须位于函数体最前端。在编写 `howManyBits` 时，我将变量声明与赋值语句混排了。
\textbf{解决：} 将函数内用到的所有 int 变量统一提取到函数第一行声明，随后再进行赋值和运算。
\end{mybluebox}

![变量声明报错](2_dlc_19errors.png)

## 问题 2：使用了非法操作符
\begin{mybluebox}
\textbf{现象：} `bits.c:165:isTmax: Illegal operator (>>)`
\textbf{原因：} 在 `isTmax` 函数中，我一开始使用了右移操作 `x >> 31` 来判断符号位，但查看题目上方的要求发现，该题不允许使用 `>>`。
\textbf{解决：} 放弃右移，根据 `~x == x+1` 这一特性，结合 `!!~x` 重写了判断逻辑，成功通过限制检查。
\end{mybluebox}

![非法操作符报错](3_illegal_op.png)

## 问题 3：边界值测试失败与大常量解析问题
\begin{mybluebox}
\textbf{现象：} 测试得分为 32/36，`howManyBits` 在输入 `-2147483648` 时返回错误。同时 `dlc` 报 `bits.c:283: parse error`。
\textbf{原因：} 
1. `howManyBits` 在处理负数位移时，因算术右移补 1 导致计算错误。
2. `floatPower2` 的代码中使用了 `0x7F800000` 这样的超大常量，老旧版本的解析器无法正确识别导致崩溃。
\textbf{解决：} 修正了负数处理的逻辑，将负数转化为正数计算后补上符号位；同时将大常量替换为位移操作 `(0xFF << 23)`，绕过了工具限制，最终实现了 0 报错。
\end{mybluebox}

![部分测试失败](4_32_36.png)

\newpage

# 四、 实验结果与总结

经过逐步排查与修复，最终代码通过了所有限制检查，并在本地测试中拿到了满分。

![btest测试满分](5_btest_36.png)

![dlc无报错](6_dlc_success.png)

\begin{mybluebox}
\textbf{总结：}
通过本次实验，我对计算机底层的位运算、补码表示以及 IEEE 754 浮点数格式有了更直观的理解。在不允许使用常规控制流和算术运算符的限制下，只能通过掩码构造、位运算逻辑推导来达成目的，这极大地锻炼了底层思维。

此外，实验中遇到的多次 `dlc` 报错，也让我了解了 C89 标准的变量声明规范以及老旧编译器的解析限制，并学会了如何通过阅读错误提示和查阅资料来解决环境与工具带来的阻碍。整体而言，这次实验不仅巩固了理论知识，也提升了实际排错能力。
\end{mybluebox}
