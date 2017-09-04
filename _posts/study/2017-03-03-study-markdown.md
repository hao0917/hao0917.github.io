---
layout: post
title: MarkDown 学习
description: 介绍一些简单的MarkDown语法
date: 2017-03-03
categories: [study]
author:     "wenhao"
catalog:    true
tags:
    - study
---

# 我的Markdown学习笔记

[Markdown](https://zh.wikipedia.org/wiki/Markdown) 是一种轻量级标记语言，创始人为约翰·格鲁伯（John Gruber）。它允许人们“使用易读易写的纯文本格式编写文档，然后转换成有效的XHTML(或者HTML)文档”。Markdown 不是想要取代 HTML，甚至也没有要和它相近，它的语法种类很少，只对应 HTML 标记的一小部分。不在 Markdown 涵盖范围之内的标签，都可以直接在文档里面用 HTML 撰写（Markdown兼容HTML）。

## 关于标题
```

#   一级标题  
##  二级标题  
### 三级标题  
以此类推，总共六级标题
```

#   一级标题  
##  二级标题  
### 三级标题  

---

## 关于字体

>用 \*\* 包含一段文本就是粗体，用 \* 包含一段文本就是斜体  

```
*斜体字Hale*  
**粗体字Hale**  
***加粗加斜Hale***  
 ~~中划线~~  
```

 *斜体字Hale*  
 **粗体字Hale**  
 ***加粗加斜Hale***  
 ~~中划线~~  

---

## 关于换行

文本中输入的换行会从最终生成的结果中删除，浏览器会根据可用空间自动换行。  
如果想强迫换行，可以在行尾插入至少两个空格。但是多次强迫换行只能显示一次换行，如果想要多次换行可以使用&lt;br&gt;标签

---

## 关于转义
如果想要在页面显示Markdown或HTML标签需要使用转义字符  
Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号：
```
\   反斜线
`   反引号
*   星号
_   底线
{}  花括号
[]  方括号
()  括弧
#   井号
+   加号
-   减号
.   英文句点
!   惊叹号
```

也可以用HTML转义字符，常用转义字符：

<p>常用转义字符表为：</p>

<table>
<thead>
<tr>
  <th align="center">显示结果</th>
  <th align="center">描述</th>
  <th align="center">实体名称</th>
  <th align="center">实体编号</th>
</tr>
</thead>
<tbody><tr>
  <td align="center"></td>
  <td align="center">空格</td>
  <td align="center">&amp;nbsp;</td>
  <td align="center">&amp;#160;</td>
</tr>
<tr>
  <td align="center">&lt;</td>
  <td align="center">小于号</td>
  <td align="center">&amp;lt;</td>
  <td align="center">&amp;#60;</td>
</tr>
<tr>
  <td align="center">&gt;</td>
  <td align="center">大于号</td>
  <td align="center">&amp;gt;</td>
  <td align="center">&amp;#62;</td>
</tr>
<tr>
  <td align="center">&amp;</td>
  <td align="center">和号</td>
  <td align="center">&amp;amp;</td>
  <td align="center">&amp;#38;</td>
</tr>
<tr>
  <td align="center">“</td>
  <td align="center">引号</td>
  <td align="center">&amp;quot;</td>
  <td align="center">&amp;#34;</td>
</tr>
<tr>
  <td align="center">‘</td>
  <td align="center">撇号</td>
  <td align="center">&amp;apos;(IE不支持)</td>
  <td align="center">&amp;#39;</td>
</tr>
</tbody></table>  

[W3C:HTML转义字符对照表](http://www.w3chtml.com/html/character.html)  
[OSCHINA:HTML转义字符对照表](http://tool.oschina.net/commons?type=2)

---

## 关于列表  

> 无序列表  

```  
 * AAA
 * BBB
 * CCC  
```
  * AAA
  * BBB
  * CCC  

> 有序列表  

```  
1. AAA
2. BBB
3. CCC
```
  1. AAA
  2. BBB
  3. CCC

***

## 关于引用
引用 如果你需要引用一小段别处的句子，那么就要用引用的格式。
```
> 例如这样
```
> 例如这样

---
## 关于链接

插入链接
```
链接为：[]()
例如：
[Google](www.google.com) [我的博客](http://www.dreamhao.com/)
```
[Google](www.google.com) [我的博客](http://www.dreamhao.com/)

插入图片
```
插入图片的语法很像，区别在一个 !号
链接为：[]()
图片为：![](){ImgCap}{/ImgCap}
例如：
![百度图片](https://www.baidu.com/img/baidu.gif)  
```

![百度图片](https://www.baidu.com/img/baidu.gif)  

参考式
```
我收藏夹中的网站[Google][1]、[我的博客][2]

[1]:http://www.google.com "Google"
[2]:http://www.dreamhao.com/ "博客"
```
我收藏夹中的网站[Google][1]、[我的博客][2]

[1]:http://www.google.com "Google"
[2]:http://www.dreamhao.com/ "博客"

***

## 关于制表
```
|  列1左对齐  |  列2居中对齐  |   列3右对齐  |
| :---------|:---------:| ---------:|
| 1         | 1         | 1         |
| 11        | 11        | 11        |
| 1111      | 1111      | 1111      |
```

|  列1左对齐  |  列2居中对齐  |   列3右对齐  |
| :---------|:---------:| ---------:|
| 1         | 1         | 1         |
| 11        | 11        | 11        |
| 1111      | 1111      | 1111      |

***

## 代码框
使用```包裹代码  

``````  
  ```
  public static voiid main(String[] args){
      System.out.println("Hello World");
  }
  ```
``````

```
public static voiid main(String[] args){
    System.out.println("Hello World");
}
```


## 代码块（指定语言）
使用```语言名  包裹代码  
``````  
  ```java
  public static voiid main(String[] args){
      System.out.println("Hello World");
  }
  ```
``````

```java
public static voiid main(String[] args){
    System.out.println("Hello World");
}
```
