---
layout: post
title: 我的notepad++ 正则表达式
description: notepad\++中常用的替换正则表达式
category: blog
---

**notepad\++是个好工具，替换文本非常方便。**


- - -


在WireShark里拷贝 **Description** 出来是这样的：

```
..0. .... = MS (Multiplexed Slot): False
.... .0.. = EA (Exception Acknowledge): False
.... ...1 = RD (Ready): True
...0 .... = EN (Exception New): False
..00 0... = PR (Priority): lowest (0)
.... .000 = RS (RequestToSend): 0
.... ..0. = ER (Exception Reset): False
```

想保留圆括号里的内容，ctrl+h替换,用如下正则表达式




| 替换目标 | `.+\(([\w\s]+)\).+` |
|--------|--------|
|     **替换为**   |    **`\1`**    |



查找模式选择**正则表达式**,不用选择**++.++匹配新行**，全部替换就OK了。


- - -

还是WireShark里拷贝出来的**Bytes -> Hex Stream** :

`01111e00000200606500490288ab04ff02fd010000004c007703010000000000000000872000100000000000000000000000000000000000f00077030100000000000000000000000000000000000000000000000000000000000000000000000000000091516407534455278ceee10050321d72
`

替换成C语言数组：

| 替换目标 | `([a-f0-9]{2})`或` (.{2})`或其它能用的都可以 |
|--------|--------|
|     **替换为**   |    **`0x\1,`**    |

检查一下，貌似最好还是把最后一个逗号删除。

- - -

markdown编辑器使用的是[haroopad][1]，环境Windows7，感觉比[MarkdownPad][]好用啊，不过貌似Vim模式切换有问题。

用的这个Jekyll模版貌似markdown格式支持有问题，就这样吧。

[1]: http://pad.haroopress.com/ "haropad"
[MarkdownPad]: http://markdownpad.com/
[zhiying678]:    http://blog.houmingjiang.cn  "zhiying678"