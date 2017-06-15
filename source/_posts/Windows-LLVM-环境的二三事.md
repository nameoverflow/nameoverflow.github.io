---
title: Windows LLVM 环境的二三事
date: 2017-04-09 01:44:05
tags:
  - LLVM
  - compile
---

讲道理，考研党被这种配环境的事情打扰了咸鱼的生活是很不开心的。

起因是 Haskell 的 LLVM binding —— `llvm-general` 在 Hackage 上只支持到 3.5 版本。当然 repo 里是有后续版本的，但是作者在 issues 里说后续的版本并没有弄好。

既然如此那就装 3.5 吧。然而下载了 LLVM 3.5.2 的源码后，Cmake 很顺利，用 VS 2017 打开也很顺利，然而编译的时候报错

```
这里假装有编译 log （好吧忘记存了）
```

总之就是某个类的方法后有 const 限定符，然后调用的上下文却又不是返回 const 于是不给过。

奇怪的是报错的位置是 VS 提供的标准库文件。这就很尴尬了。我要么改 VS 的标准库要么研究一波 LLVM 源码。

在 WSL 里试了一下，无痛编译。看来是 VS 有什么蛋疼的限制。网上稍微搜了一圈也没发现有类似情况（毕竟 3.5 的年代他们还没官方支持 VS 编译吧）。于是决定上 MinGW 来构建。

使用 CMake 生成 MinGW 的 makefile 并没有问题，然而在 make 的时候提示命令语法错误

```
命令语法不正确。
mingw32-make.exe[2]: *** [tools\lto\CMakeFiles\LTO_exports.dir\build.make:61: tools/lto/LTO.def] Error 1
mingw32-make.exe[2]: *** Deleting file 'tools/lto/LTO.def'
mingw32-make.exe[1]: *** [CMakeFiles\Makefile2:9977: tools/lto/CMakeFiles/LTO_exports.dir/all] Error 2
mingw32-make.exe: *** [Makefile:151: all] Error 2
```

找到了这个 build.make 发现这一行的命令很奇怪

```powershell
cd /d C:\Users\hcyue\code\llvm-3.5.2.build\tools\lto && "C:\Program Files\CMake\bin\cmake.exe" -E echo EXPORTS > LTO.def
cd /d C:\Users\hcyue\code\llvm-3.5.2.build\tools\lto && type C:/Users/hcyue/code/llvm-3.5.2.src/tools/lto/lto.exports >> LTO.def
```

<del>纵横 Windows 这么多年没见过 cd 还可以带个 `/d` 参数的</del> 似乎 powershell 的 cd 并不支持 /d 参数。那把它删了吧。

之后继续报错，link 的时候找不到 symbol。发现是刚刚改的那两行生成的目标文件 `LTO.def` 有问题，不知道为什么没有写入成正常的文本文件，而是二进制文件（VS code 打开提示文件过大或为二进制文件）。直接 `type` 出来是一个奇怪的、字距拉得很开的、看起来像是文本文件的格式。很气。

然而既然 `.def` 反正是文本文件，我大不了手动构建一下 `def` 。于是手动运行命令把 `lto.exports` 里的东西 `type` 出来

```powershell
type C:/Users/hcyue/code/llvm-3.5.2.src/tools/lto/lto.exports
```

得到一大堆符号名

```
lto_get_error_message
lto_get_version
lto_initialize_disassembler
lto_module_create
lto_module_create_from_fd
lto_module_create_from_fd_at_offset
lto_module_create_from_memory

（……一大堆 lto 符号名）

LLVMCreateDisasm
LLVMCreateDisasmCPU
LLVMDisasmDispose
LLVMDisasmInstruction
LLVMSetDisasmOptions
```

新建个 `LTO.def` 把原来的覆盖掉，贴进去。

重新运行 `cmake --build .` ，终于过了。
