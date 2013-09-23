---
layout:     post
title:      设置JDK默认编码
category: blog
description: 在学习Java的过程中，很多时候我们是使用手工方式来开发的，而不是使用IDE，在windows上使用javac口令编码java源文件和使用java运行字节码文件，有时候会出现乱码问题。
---

##为什么要手工编码java

也可以一开始就使用IDE来学习Java，比如[Eclipse][]，但是这样会让你对java的运行原理掌握很模糊，推荐在刚学习一门知识时，尽量不要使用开发环境，而是使用一个文本编辑器

##为什么会乱码

一般乱码的原因是windows默认编码的原因，由于我们的操作系统是中文，系统会自动识别，然后默认采用中文的编码“GBK”，然后在windows上安装的软件一般都是采用此编码。
比如你安装了[notepad++][]，用它来编码Java源文件，在任意文件夹下，建立一个空文本，这个文本就会使用windows的默认编码“GBK”，然后使用NotePad++打开它，观察这里：
![](/images/java-coding/2013-09-23_115411.jpg)
NotePad++新版中默认使用“UTF-8”的编码，而你安装的JDK，由于中文windows的原因，JDK使用了“GBK”编码，这种情况下，编码不统一了，如果编译和运行Java文件，就会出现乱码：

    public class Test{
	    public static void main(String[] args){
		    System.out.println("你好");
	    }
	}
结果为：
![](/images/java-coding/2013-09-23_113846.jpg)
这种乱码的解决也很简单。
###第一种办法（推荐）
也是比较常用的办法，就是修改NotePad++的编码：
![](/images/java-coding/2013-09-23_114039.jpg)
将编码设置为：“ANSI”，修改之后，源文件中的中文会乱码，那么你就要重新写一次中文，如果中文较多，推荐先备份一份，然后将备份中的中文拷贝进来。
###第二种办法
在编译Java源文件时指定编码：
	javac -encoding utf8 Test.java
这样去编译，由于源文件是utf8编码，而JDK是“GBK”编码，现在我们指定JDK使用“UTF8”编码，这样就不会乱码了，但是这样比较麻烦，每次都要指定
###第三种办法
修改系统上JDK的默认编码，很简单，在环境变量中，添加一个变量，名称为：“JAVA_TOOL_OPTIONS”，值为：“-Dfile.encoding=UTF-8”，这样就指定了JDK编码为“UTF8”了。
##总结
不论使用哪种办法，思路都是一样的，就是**统一编码**，要清楚中文windows默认使用“GBK”编码，那么安装在中文windows系统上的软件一般直接使用windows系统的编码“GBK”，而[NotePad++][]和[Sublime Text2][]等文本编辑器默认使用“UTF-8”编码，这是造成编码的主要原因。推荐修改编辑器的默认编码和系统保持一致，Linux系统默认编码是“UTF-8”，一般不存在这种乱码。


[Eclipse]: http://www.eclipse.org/downloads/
[NotePad++]: http://notepad-plus-plus.org/
[Sublime Text2]: http://www.sublimetext.com/2