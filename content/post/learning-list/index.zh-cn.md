---
title: "学习清单[持续更新]"
description: 
date: 2023-08-17T00:19:27+08:00
image: 
math: 
license: 
hidden: false
comments: true
categories: ['Journal']
tags: ['learn', 'read']
draft: false
---

> 通过这篇文章我会持续更新一些我种草的或者已经买了的书籍，以及一些我想学习的东西。我会同步更新一些我学习过程中的笔记，但需要说明的是笔记是每个人知识结构化的中间过程，是每个人查漏补缺的工具，所以我认为笔记并不具备通用性，根据每个人自己先验知识的不同笔记可能会天差地远，因此每个人应该通过自己的学习来总结得到自己的笔记。

### [The Unix Workbench](https://seankross.com/the-unix-workbench/working-with-unix.html)

*学习 Unix-like System 和 Shell Scripting 最好的入门读物，非常基础但非常实用，适合初学者。也可以配套 coursera 的教程一起食用：[The Unix Workbench](https://www.coursera.org/learn/unix/home/week/1)*

- `ls -l` means "long format"
- `apropos editor` to find related tools
- grep defaults to regex expression
	- `(xxx)*` means a group
	- `{2,3}` both sides are closed
- `mv ~/.Trash` on mac and `mv ~/.local/share/Trash` on Ubuntu to delete file/directory
- `cat` means "concatenate" file, but we use it to print file contents
- `sdiff` to compare side by side
- `makefile` command should be indented with `TAB`
- `bc` for Bash benchmark
- `$()` is command substitution ,`$()`
- `$@` array of all arguments, `$#` number of arguments, `$?` for last exit status
- You can combine AND and OR operators in commands, which are evaluated from left to right, no priority should be concerned, e.g. `echo Athos || echo Porthos && echo Aramis` 
- `[[]]` for bool ops
	- `-eq`
	- `-ne`
	  - `-e` for file exists
	  - `-d` for dir exists 
	  - `-z`for zero string  
	  - `-n` for non-zero string
- `=~` for regex match `=` for string equal to `!=` for string not equal to
- `(xx yy)` for bash array
	- `${name[index]}` to get item
	- starts from 0 and `*` for all the items
	- `name[index]=xxx` to set value
	- `${plagues[*]:5:3} ` to get by range(start:count)
	- `${#plagues[*]}`  to get length
	- `dwarfs+=(bashful dopey happy)` to append to list
- `echo a{0..4}`  -> `a0 a1 a2 a3 a4`
	- `echo {1..3}{A..C}` -> `1A 1B 1C 2A 2B 2C 3A 3B 3C`
	- `eval echo {$start..$end}` -> `4 5 6 7 8 9`
	- `echo {{1..3},{a..c}}` -> `1 2 3 a b c`
	- `echo {Who,What,Why,When,How}?` -> `Who? What? Why? When? How?`
- `let` to declare/redeclare a variable
	- `let sum=sum+$element` it does not need a a`$` to refer to the param itself
	- `let` clause is alike  `sum=$((sum + element))` 
- use `local` to prevent parameter overwriting
- chmod 
	- `u` for user, `g` for group, `a` for everyone and `o` for everyone else
	- ops: `+` `-` `=`

