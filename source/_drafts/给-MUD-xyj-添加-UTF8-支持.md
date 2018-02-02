---
title: 给 MUD xyj 添加 UTF8 支持
tags: MUD, LPC, C, iconv, xyj
---
1.问题的提出

好久没有研究mud了，最近又想琢磨琢磨，然后就发现了[fluffos](https://github.com/fluffos/fluffos)这个项目。其中有一个改进，那就是终于支持utf8了。说起来mud这种史前产物大多数都是gb2312的，放到github上网页上竟是乱码，不爽的一笔。utf8这个改进还是值得跟进的。

其实仔细研究MudOS代码的话，会发现其实从MudOS的层次上并不存在是否支持UTF8的问题。因为对于大多数情况，MudOS并没有什么需要解析的地方，所有字符串的内容，MudOS都是不关心的，MudOS只要用char数组整体传递就行了。
