---
layout:     post
title:      "Moven"
subtitle:   "Moven介绍"
date:       2018-1-20 12:00:00
author:     "wenhao"
catalog:    true
tags:
    - study
---
## Maven，Gradle与MavenCentral和Jcenter
### 总的来说
1. Maven 是项目管理和构建工具
2. Gradle 也是项目管理和构建工具
3. mavenCentral和Jcenter是由Maven衍生出的两个比较流行的代码仓库（类似的代码库还有很多）

### Maven
>Apache Maven，是一个软件（特别是Java软件）项目管理及自动构建工具，由Apache软件基金会所提供。基于项目对象模型（缩写：POM）概念，Maven利用一个中央信息片断能管理一个项目的构建、报告和文档等步骤。
Maven也可被用于构建和管理各种项目，例如C#，Ruby，Scala和其他语言编写的项目。Maven曾是Jakarta项目的子项目，现为由Apache软件基金会主持的独立Apache项目。

Maven项目使用项目对象模型（Project Object Model，POM）来配置。
项目对象模型存储在名为 pom.xml 的文件中.Maven的本质是以POM来管理整个项目.

pom.xml举例
```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.myapp</groupId>
  <artifactId>myapp</artifactId>
  <packaging>jar</packaging>
  <version>1.0</version>
  <name>myapp</name>
  <url>http://maven.apache.org</url>

  <dependencies>
    <dependency>
      <groupId>com.google.code.findbugs</groupId>
      <artifactId>jsr305</artifactId>
      <scope>provided</scope>
    </dependency>
  </dependencies>

</project>
```
* **project**  这个元素是Maven的pom.xml文件的顶层元素
* **modelVersion**  项目对象模型(POM)的版本
* **groupId**  这个元素表示创建这个项目的组织的唯一标识。这个元素的值是区分一个项目的关键信息之一，它的值通常是由该项目的组织的域名的反写产生的（和Java的包名的机制一样）
* **artifactId**  这个元素是唯一标识了该项目最终生成的artifact一个项目的artifact一般是一个jar文件。一个项目最终生成的artifact的名字的格式是<artifactId>-<version>.<extension>
* **packaging**  这个元素指定了该artifact打包的类型（比如：jar、war、ear等）默认jar
* **version**  这个元素指定了项目生成的artifact的版本号
* **name**  这个元素表示项目的名字，常用于生成的文档中。
* **url**  这个元素表示该项目所在的站点，常用于生成的文档中。
* **description**  这个元素提供了对项目的描述，通常用于生成的文档中。

在项目创建后除了会生成pom.xml文件，还会产生一个标准的目录结构
```
myapp
|-- pom.xml
`-- src
    |-- main
    |   `-- java
    |       `-- com
    |           `-- myapp
    |               `-- App.java
    `-- test
        `-- java
            `-- com
                `-- myapp
                    `-- AppTest.java
```
### MavenCentral和Jcenter
[MavenCentral](https://oss.sonatype.org/content/repositories/releases/) 是由sonatype.org维护的Maven仓库  

[Jcenter](http://jcenter.bintray.com/)是由 bintray.com维护的Maven仓库

这两个仓库是完全独立的，也就是说有些库可能存在于MavenCentral而Jcenter中却没有，反之亦然。在AndroidStudio中原默认使用MavenCentral，后改为默认使用Jcenter。

### Gradle
Gradle是不同于Maven的新型项目构建工具，Maven使用XML描述项目，Gradle使用Groovy描述项目
### Gradle 中引用Maven仓库
虽然在AndroidStudio中使用Gradle来构建项目，但是却并不影响我们使用Maven仓库
当我们需要引入Maven仓库中项目时只需按照规则填写库的GROUPID、POM_ARTIFACT_ID、VERSION即可。规则为：

>groupId:artifactId:version  

比如junit在Jcenter仓库中定义为
```xml
<groupId>junit</groupId>
<artifactId>junit</artifactId>
<version>4.12</version>
```
那么我们引入的时候按照语法规则
compile 'groupId:artifactId:version'
可以确定语句为
compile 'junit:junit:4.12'
