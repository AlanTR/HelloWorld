title: Windows平台下git中文乱码的问题
date: 2013-02-13 20:38:13
tags: 
 - Git
 - 版本控制
categories: 笔记
---
进入git安装目录，改一下配置就可以基本解决：
1、etc\gitconfig：

{% blockquote %}
[gui]
     encoding = utf-8
[i18n]
     commitencoding = gbk
[svn]
     pathnameencoding = gbk
{% endblockquote %}

说明：打开 Git 环境中的中文支持。pathnameencoding设置了文件路径的中文支持。
<!--more-->
2、etc\git-completion.bash：

{% blockquote %}
alias ls='ls --show-control-chars --color=auto'
{% endblockquote %}

说明：使得在 Git Bash 中输入 ls 命令，可以正常显示中文文件名。

3、etc\inputrc：

{% blockquote %}
set output-meta on set convert-meta off
{% endblockquote %}

说明：使得在 Git Bash 中可以正常输入中文，比如中文的 commit log。

4、etc\profile：

{% blockquote %}
export LESSCHARSET=iso8859    #或者utf-8、gbk
{% endblockquote %}

说明：$ git log 命令不像其它 vcs 一样，n 条 log 从头滚到底，它会恰当地停在第一页，按 space 键再往后翻页。这是通过将 log 送给 less 处理实现的。以上即是设置 less 的字符编码，使得 $ git log 可以正常显示中文。其实，它的值不一定要设置为 utf-8，比如 latin1 也可以……。还有个办法是 $ git –no-pager log，在选项里禁止分页，则无需设置上面的选项。


转自[jizhengjieing的专栏][1]

[1]: http://blog.csdn.net/jizhengjieing/article/details/6799211