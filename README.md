本文主要介绍LATEX的相关术语以及在文件编译过程中发生了什么。

---

新人在刚接触和使用LATEX时可能会有以下一些概念的困扰：

> * 什么是TEX/LATEX，它们之间有什么关系？
> * pdfTeX、LuaTeX、XeTeX这些是什么？
> * pdflatex、lualatex、xelatex这些是什么？
> * CTEX套装、TeX Live、MacTeX、MiKTeX这些是什么？
> * tex文件所在目录里面一大堆不同后缀名的文件都是什么东西？
> * 如何通过一堆代码就能生成优雅的pdf文件，底层究竟发生了什么？

了解LATEX在编译过程中底层发生了哪些事情对于我们遇到问题时快速定位原因是大有帮助的，这增强了我们使用LATEX的底气，使我们能够心定神闲地编写tex文件，遇到问题时心中不慌。同时，这也使我们更深入地理解LATEX。

本文主要介绍LATEX的相关术语以及在文件编译过程中发生了什么。

---

## TEX与LATEX

### TEX

TeX 的核心程序（即“TeX 引擎”）最初由高德纳（Donald Knuth）在 1978-1982 年间用**Pascal 语言**编写。TeX 引擎的执行逻辑是通过 Pascal 代码描述，再经 Pascal 编译器转换为汇编语言，最终汇编为机器码运行。其核心功能（如字符处理、排版算法、文件 IO）都对应底层汇编级的内存操作、分支跳转和系统调用。

当你启动一个纯粹的、未经任何初始化的 TeX 引擎时，它只认识大约 300 个原始指令，比如 `\def`, `\hbox`, `\vskip`, `\advance`等。在此时，这些原始指令本身不是宏。它们是引擎的内置功能，是原子操作，无法被展开或分解。可以把它们想象成 CPU 的硬件指令。

纯粹的 TeX 太难用了。因此，每次你运行 tex 或 latex 命令时，引擎做的第一件事不是读你的 .tex 文件，而是先加载一个“格式文件”。

这个格式文件是一个宏定义的集合，它是由 TeX 原始指令预先编写好的、并被引擎预编译成了一种高效加载的二进制形式。

加载格式文件的过程，就像是给一个只有基本指令的计算机安装了一个操作系统和标准库。

高德纳本人还编写了一个简单的 plain TEX 格式，没有定义诸如 `\documentclass` 和 `\section` 等等命令。

### LATEX

LATEX 也是一种格式，建立在 Plain TeX 之上的、一个更庞大、更结构化、更易用的宏包集合。它定义了像 `\documentclass`, `\begin{document}`, `\section` 这样的高级命令。

所有这些 LaTeX 命令，最终都会被一步步展开，转换成 Plain TeX 的宏，然后再展开成 TeX 的原始指令，最后由引擎执行。

### TeX、Plain TeX、LaTeX 的关系

三者本质是**不同抽象层次的排版工具**，从底层技术看，三者的关系类似“汇编 → C 标准库 → 高级语言”，宏的展开过程类似编译中的预处理和代码生成，最终由 TeX 引擎（二进制程序）执行底层操作：

1. **TeX**：最底层的“排版引擎”
   它是一个**编程语言解释器**，自带一套极简的排版原语（如字符输出、行距控制、页面分割等）和语法规则（变量、条件判断、循环、宏定义等）。但直接用 TeX 原语写文档非常繁琐（类似用汇编语言写程序）。
2. **Plain TeX**：TeX 的“标准宏包”
   为简化使用，高德纳在 TeX 基础上定义了一套**预定义宏（macro）**，封装了常用功能（如段落格式、标题、列表等），形成了“Plain TeX 格式”。
   它相当于给 TeX 内核加了一层“标准库”，类似 C 语言的标准库（`stdio.h` 等）对汇编的封装，让用户无需重复编写基础功能。
3. **LaTeX**：基于 TeX 的“高级文档排版系统”
   LaTeX 由 Leslie Lamport 设计，是更高层的“应用框架”，是在 Plain TeX 之上进一步封装的**宏集合**，提供了更高层次的语义（如 `\section`、`\begin{document}` 等），专注于“文档结构”而非底层排版细节。
   它的定位类似高级编程语言，而 TeX 内核相当于它的“解释器/虚拟机”，Plain TeX 则是其依赖的底层库之一。

---

## 格式

对于TeX系统，其在编译.tex源文件前，会预加载一个格式文件，其中包含各种提前定义好的宏，以被用户在源文件中调用。

**格式文件（.fmt）** 是预编译的宏集合与状态信息的二进制文件，用于加速 TeX 引擎的启动和执行。它们本质是将常用格式（宏）（如 Plain TeX、LaTeX 等的核心定义）预先解析、展开并存储，避免每次运行时重复处理，类似“预编译的标准库”。

常见的格式文件如下：

### 基础格式文件

* **plain.fmt**
  对应 Plain TeX 格式的格式文件，包含高德纳定义的基础宏集合（如段落、标题、列表等基础排版功能）。
* **latex.fmt**
  对应 **LaTeX** 格式的基础格式文件，是由Leslie Lamport设计的格式，属于Plain TeX的套娃，实现了很多强大的宏。包含 LaTeX 核心宏（如 `\documentclass`、`\section`、文档环境等）。

### 扩展格式文件

* **pdflatex.fmt**
  对应 **PDFLaTeX** 格式的格式文件，是 LaTeX 格式的变体，直接生成 PDF 而非 DVI（需配合 pdfTeX 引擎），格式中包含 PDF 相关的宏定义（如图片嵌入、字体映射等）。
* **xelatex.fmt**
  对应 **XeLaTeX** 格式的格式文件，基于 XeTeX 引擎，支持 Unicode 和系统原生字体，格式中包含 Unicode 处理、OpenType 字体支持等宏。
* **lualatex.fmt**
  对应 **LuaLaTeX** 格式的格式文件，基于 LuaTeX 引擎，集成 Lua 脚本功能，格式中包含 Lua 交互、高级字体处理等宏。
* **amstex.fmt**
  对应 **AMS-TeX** 格式的格式文件，专注于数学公式排版，提供更丰富的数学宏（如复杂方程、定理环境等）。

---

## 引擎

**DFTeX、LuaTeX、XeTeX是由TeX衍生的排版引擎**，是用于编译源代码并生成文档的程序，有时也称为**编译器**。

高纳德将TeX的排版引擎设计得如此开放且易扩展，以至于出现了一些由全球社区在此基础上编写的新排版引擎，它们虽然拓展了若干高级特性，但仍严格兼容TeX引擎本身的严谨性。

### pdfTeX

pdfTeX 是 TeX 引擎的一个重要扩展版本。您可以把它理解为 TeX 程序的一个“升级版”，它最革命性的功能是能够直接输出 PDF 文件，而不仅仅是传统的 DVI 文件。

### LuaTeX

LuaTeX于pdfTeX的基础上开发而来，主要特性是内置Lua脚本引擎，理论上能利用Lua获得更灵活的扩展性，但其流行性及性能均不如XeTeX。

### XeTeX

由Jonathan Kew开发，在TeX基础上增加了对unicode的支持，同时增加若干高级字体渲染技术、高级数学排版功能，其预载的为Plain TeX格式。XeTeX生成的目标文件为.xdv(extend DVI)，其可由dvipdf或其他工具转换为PDF文件。

---

## 编译命令

**编译命令** 是实际调用的、结合了引擎和格式的命令（可执行程序）。如 xelatex 命令是结合 XeTeX引擎和 XeLaTeX 格式的一个编译命令（类似于选择编译器（XeTeX引擎）和链接库函数（选择XeLaTeX 格式）的过程）。

常见的引擎、格式和编译命令的关系总结于下表。
其中[xxx]LATEX 格式 表示与对应命令相匹配的格式，比如 latex 命令对应LaTeX 格式，pdflatex 命令对应PDFLaTeX格式。

|  | 文档格式 | plain TEX 格式 | [xxx]LATEX 格式 |
| --- | --- | --- | --- |
| TeX 引擎 | DVI | tex | N/A |
| pdfTeX 引擎 | DVI | etex | latex |
|  | PDF | pdftex | pdflatex |
| XeTeX 引擎 | PDF | xetex | xelatex |
| LuaTeX 引擎 | PDF | luatex | lualatex |

在此介绍一下几个编译命令的基本特点：

| 编译命令 | 解释 |
| --- | --- |
| latex | 虽然名为 latex 命令，底层调用的引擎其实是 pdfTeX。 该命令生成 dvi（Device Independent）格式的文档, 用 dvipdfmx 命令可以将其转为 pdf。 |
| pdflatex | 底层调用的引擎也是 pdfTeX，可以直接生成 pdf 格式的文档。 |
| xelatex | 底层调用的引擎是 XeTeX，支持 UTF-8 编码和对 TrueType/OpenType 字体的调用。 当前较为方便的**中文排版**解决方案基于 xelatex。 |
| lualatex | 底层调用的引擎是 LuaTeX。这个引擎在pdfTeX 引擎基础上发展而来，除了支持 UTF-8 编码和对 TrueType/OpenType 字体的调用外，还支持通过 Lua 语言扩展 TEX 的功能。 lualatex 编译命令下的中文排版支持需要借助 `luatexja`宏包。 |

---

## LaTeX 发行版

LaTeX 发行版（LaTeX Distribution）是一套**预打包的 TeX/LaTeX 系统集合**，包含了编译文档所需的所有核心组件（引擎、宏包、字体、工具等），目的是让用户无需手动零散安装各种组件就能直接使用 LaTeX（类似于Linux的发行版）。

简单说，它类似 “软件套件”—— 就像 “Office 套件” 包含 Word、Excel 等工具，LaTeX 发行版包含了排版所需的 “引擎、宏包、字体、编译工具” 等一整套工具链。

### 发行版的核心组成

一个完整的 LaTeX 发行版通常包含：

1. **TeX 引擎**：如 pdfTeX、XeTeX、LuaTeX 等，负责解析代码并生成输出文件（PDF 或 DVI）；
2. **基础宏包与文档类**：如 LaTeX 核心宏包（`latex.ltx`）、标准文档类（`article.cls`、`book.cls`）、常用扩展宏包（`amsmath`、`graphicx` 等）；
3. **字体文件**：包括 TeX 原生字体（如 Computer Modern 系列）和现代字体（OpenType/TrueType 等，供 XeLaTeX/LuaLaTeX 使用）；
4. **辅助工具**：如文献管理工具（BibTeX、Biber）、索引生成工具（MakeIndex）、格式文件生成工具（iniTeX）等；
5. **配置文件与搜索路径**：定义宏包、字体的存储位置，确保引擎能正确找到所需文件。

宏包就是别人通过编写宏集造的轮子，直接拿来用就可以了。类似C的标准库或者第三方库。

### 主流的 LaTeX 发行版

不同发行版针对不同操作系统优化，核心功能一致，但安装和维护方式略有差异：

1. **TeX Live**
   * 最主流、跨平台（Windows、macOS、Linux）的发行版，由国际 TeX 用户组（TUG）维护；
   * 每年更新一次，包含几乎所有常用宏包和工具，兼容性极强；
   * 适合多数用户，尤其是需要跨平台一致性的场景（如团队协作）。
2. **MiKTeX**
   * 主要面向 Windows 系统（也支持 macOS/Linux），特点是 “按需安装”—— 初始安装体积小，使用时自动下载缺失的宏包；
   * 适合初学者或对磁盘空间敏感的用户，但跨平台兼容性略逊于 TeX Live。
3. **MacTeX**
   * 基于 TeX Live 的 macOS 专用发行版，预装了针对 macOS 优化的组件（如 PDF 预览工具 Skim、字体管理器等）；
   * 是 macOS 用户的首选，无需手动配置系统适配。
4. **CTeX 套装**
   * 针对中文用户的 Windows 发行版，集成了中文支持宏包（如 CJK）和字体；
   * 逐渐被 TeX Live + 现代中文宏包（如 `ctex`）替代。

### 为什么需要发行版？

LaTeX 系统的组件极其庞大（宏包数千个，字体和工具繁多），手动收集、安装和维护这些组件会非常繁琐，且容易出现版本冲突（如宏包依赖不兼容）。发行版通过预打包和统一管理，解决了这些问题：

* 确保所有组件版本匹配，减少 “编译报错”；
* 提供统一的更新机制（如 TeX Live 的 `tlmgr` 工具）；
* 内置中文、日文等多语言支持（现代发行版中已默认集成）。

LaTeX 发行版是 “开箱即用” 的 TeX/LaTeX 工具集合，包含了编译文档所需的引擎、宏包、字体和工具。主流选择是跨平台的 **TeX Live**（适合多数用户）和 macOS 专用的 **MacTeX**，Windows 用户也可考虑 **MiKTeX**。安装发行版后，即可通过 `pdflatex`、`xelatex` 等命令编译 LaTeX 文档。

---

## 编辑器

所谓编辑器就是可以编辑和书写latex源码的程序软件，比如Notepad（记事本）、NotePad3、Vim等。在这些简单的编辑器中写好代码保存后，需要到命令行中输入编译命令进行编译（熟练之后可以编写批处理文件和Makefile文件到命令行编译）。

> “四十岁后，不滞于物，草木竹石均可为剑。自此精修，渐进于无剑胜有剑之境。”——《神雕侠侣》独孤求败

为了简化书写和编译的复杂度，一些集成开发环境（IDE，Integrated Development Environment）被开发出用于帮助用户提高效率。 在 LaTeX 领域，常见的 IDE 有 VS Code、TeXstudio、WinEdt、Texworks、TeXShop 等，它们集成了 LaTeX 代码编辑、语法高亮、一键编译、PDF 预览等功能，方便用户编写和排版文档。

> IDE是一种集成了代码编辑、编译、调试、项目管理等多种功能的软件工具，旨在为开发者提供一个统一的工作环境，提高开发效率。

较为常用的是VS Code和TeXstudio，这两个都支持跨操作系统，WinEdt主要搭配CTeX套装在Windows环境下使用。

个人建议WinEdt只搭配CTEX使用，原因有三个，其一，CTEX套装默认集成了WinEdt编辑器。其二，WinEdt为商用软件，需要付费，虽然免费版也能使用全部功能。其三，软件闭源，更新缓慢。

VS Code和Texstudio看个人习惯，没使用过VS Code的推荐使用Texstudio，新手推荐使用Texstudio，原因是它职责单一，只用来编写tex文件，并且个人感觉Debug比VS Code好用。并且Texstudio是用QT框架编写的开源软件，如果有功能建议可以去其Github[主页](https://github.com)提issues。

## LATEX 用到的文件一览

除了源代码文件 .tex 以外，使用 LATEX 时还可能接触到各种格式的文件。本节简单介绍一下经常见到的文件。

每个宏包和文档类都是带特定扩展名的文件，除此之外也有一些文件出现于 LATEX 模板中：

| 文件扩展名 | 作用 |
| --- | --- |
| .sty | 宏包文件。宏包的名称与文件名一致。 |
| .cls | 文档类文件。文档类名称与文件名一致。 |
| .bib | 参考文献数据库文件。 |
| .bst | 用到的参考文献格式模板。 |

在编译过程中可能会生成相当多的辅助文件和日志。一些功能如交叉引用、参考文献、目录、索引等，需要先通过编译生成辅助文件，然后再次编译时读入辅助文件得到正确的结果，所以复杂的源代码可能要编译多次。

| 中间文件 | 作用 |
| --- | --- |
| .log | 排版引擎生成的日志文件，供排查错误使用。 |
| .aux | 生成的主辅助文件，记录交叉引用、目录、参考文献的引用等。 |
| .toc | 生成的目录记录文件。 |
| .lof | 生成的图片目录记录文件。 |
| .lot | 生成的表格目录记录文件。 |
| .bbl | BibTeX生成的参考文献记录文件。 |
| .blg | BibTeX生成的日志文件。 |
| .idx | 生成的供 `makeindex` 处理的索引记录文件。 |
| .ind | `makeindex` 处理 .idx 生成的用于排版的格式化索引文件。 |
| .ilg | `makeindex` 生成的日志文件。 |
| .out | `hyperref` 宏包生成的 PDF 书签记录文件。 |

---

## 编译过程发生了什么

以`xelatex`编译命令为例（其他编译命令类似），结合一个包含参考文献、图表的最简示例，详细描述编译流程，并说明中间文件的作用。

### 最简 LaTeX 示例代码

先定义一个包含文档结构、图表、参考文献的示例文件 `main.tex`：

```
\documentclass{article}
\usepackage{graphicx}  % 插入图片
\usepackage{caption}   % 图表标题
\usepackage{biblatex}  % 参考文献管理
\addbibresource{refs.bib}  % 关联参考文献库

\begin{document}
\section{引言}
这是一个示例文档，包含图\ref{fig:example}和参考文献\cite{knuth1984tex}。

\begin{figure}[h]
  \centering
  \includegraphics[width=0.5\textwidth]{example.png}  % 插入图片
  \caption{示例图片}
  \label{fig:example}
\end{figure}

\printbibliography  % 输出参考文献列表
\end{document}
```

配套文件：

* `refs.bib`（参考文献库，包含一条条目）；
* `example.png`（图片文件）。

### XeLaTeX 编译全流程（分阶段解析）

XeLaTeX 编译本质是 **XeTeX 引擎加载 `xelatex.fmt` 格式文件，解析 `.tex` 源文件，经宏展开、排版计算、调用外部工具（如 Biber），最终生成 PDF** 的过程。核心步骤如下：

#### 阶段 1：初始化与格式文件加载（`xelatex main.tex` 启动时）

1. **XeTeX 引擎启动**

   XeLaTeX 是 XeTeX 引擎的 “前端命令”，执行 `xelatex main.tex` 时，实际启动的是 XeTeX 二进制程序（底层为汇编指令实现的机器码，如内存分配、文件 IO 等系统调用）。
2. **加载 `xelatex.fmt` 格式文件**

   * `xelatex.fmt` 是预编译的二进制格式文件（类似 “预编译的宏库”），包含 LaTeX 核心宏定义（如 `\documentclass`、`\section`）、XeTeX 特有的 Unicode 和字体处理宏（如 `fontspec` 基础定义）。
   * 加载目的：避免每次编译重新解析 LaTeX 核心宏，加速启动（类似 C 程序加载预编译的标准库 `.so`/`.dll`）。

#### 阶段 2：解析 `.tex` 源文件，宏展开与结构分析

XeTeX 引擎逐行读取 `main.tex`，对指令进行**宏展开**（文本替换）和**语义分析**（识别文档结构、图表、引用等）。

1. **预处理与宏展开**
   * 遇到 `\documentclass{article}`：展开为 `article.cls` 文档类的宏定义（如页面大小、字体默认设置等），本质是一系列底层 TeX 原语（如 `\textwidth=345pt` 等长度设置）。
   * 遇到 `\usepackage{graphicx}`：加载 `graphicx.sty` 宏包，展开为图片处理的宏（如 `\includegraphics` 对应处理图片路径、缩放的底层指令）。
   * 遇到 `\begin{document}`：标志文档内容开始，触发页面初始化（如页眉页脚、页边距设置）。
2. **处理交叉引用与标签**
   * 遇到 `\label{fig:example}`：将标签 `fig:example` 与当前图号（如 “1”）关联，写入 **`.aux` 辅助文件**（文本格式），供后续编译解析引用（如 `\ref{fig:example}` 需要读取 `.aux` 中的图号）。
   * 此时 `\ref{fig:example}` 暂时无法确定具体数值（因标签定义和引用可能跨页），会先记录为占位符（如 `??`）。
3. **处理图片**
   * `\includegraphics{example.png}`：调用 XeTeX 内置的图片处理模块（底层为图像处理库的汇编指令，如解析 PNG 格式、计算像素与 TeX 单位的转换），记录图片在 PDF 中的位置和尺寸，但不直接嵌入（需后续步骤生成时写入）。

#### 阶段 3：第一次编译生成中间文件（未完成引用和参考文献）

XeTeX 引擎完成源文件解析后，进行排版计算（如行间距、分页），生成 **`.xdv` 中间文件** 和其他辅助文件：

1. **`.xdv` 文件**
   * 全称 “eXtended Device Independent”，是 XeTeX 特有的中间格式，包含排版后的文本、字体、图形位置信息（但未包含实际图片和完整字体数据），类似 “排版指令清单”。
2. **其他辅助文件**
   * `.aux`：记录交叉引用（标签与编号的映射，如 `\newlabel{fig:example}{{1}{1}}` 表示图 1 在第 1 页）、参考文献引用信息（如 `\abx@aux@cite{knuth1984tex}`）。
   * `.log`：编译日志，包含宏展开过程、错误信息（如宏未定义、图片缺失）、加载的宏包和字体列表（用于调试）。
   * `.out`：部分图表位置信息（如浮动体位置计算结果）。

#### 阶段 4：调用参考文献工具（Biber）处理引用

由于 LaTeX 无法直接解析 `.bib` 文件，需通过外部工具生成可识别的参考文献列表：

1. **执行 `biber main`**
   * Biber 读取 `.aux` 中记录的引用条目（如 `knuth1984tex`），解析 `refs.bib` 中的 BibTeX 格式数据（如 `@book{knuth1984tex, ...}`），生成 **`.bbl` 文件**（LaTeX 可识别的参考文献列表宏代码）。
   * 例如，`refs.bib` 中的条目会被转换为 `\bibitem` 或 `biblatex` 专用的宏定义，包含作者、标题、出版信息等。

#### 阶段 5：第二次 XeLaTeX 编译（解决引用和参考文献）

再次执行 `xelatex main.tex`，目的是读取第一次编译生成的 `.aux`（交叉引用）和 `.bbl`（参考文献），填充占位符：

1. **解析 `.aux` 中的交叉引用**
   * `\ref{fig:example}` 读取 `.aux` 中的 `\newlabel` 指令，替换为实际编号 “1”。
2. **插入参考文献列表**
   * `\printbibliography` 展开为 `.bbl` 中的宏代码，将参考文献条目排版到文档末尾。
3. **更新 `.aux` 和生成最终 `.xdv`**
   * 此时交叉引用和参考文献已确定，`.aux` 会被更新（确保无遗漏），生成包含完整内容的 `.xdv` 文件。

#### 阶段 6：转换 `.xdv` 为 PDF（最终输出）

XeTeX 引擎调用内置的 PDF 生成模块，将 `.xdv` 转换为 PDF：

1. **嵌入字体**
   * XeTeX 基于 `xelatex.fmt` 中的字体配置，调用系统字体库（如计算机中的 `Times New Roman` 或中文字体），将文档中使用的字体轮廓数据（TrueType/OpenType）嵌入 PDF（避免字体缺失导致乱码）。
2. **嵌入图片**
   * 读取 `example.png` 的二进制数据，按 `.xdv` 中记录的位置和尺寸嵌入 PDF，底层通过图像压缩算法（如 PNG 解码）处理像素数据。
3. **生成 PDF 结构**
   * 构建 PDF 的页面树、目录（若有）、交叉引用表（点击引用跳转）等结构，最终生成 `main.pdf`。

### 为什么需要多轮编译？

核心原因是 **LaTeX 是 “单遍扫描” 引擎**，无法在一次编译中同时确定 “引用” 和 “被引用对象” 的位置 / 编号：

* 第一次编译：识别 “被引用对象”（如图、文献条目），记录到 `.aux`，但无法知道它们最终的编号 / 页码。
* 第二次编译（或调用 BibTeX/Biber 后）：读取 `.aux` 或 `.bbl` 中的记录，反向填充 “引用” 的内容。
* 若内容长度导致页码变化（如参考文献列表增加新页），需额外编译一次同步页码引用。

调用参考文献工具处理引用的参考文献工具通常有两种：BibTeX和Biber。

| 工具 | 编译流程（标准轮次） | 核心中间文件变化 |
| --- | --- | --- |
| BibTeX | xelatex → xelatex → bibtex → xelatex | .aux（记录引用）→ .bbl（BibTeX 生成）→ 最终 PDF |
| Biber | xelatex → biber → xelatex | .aux（记录引用）→ .bbl（Biber 生成）→ 最终 PDF |

*注：Biber 流程通常比 BibTeX 少一轮初始 `xelatex`，因为 `biblatex` 对 `.aux` 的处理更高效。*

**使用 BibTeX 时，标准流程需要 “xelatex 两次 → bibtex → xelatex 一次（或多次）”**，这是因为传统 BibTeX 对 `.aux` 的依赖更严格，需要两次初始编译确保标签信息完整。而现代 Biber 配合 `biblatex` 可简化流程，但本质仍是通过多轮编译解决 “引用 - 被引用” 的依赖关系。

实际使用中，无论哪种工具，**最终目标都是确保交叉引用、页码、参考文献列表完全同步**，因此建议在复杂文档中多编译 1-2 次，避免遗漏。

### 中间文件汇总及作用

| 文件名 | 类型 | 作用 |
| --- | --- | --- |
| `main.aux` | 辅助文件 | 记录交叉引用（标签与编号）、参考文献引用信息，供多轮编译同步数据 |
| `main.log` | 日志文件 | 记录编译过程（宏加载、错误信息、字体使用），用于调试 |
| `main.xdv` | 中间格式 | 包含排版后的文本、图形位置信息，是 XeTeX 特有的 “排版指令集” |
| `main.bbl` | 参考文献 | BibTeX/Biber 生成的 LaTeX 宏代码，包含格式化后的参考文献条目 |
| `main.blg` | BibTeX/Biber 日志 | 记录 BibTeX/Biber 处理参考文献的过程（如条目解析、格式转换） |
| `main.out` | 浮动体信息 | 记录图表等浮动体的位置计算结果，辅助排版优化 |

### 底层技术补充（汇编 / 编译器视角）

* **XeTeX 引擎的本质**：是用 C 语言编写的程序（最终编译为 x86-64/ARM 汇编指令），核心逻辑包括：
  + 词法分析（识别 `\section` 等指令为 “宏” token）；
  + 语法分析（解析宏的嵌套结构，如 `\begin{figure}` 与 `\end{figure}` 的匹配）；
  + 内存管理（分配缓冲区存储宏展开结果、排版数据）。
* **宏展开的底层**：类似 C 预处理器的 `#define` 替换，但更复杂（支持参数、条件判断），由 XeTeX 引擎中的 “宏处理器” 模块通过字符串操作指令（汇编层面的 `mov`、`cmp`）实现。
* **PDF 生成**：最终调用系统的文件写入指令（如 `write` 系统调用，对应汇编的 `sys_write`），将二进制数据（字体、图片、文本）按 PDF 规范组织成 `main.pdf`。

(PS：这样的编译器我们能写出来吗，怎么都是老外写的？)

---

## 参考文章

1. [一份 (不太) 简短的 LaTeX2ε 介绍](https://github.com)
2. [杂谈： Tex 排版系统历史及各引擎版本梳理](https://github.com)
3. [LaTeX引擎、格式、宏包、发行版大梳理](https://github.com)

* [1. Invinc-Z](#tid-S23cdw):[西部世界westwingy加速器](https://www.yicheer.com)
* **本文链接：** [https://github.com/Invinc-Z/p/19176617/latex-intro-terms](https://github.com)
* **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
* **版权声明：** 本博客所有文章除特别声明外，均采用 [BY-NC-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
* **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
