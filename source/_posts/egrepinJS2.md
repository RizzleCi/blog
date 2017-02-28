---
title: JS中的正则（引擎）
date: 2016-11-20 19:46:46
tags: [javascript]
---

## 使用传统NFA
JS和大部分语言一样使用传统的NFA作为正则引擎。检测方法如下。
```js
"nfa not".match(/nfa|nfa not/) // ['nfa']
```
## 引擎的构造
正则的引擎是由一些零部件组合而成的。在上一篇里我把它们笼统地放在一起讲，下面逐类分析一些。
<!--more-->
### 文字字符
普通的文字字符在正则里可以被直接解析，比如 /cxy/ 可以匹配出字符串里连续的'cxy'。
### 量词
量词提供计数功能：`*`/+/?/{n}/{min,max}
### 字符组
字符组是指[...]，无论它有多长，都是匹配一个字符。
### 锚点
^,$这样表示开始或结束的属于锚点，(?=...)这样的环视也算作锚点。它们只匹配字符串的位置。
### 捕获括号
捕获括号及引用是NFA的特色。

## 两个规则
### 优先选择最左的匹配结果
如果引擎不能在字符串开始找到匹配结果，传动装置推动引擎，从下一个位置开始尝试。
### 标准量词匹配优先
标准量词（?、`*`、+、{min,mix}）总是匹配尽可能多的字符，直到匹配上限。
下面re中 \d+ 可以匹配到全部的字符，但是为了让正则中的 0 得到匹配，\d+ “交出”了一个“0”，匹配到的是"1200"。
```js
var re = /^(\d+)(0)$/
re.exec("12000")  // ["12000", "1200", "0"]
```
今天同学恰好给我看了另外一个类似的例子。下面的代码中 0* **不必须** 得到匹配，\d+ 并没有交出字符，匹配了“12000”，而 0* 没有匹配任何字符。
```js
var re = /^(\d+)(0*)$/
re.exec("12000")  // ["12000", "12000", ""]
```

## NFA的特色
### 表达式主导
NFA被称为表达式主导的引擎。
引擎从正则表达式的第一个字符开始，每次检查一部分，同时检查当前文本是否匹配表达式的当前部分，如果是则继续表达式的下一部分，如果不是则推动引擎重复尝试。如此继续直到表达式的所有部分都能匹配。
表达式中的控制权在不同元素之间转换，因此称为表达式主导。

也说一下相对的DFA引擎。DFA在扫描字符串时会记录当前有效的所有匹配可，逐渐淘汰掉不匹配的可能
在用 /nfa|nfa not/ 匹配"nfa not"时，NFA引擎会从正则表达式第一个字符开始对字符串进行匹配，匹配到nfa结束时匹配结束，不再考虑另外的可能。而DFA引擎会从字符串的第一个字符开始对所有的选项开始匹配，检索到空格时淘汰掉第一个可能选项。
理论上讲，DFA引擎相对会快一些。NFA在尝试不同选项时可能浪费更多的时间。这也提示我们在使用正则表达式时避免使用选项繁多的正则表达式。
### 回溯和匹配优先
在遇到量词和多选结构的情况下，NFA会依次处理各个子表达式或组成元素，遇到需要可能在两个可能成功的选项时它会选择其一，同时记住选择的位置，以备后续需要。这个记住的位置在术语中称为 **备用状态** ，它包含两个位置，一个是正则表达式中的位置，另一个是在字符串中的位置。如果余下的部分也成功了，匹配完成。如果余下的部分匹配失败，引擎会回溯到备用状态继续尝试。
当匹配失败回溯时会返回到距离当前最近储存的备用状态。原则是先进后出（LIFO）,也就是说最先储存的备用状态会在最后尝试匹配。

JS支持忽略优先量词 `??` `*?` `+?` `{min,max}?` 他们会让引擎选择跳过尝试，匹配尽量少的字符。如果在“进行尝试”和“跳过尝试”之间选择，对于匹配优先的量词，引擎会选择“进行尝试”，对于忽略优先的量词，引擎会“跳过尝试”。

比如想要提取一个html标签中内容的时候：
```js
var re = /<div>.*?<\/div>/
var str = "<div>ccc</div><div>xxxyyy</div>"
str.match(re) // ["<d  iv>ccc</div>"]

// 如果不使用忽略优先量词
var re2 = /<div>.*<\/div>/
str.match(re2) // ["<div>ccc</div><div>xxxyyy</div>"]

// 但是这种正则对标签嵌套无能为力,此时我们可以使用环视。
var str2 = "<div><div>ccc</div><div>xxxyyy</div></div>"
str2.match(re) // ["<div><div>ccc</div>"]
var re3 = /<div>(?!<\/?div).*?<\/div>/
str2.match(re3) // ["<div>ccc</div>"]
```
在一些语音所应用的正则引擎中支持固化分组`(?>...)`。当匹配完固化分组中的内容后会丢弃掉分组中保存的备用状态。
在环视结构中，只要环视结构的匹配尝试结束，引擎就会丢掉环视结构中的备选状态。由于JS中不支持固化分组，可以使用环视来模拟。由于环视只匹配一个位置，我们用捕获括号+引用来匹配字符。也就是说`(?>...)`相当于`(?=(...))\1`。
下面用一个处理数字。需求：如果小数第三位为0，则保留两位，如果第三位不为0，则保留三位小数。
```js
var re0 = /(\.\d\d[1-9]?)\d+/
var str0 = "0.330000001"
var str1 = "0.625"

str0.replace(re0, '$1') // 0.33
str1.replace(re0, '$1') // 0.62

var re1 = /(\.\d\d(?=([1-9]?))\2)\d+/
str0.replace(re1, '$1') // 0.33
str1.replace(re1, '$1') // 0.625
// re1匹配str0的时候，引擎匹配到环视结构，[1-9]?没有匹配任何内容，\d+匹配后面的所有数字。
// re1匹配str1的时候，[1-9]?匹配到了'5'，\d+匹配失败，但是环视结构已经丢弃了备选状态，整个正则匹配失败。
```

在传统NFA引擎中，多选结构既不是匹配优先也不是忽略优先，而是按顺序排列的。对于一个多选结构，引擎会逐个对选项进行匹配，匹配成功一个选项的时候则匹配成功。在DFA和POSIX NFA引擎中有匹配优先的多选，总是匹配所有分支中能匹配最多文本的。这就是最开始进行引擎检测的原理。