---
title: 小白配置vim
date: 2015-01-15 05:47:33
tags:
  - Linux
  - vim
---

现在折腾vim用力太猛，于是把vim配置统统同步到了github上面

https://github.com/NameIsUseless/vimconf

初步集成了各种tag各种window，长得越来越像一个ide了。

==========

虽然作为小白，但是仍然要有一颗成为大神的心！

于是抱着这个信念，一个懵懂纯洁的小白走上了VIM的不归路。。。

以前都是用的sublime-text这个所谓“世界上最性感的编辑器”，对强大粗犷的vim一直敬而远之。不过最近突然醍醐灌顶，于是便打开笔记本在ubuntu的终端里输入了一段历史性的命令：

    sudo apt-get install vim vim-core vim-scripts vim-gnome

<!--more-->
然后……就……没有了……

好吧，第一次打开vim这等神器，第一感觉便是：卧槽这么丑！

vim的默认界面很好的保留了三十年前的UNIX风格，白字，黑底（或者白条黑字），字体与行距间距均是那令人感动的shell风格。。

无图无真相：

[![2015-01-14 19:23:29屏幕截图](http://blogr.hcyue.ml/wp-content/uploads/2015/01/2015-01-14-192329屏幕截图-300x192.png)](http://blogr.hcyue.ml/wp-content/uploads/2015/01/2015-01-14-192329屏幕截图.png)

&nbsp;

（由于我的vim已经被动过刀子，所以这个是ssh到vps上截的图）

如此简洁的界面简直让人泪流。我不禁想到了IT界的前辈们，他们在纯命令行年代，一日复一日面对着这样的黑底白字，敲出一行行神奇的代码，让今天的我们能够沐浴在阳光下幸福的用着GUI……

好吧，既然如此，那就动刀子动刀子动刀子！改改改！

研究了vim的文档以及参考了网上前辈们的代码之后，我的第一份vim配置文件出炉了……

```vim
""""""""""""""""""""""""""""""""""""""
""""""""""""""""""""""""""""""""""""""
" common settings
""""""""""""""""""""""""""""""""""""""
set fenc=utf-8
set fencs=utf-8,usc-bom,gb18030,gbk,gb2312,cp936

" line of history file
set history=1000
" put confirm while read only
set confirm
" share clipboard with windows
set clipboard+=unnamed

" distinguish the type of file
filetype on

" filetype plugin
filetype plugin on

" save global variables
set viminfo+=!

set iskeyword+=_,$,@,%,#,-

" enable the syntax
syntax enable
syntax on
" set the color theme
colorscheme monokai 
set nu!
set guifont=monofur 12
set guioptions-=M
set guioptions-=T
"""""""""""""""""""""""""""""""""""""""
" file settings
"""""""""""""""""""""""""""""""""""""""
set linespace=0
set wildmenu

set ruler
set rulerformat=%20(%2*%&lt;%f%= %m%r %3l %c %p%%%)

set cmdheight=2

set backspace=2

set whichwrap+=&lt;,&gt;,h,l

set mouse=a
set selection=exclusive
set selectmode=mouse,key

set report=0

set noerrorbells

set fillchars=vert: ,stl: ,stlnc:

"""""""""""""""""""""""""""""""""""""""""""""
" search settings
"""""""""""""""""""""""""""""""""""""""""""""

set showmatch

set ignorecase
set incsearch

set listchars=tab:&gt;-,trail:-

set scrolloff=3
set novisualbell

set statusline=%F%m%r%h%w FORMAT:%{&amp;ff}%=TYPE:%Y  %lL,%vC  [%p%%]  LEN:%LL

set laststatus=2
""""""""""""""""""""""""""""""""""""""""""""""
"  formates
""""""""""""""""""""""""""""""""""""""""""""""

set formatoptions=tcrqn

set autoindent

set smartindent

set cindent
set tabstop=4

set softtabstop=4
set shiftwidth=4

set expandtab

set nowrap

"""""""""""""""""""""""""""""""""""""""""""""
" settings about CTags
"""""""""""""""""""""""""""""""""""""""""""""
let Tlist_Sort_Type = "name"
let Tlist_Use_Right_Window = 1
let Tlist_Compart_Format = 1
let Tlist_Exist_OnlyWindow = 1
let Tlist_File_Fold_Auto_Class = 0

"""""""""""""""""""""""""""""""""""""""""""""
" C complite
"""""""""""""""""""""""""""""""""""""""""""""
map  :call CompileRunGcc()
func! CompliRunGcc()
exec "w"
exec "!gcc % -o %&lt;"
exec "! ./%&lt;"
endfunc

""""""""""""""""""""""""""""""""""""""""""""
" C++
""""""""""""""""""""""""""""""""""""""""""""
map  :call ComplieRunGpp()
func! ComplieRunGpp()
exec "w"
exec "!g++ % -o %&lt;"
exec "! ./%&lt;"
endfunc

"       end
```


动过刀子后的vim界面：

[![2015-01-14 21:33:59屏幕截图](http://blogr.hcyue.ml/wp-content/uploads/2015/01/2015-01-14-213359屏幕截图-300x255.png)](http://blogr.hcyue.ml/wp-content/uploads/2015/01/2015-01-14-213359屏幕截图.png)

虽然说还是不怎么好看= =毕竟刚开始用，慢慢来

这个设置，主要是参照网上的方案，弄了个底部状态栏（显示文件类型 光标位置 文件长度），屏蔽了Gvim那个充满残念的菜单栏，弄出了行号显示，改了配色，改了字体，设置了缩进方式，BLABLABLA。。。

还有F5直接调试C程序。学生党必备嗯= =

monokai配色方案是我在sublimetext中就一直在使用的方案，看着很舒服。

字体是monofur（这俩货名字里怎么都有个mono），很独特的字体，单个字看起来比较奇怪但是整体看起来很赏心悦目啊～

另外，我第二喜欢的字体就是inconsolata，由 Raph Levien 设计，免费的开源的噢

初步就这样吧。vim的扩展性是无穷的，以后把它再整整弄成个ide吧。

另外，为了熟悉vim操作，这么长的配置文件劳资是纯手打0 0已经感受到了命令的强大，等熟练了目测效率会提升几个数量级

果然是编辑器之神。