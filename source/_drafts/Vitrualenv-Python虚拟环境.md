---
title: Vitrualenv:Python虚拟环境
date: 2015-02-01 21:39:08
tags:
  - Linux
  - python
---

> virtualenv 有什么用？如果你象我一样热爱 Python ，那么除了基于 Flask 的项目外 还会有其他项目用到 Python 。当项目越来越多时就会面对使用不同版本的 Python 的 问题，或者至少会遇到使用不同版本的 Python 库的问题。摆在你面前的是：库常常不能 向后兼容，更不幸的是任何成熟的应用都不是零依赖。如果两个项目依赖出现冲突， 怎么办？
> 
> Virtualenv 就是救星！它的基本原理是为每个项目安装一套 Python ，多套 Python 并存。但它不是真正地安装多套独立的 Python 拷贝，而是使用了一种巧妙的方法让不同 的项目处于各自独立的环境中。
（来自flask官方文档）

（再次深刻认识到我的渣英语和渣翻译）

在linux下安装：

    $ pip install vitrualenv

<!--more-->在项目文件夹下创建虚拟环境
    $ virtualenv venv
    New python executable in venv/bin/python
    Installing setuptools, pip...done.
venv为虚拟环境所在目录。

开启虚拟环境：
    $ source venv/bin/activate
此时进入了名为venv的虚拟环境。

在虚拟环境下安装python套件
    (venv)$ pip install Flask
    Collecting Flask
        Downloading Flask-0.10.1.tar.gz (544kB)
            100% |################################| 544kB 69kB/s 
    Collecting Werkzeug&gt;=0.7 (from Flask)
        Downloading Werkzeug-0.10.tar.gz (1.1MB)
            100% |################################| 1.1MB 18kB/s 
    Collecting Jinja2&gt;=2.4 (from Flask)
        Downloading Jinja2-2.7.3.tar.gz (378kB)
            100% |################################| 380kB 42kB/s 
    Collecting itsdangerous&gt;=0.21 (from Flask)
        Downloading itsdangerous-0.24.tar.gz (46kB)
            100% |################################| 49kB 30kB/s 
    Collecting markupsafe (from Jinja2&gt;=2.4-&gt;Flask)
        Downloading MarkupSafe-0.23.tar.gz
    Installing collected packages: markupsafe, itsdangerous, Jinja2, Werkzeug, Flask
        Running setup.py install for markupsafe
            building 'markupsafe._speedups' extension
            x86_64-linux-gnu-gcc -pthread -DNDEBUG -g -fwrapv -O2 -Wall -Wstrict-prototypes -fno-strict-aliasing -D_FORTIFY_SOURCE=2 -g -fstack-protector-strong -Wformat -Werror=format-security -fPIC -I/usr/include/python2.7 -c markupsafe/_speedups.c -o build/temp.linux-x86_64-2.7/markupsafe/_speedups.o
            markupsafe/_speedups.c:12:20: fatal error: Python.h: 没有那个文件或目录
             #include &lt;Python.h&gt;
                                                    ^
            compilation terminated.
            ==========================================================================
            WARNING: The C extension could not be compiled, speedups are not enabled.
            Failure information, if any, is above.
            Retrying the build without the C extension now.
            ==========================================================================
            WARNING: The C extension could not be compiled, speedups are not enabled.
            Plain-Python installation succeeded.
            ==========================================================================
        Running setup.py install for itsdangerous
        Running setup.py install for Jinja2
        Running setup.py install for Werkzeug
        Running setup.py install for Flask
    Successfully installed Flask-0.10.1 Jinja2-2.7.3 Werkzeug-0.10 itsdangerous-0.24 markupsafe-0.23
&nbsp;

之后便可在项目文件夹下编辑.py文件，并用python命令运行。

退出虚拟环境
    $ deactivate
&nbsp;