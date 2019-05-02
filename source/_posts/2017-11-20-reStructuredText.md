---
layout: post
title: reStructuredText语法
date: 2017-11-20
tag: reStructuredText
---
### 参考资料

1. [reStructuredText官网-使用手册](http://docutils.sourceforge.net/docs/user/rst/quickref.html)
2. [reStructuredTex官方语法](http://docutils.sourceforge.net/docs/user/rst/quickstart.html)
3. [关于TOC树](http://www.pythondoc.com/sphinx/markup/toctree.html)
4. [博客1](https://www.cnblogs.com/zzqcn/p/5096876.html)
5. [博客2](http://blog.csdn.net/u012150179/article/details/37743605)
6. [博客3](http://www.jianshu.com/p/1885d5570b37)



### 说明

一般语法可以参考上述所有资料，本文记录reStructedText的特殊语法。

### 内部链接

* 文档内链接跳转语法

  1. 在目标前 标识：

     ```
     .. highlight:: rst
     .. _3.1.1 云主机服务:
     ```

     ..后有一个空格，::后有一个空格

  2. 在起始处 标识：

     ```
      :ref:`“3.4.1 报警” <3.4.1 报警>` 
     ```

     前后各有一个空格，<>前有一个空格


### 表格

实际使用中发现，生成的pdf里，表格中的内容如果是以下划线连接的文字、斜杠连接的文字，表格在宽度不够的情况下无法自动分割文字至多行来适应有限的宽度。（也可能行数超过30行，但目前写的文档最后一列可以自动分割，推测是上述原因）

* 写表格使用vim配合vim-table-mode插件。
  * 安装：
    1. 首先安装**[NeoBundle](https://github.com/Shougo/neobundle.vim)** ；
       * 注意：.vimrc文件要自己创建，内容可以复制官方文档中给出的内容。
    2. 调出vim，执行`:NeoBundleInstall` 在官方文档上有说明；
    3. 在**~/.vimrc**文档中添加`NeoBundle 'dhruvasagar/vim-table-mode'` ；
  * 使用：vim界面输入`:TableModeToggle` 开启表格模式，接着分别输入`let g:table_mode_corner_corner='+'` 以及`let g:table_mode_header_fillchar='='` 进入reStructuredText模式。详细使用见[官方文档](https://github.com/dhruvasagar/vim-table-mode) 。
  * 修改：如果是重新打开文档，那么修改前要输入上述三个命令，进入正确的模式。**修改只要修改内容，不用管边框是否对齐，不要手动调整！vim-table-mode插件会自动补齐。**









