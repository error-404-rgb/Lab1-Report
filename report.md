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
