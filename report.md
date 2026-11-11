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

在 Linux 虚拟机中，首先使用脚本完成了 `datalab` 目录的初始化：

```bash
mkdir datalab && cd datalab
curl -sSLf 'https://chrjeb.cn/files/scripts/init-lab1.sh' | bash
```

![环境初始化](1_init.png)

*(图 1：实验环境初始化)*

\begin{mybluebox}
\textbf{开发工具的更换：}
在实验初期，我直接在虚拟机的终端里使用 nano 和 sed 命令修改 bits.c。但因为我的笔记本键盘左侧 Ctrl 键损坏，导致终端快捷键操作极度不便，而且稍有不慎就会覆盖掉原文件中的大量注释。

为了解决这个问题，我查阅了讲义中关于 VSCode 的提示，通过配置 VirtualBox 的 NAT 端口转发，在 Windows 宿主机上使用 VSCode 的 SSH 插件连接到了虚拟机。此后，所有的代码编写和修改都在 VSCode 中完成，大幅提高了效率。
\end{mybluebox}

\newpage

# 二、 题目实现与逻辑分析

本次实验共包含 13 个函数，以下选取部分具有代表性的题目进行逻辑说明。

## 1. 基础位运算
*   \textbf{bitXor}：题目要求不使用 `|`。我使用了德摩根定律进行转换，公式为 `x ^ y = ~(~(x & ~y) & ~(~x & y))`。
*   \textbf{isTmax}：判断最大值。最大值的二进制特征满足 `~x == x+1`，但 `-1` 也符合此规律。由于题目限制了操作符，不能通过右移判断符号位，因此我使用了 `!!~x` 来构造判断条件，成功排除了 `-1`。

## 2. 进阶位运算
*   \textbf{conditional}：实现三目运算符 `x ? y : z`。核心是构造掩码，当 `x` 不为 0 时，`~(!!x)+1` 会生成全 1 的掩码；当 `x` 为 0 时，生成全 0 掩码。利用该掩码与 `y` 和 `z` 分别进行与运算再合并。
*   \textbf{isLessOrEqual}：判断 `x <= y`。直接相减可能会因为溢出导致符号位判断错误。因此先提取两者的符号位，若异号，则 `x` 为负 `y` 为正时直接成立；若同号，则判断 `y-x` 的符号位。

## 3. 复杂算法与浮点数
*   \textbf{howManyBits}：求补码表示所需的最少位数。使用了二分法的思路，通过位运算依次判断高 16 位、8 位、4 位等区间内是否存在 1，最后加上符号位得出总长度。
*   \textbf{floatFloat2Int}：浮点数转整数。需要提取 IEEE 754 的符号、阶码和尾数，并根据阶码（减去偏差值 127 后）的大小，对尾数进行相应的左移或右移。对于越界情况需返回特定值。

\newpage

# 三、 实验中遇到的问题与解决过程

在本地运行 `./btest` 时，逻辑功能完全正确，但在使用 `./dlc` 语法检查器时，遇到了多次报错，过程如下。

## 问题 1：变量声明位置导致的编译失败

\begin{mybluebox}
\textbf{现象：} 运行时出现大量 `parse error` 和 `undeclared variable`，共计 19 个报错。

\textbf{原因：} dlc 按照 C89 标准进行语法检查，要求所有的局部变量声明必须位于函数体最前端。在编写 howManyBits 时，我将变量声明与赋值语句混排了。

\textbf{解决：} 将函数内用到的所有 int 变量统一提取到函数第一行声明，随后再进行赋值和运算。
\end{mybluebox}

![变量声明报错](2_dlc_19errors.png)

*(图 2：变量声明位置报错)*

## 问题 2：使用了非法操作符

\begin{mybluebox}
\textbf{现象：} `bits.c:165:isTmax: Illegal operator (>>)`

\textbf{原因：} 在 isTmax 函数中，我一开始使用了右移操作 `x >> 31` 来判断符号位，但查看题目上方的要求发现，该题不允许使用 `>>`。

\textbf{解决：} 放弃右移，根据 `~x == x+1` 这一特性，结合 `!!~x` 重写了判断逻辑，成功通过限制检查。
\end{mybluebox}

![非法操作符报错](3_illegal_op.png)

*(图 3：非法操作符报错)*

## 问题 3：边界值测试失败与大常量解析问题

\begin{mybluebox}
\textbf{现象：} 测试得分为 32/36，howManyBits 在输入 `-2147483648` 时返回错误。同时 dlc 报 `bits.c:283: parse error`。

\textbf{原因：} 
1. howManyBits 在处理负数位移时，因算术右移补 1 导致计算错误。
2. floatPower2 的代码中使用了 `0x7F800000` 这样的超大常量，老旧版本的解析器无法正确识别导致崩溃。

\textbf{解决：} 修正了负数处理的逻辑，将负数转化为正数计算后补上符号位；同时将大常量替换为位移操作 `(0xFF << 23)`，绕过了工具限制，最终实现了 0 报错。
\end{mybluebox}

![部分测试失败](4_32_36.png)

*(图 4：部分测试用例失败)*

\newpage

# 四、 实验结果与总结

经过逐步排查与修复，最终代码通过了所有限制检查，并在本地测试中拿到了满分。

![测试满分](5_btest_36.png)

*(图 5：btest 测试满分)*

![dlc无报错](6_dlc_success.png)

*(图 6：dlc 检查无报错)*

\begin{mybluebox}
\textbf{总结：}
通过本次实验，我对计算机底层的位运算、补码表示以及 IEEE 754 浮点数格式有了更直观的理解。在不允许使用常规控制流和算术运算符的限制下，只能通过掩码构造、位运算逻辑推导来达成目的，这极大地锻炼了底层思维。

此外，实验中遇到的多次 dlc 报错，也让我了解了 C89 标准的变量声明规范以及老旧编译器的解析限制，并学会了如何通过阅读错误提示和查阅资料来解决环境与工具带来的阻碍。整体而言，这次实验不仅巩固了理论知识，也提升了实际排错能力。
\end{mybluebox}
