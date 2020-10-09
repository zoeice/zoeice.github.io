---
layout:     post
title:      用Kotlin创建Spring应用
subtitle:   用SpringBoot、Kotlin和Gradle，配上web,mustache,jpa,h2,devtools搭建工程
description: 通过Spring Boot可以轻松地创建和运行独立的，可以商用的应用程序
date:        2020-07-24 11:56:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-spring.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-spring.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   server
tags:
    - Spring
    - Kotlin

---

## 创建基础项目
先来创建一个Spring Boot的工程，我们可以用如下几种方式：

### 1.用官网的初始化工具
使用官方提供的在线初始化工具 [spring initializr](https://start.spring.io/) <br>
我们选择 **Kotlin** 语言, 选择 **Gradle**<br>
>用Kotlin时，Gradle是最常用的构建工具，它提供了Kotlin的DSL。

填写项目的基本信息。
然后选择依赖的库：
- **Spring Web**：用来构建Web应用（包括Restful）
- **Mustache**：一款没有逻辑的模板引擎，只有标签。
- **Spring Data JPA**：在JPA规范下提供Repository层的实现，可以很方便的切换ORM框架。
- **H2 Database**：一个开源的嵌入式数据库引擎。
- **Spring Boot DevTools**：可以热部署。
![配置示例](http://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-spring-initializr-config.jpg)

2. 点击 **GENERATE** 会下载一个压缩包，名字和Artifact利配置的一样
3. 解压下载下来的springbootdemo.zip, 并用编译器以Gradle方式导入(比如IntelliJ IDEA)：
Import Project(或者File -> Open) -> Import project from external model -> Gradle，然后一路Next。

### 2.用命令行
在终端里输使用 **Initializr HTTP API**
~~~shell
$ mkdir buyone && cd buyone
$ curl https://start.spring.io/starter.zip -d language=kotlin -d style=web,mustache,jpa,h2,devtools -d packageName=com.onetree.buyone -d name=BuyOne -o buyone.zip -d type=gradle-project
~~~

### 3.使用IntelliJ IDEA
如果使用的额是UntelliJ IDEA, 可以在IDE中直接创建Spring Boot的项目：
File -> New -> Spring Initializr, 后面配置跟官网差不多，

Dependencies选择 Developer Tools -> 依次选上web,mustache,jpa,h2,devtools等依赖库
点击Next，配置工程目录，Finish就完成项目的创建了。

## 可能遇到的问题
>**ps: 可能遇到的问题**:<br>
>**Cannot inline bytecode built with JVM target 1.8 into bytecode that is being built with JVM target 1.6**，可以这样操作:<br>
`IntelliJ preferences` > `Build, Execution, Deployment` > `Compiler` > `Kotlin Compiler`<br>
`Target JVM version` 改为 1.8， 点击 `Apply`。 就解决了。

>**Error starting ApplicationContext. To display the conditions report re-run your application with 'debug' enabled.**, 可以这样操作：<br>
`Run/Debug Configurations` > `xxApplication` > `Configuration` > `Environment` > `VM Options`里加上
~~~shell
-Ddebug=true
~~~

## Gradle构建

### 外挂程式
build.gradle.kts
~~~kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
	id("org.springframework.boot") version "2.4.0-M2"
	id("io.spring.dependency-management") version "1.0.10.RELEASE"
	kotlin("jvm") version "1.3.72"
	kotlin("plugin.spring") version "1.3.72"
	kotlin("plugin.jpa") version "1.3.72"
}
~~~
其中：

**kotlin-spring插件**: 会自动打开用 `Spring annotations` 进行注解或者元注解的类和方法（与Java不同，默认限定符是final）。例如，这对于能够创建 `@Configuration` 或 `@Transactionalbean` 而不用添加**CGLIB**代理所需的限定符 `open` 很有用。

**kotlin-jpa插件**: 为了支持在**JPA**中使用kotlin非空属性才加的。它为任何用了 `@Entity`，`@MappedSuperclass` 或 `@Embeddable` 注解的类生成无参数的构造函数。

### 编译器选项
Kotlin的主要功能之一是空安全，它在编译期间干净地处理的空值，而不是到运行期间才遇到著名的 `NullPointerException`, 它通过可空声明是应用更安全, 并且不用使用类似 `Optional` 一样额外的包装器来表达"有值或无值"。注意，kotlin允许使用值可空的构造函数。可以查阅[《Kotlin空安全的完整指南》](https://www.baeldung.com/kotlin-null-safety)

build.gradle.kts
~~~kotlin
tasks.withType<KotlinCompile> {
	kotlinOptions {
		freeCompilerArgs = listOf("-Xjsr305=strict")
		jvmTarget = "11"
	}
}
~~~

### 依赖项
spring boot应用的默认配置要用到三个Kotlin的库：
- **kotlin-stdlib-jdk8**：Kotlin标准库的java 8变体
- **kotlin-reflect"**：Kotlin的反射库
- **jackson-module-kotlin**：添加对Kotlin类和数据类的序列化/反序列化的支持

~~~kotlin
dependencies {
	implementation("org.springframework.boot:spring-boot-starter-data-jpa")
	implementation("org.springframework.boot:spring-boot-starter-mustache")
	implementation("org.springframework.boot:spring-boot-starter-web")
	implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
	implementation("org.jetbrains.kotlin:kotlin-reflect")
	implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
	developmentOnly("org.springframework.boot:spring-boot-devtools")
	runtimeOnly("com.h2database:h2")
	testImplementation("org.springframework.boot:spring-boot-starter-test")
}
~~~

springboot的gradle插件自动使用通过Kotlin Gradle插件声明的Kotlin版本

## 了解生成的应用
src/main/kotlin/com/onetree/buyone/BuyoneApplication.kt
~~~kotlin
package com.onetree.buyone

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class BuyoneApplication

fun main(args: Array<String>) {
	runApplication<BuyoneApplication>(*args)
}
~~~

对比Java，你会注意到缺少分号，空类缺少括号，