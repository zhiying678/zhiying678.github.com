---
layout: post
title: win7批处理变量的延迟扩充
description: 批处理 for 中的变量 延迟扩充
category: blog
---

6/21/2014 11:50:20 AM 

```bat
rem requre cmd /v:ON
@echo off
for /f "skip=1 delims=, tokens=5" %%a in (du.csv) do (
rem	echo %%a

	set b=%%a
	set c=!b:~0,3!
	echo !c!
)
rem for skip=1 delims=, 后要有空格
rem !c! 表示 ****延迟环境变量扩充****
rem 参考 
```
*for skip=1 delims=, *中的1和逗号后要有空格
*!c!* 表示**延迟环境变量扩充**
参考[批处理下载全校毕业照](http://yoursunny.com/t/2008/spring-pic/)
> 这里要注意的是!c!这样的写法。批处理中，!变量名!表示“延迟环境变量扩充”。%c%会在执行到for语句的一开始就被扩充(扩充成空字符串)，循环体中对它赋值就不会生效；而!c!则要在执行到这一行时才被扩充(扩充成学号前3位)。
> 默认情况下，“延迟环境变量扩充”功能并没有启用。因此，要执行上述脚本，必须用cmd /v:ON命令启动命令提示符。


github博客也没弄好 先就这样吧，以后再弄

[zhiying678]:   http://zhiying678.github.io  "zhiying678"
