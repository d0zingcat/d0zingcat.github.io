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

以及下面的书单是 GPT 告诉我的：

1.《计算机程序的构造和解释》(SICP) - 由Harold Abelson、Gerald Jay Sussman和Julie Sussman合著的经典教材，深入浅出地讲解了计算机程序设计的基本原理，是一本极具启发性的书籍。

2.《算法导论》(Introduction to Algorithms) - 由Thomas H. Cormen、Charles E. Leiserson、Ronald L. Rivest和Clifford Stein合著，是一本全面介绍算法设计和分析的权威性教材，广泛被计算机科学的学生和专业人士所采用。

3.《深入理解计算机系统》(Computer Systems: A Programmer's Perspective) - 由Randal E. Bryant和David R. O'Hallaron合著，帮助读者理解计算机底层系统的工作原理和编程技巧。

4.《编程珠玑》(The Pragmatic Programmer) - 由Andrew Hunt和David Thomas合著，涵盖了实用的编程技巧、设计原则和项目管理建议，是一本广受欢迎的实用型计算机书籍。

5.《人工智能：一种现代方法》(Artificial Intelligence: A Modern Approach) - 由Stuart Russell和Peter Norvig合著，是一本广泛引用的人工智能教科书，全面涵盖了AI的各个方面。

6.《计算机网络：自顶向下方法》(Computer Networking: A Top-Down Approach) - 由James F. Kurose和Keith W. Ross合著，以自顶向下的方式解释计算机网络的概念和协议。

7.《Clean Code: A Handbook of Agile Software Craftsmanship》 - 由Robert C. Martin撰写，强调编写清晰、可维护和高质量代码的重要性，是一本在软件开发社区广受推崇的书籍。

8.《代码大全》(Code Complete) - 由Steve McConnell撰写，是一本涵盖软件构建过程中最佳实践的经典著作。

9.《设计模式：可复用面向对象软件的基础》(Design Patterns: Elements of Reusable Object-Oriented Software) - 四位作者Erich Gamma、Richard Helm、Ralph Johnson和John Vlissides一同创作的书籍，介绍了常见的软件设计模式。

10.《黑

客与画家》(Hackers & Painters) - 由Paul Graham撰写，该书探讨了编程、创新和技术的关系，对程序员和创业者都有很大启发。