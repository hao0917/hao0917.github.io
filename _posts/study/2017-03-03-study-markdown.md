---
layout: post
title: MarkDown 学习
description: 介绍一些简单的MarkDown语法
date: 2017-03-03
category: blog
---

# 我的Markdown

## 关于标题
```
有两种方式

第一种
一级标题  
======
二级标题  
-------
共二级标题

第二种
#   一级标题  
##  二级标题  
### 三级标题  
以此类推，总共六级标题
```

一级标题  
======
二级标题  
-------
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

## 关于列表

1. 无序列表
```  
 * AAA
 * BBB
 * CCC
```
 * AAA
 * BBB
 * CCC
2. 有序列表
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

```
[Google](www.google.com) [我的博客](http://www.dreamhao.com/)

插入图片
```
插入图片的语法很像，区别在一个 !号
链接为：[]()

图片为：![](){ImgCap}{/ImgCap}
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

```  
  \`\`\`
  public static voiid main(String[] args){
      System.out.println("Hello World");
  }
  \`\`\`
```

```
public static voiid main(String[] args){
    System.out.println("Hello World");
}
```
