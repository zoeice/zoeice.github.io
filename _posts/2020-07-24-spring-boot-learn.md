---
layout:     post
title:      Spring Boot+Kotlin+Gradle学习记录
subtitle:   简化的Spring配置
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

## 介绍
通过Spring Boot可以轻松地创建和运行独立的，可以商用的应用程序

优点：
- 可以创建独立的Spring程序
- 直接嵌入Tomcat， Jetty或者Undertow等容器（无需部署WAR文件）
- 提供一些默认依赖项，简化构建配置，尽可能自动配置Spring和三方库，让开发者更快入门
- 没有冗余代码生成和XML配置的要求

## 快速入门
### 创建基础项目
1. 使用官方提供的 [spring initializr](https://start.spring.io/) 来创建项目, 选项说明：<br>

a) **Project**：使用什么构建工具, 这里选择比较流行的 Gradle <br>

>Andy Wilkinson在Twitter说的大概意思是：Spring Boot 2.3中，将构建从Maven迁移到了Gradle，大大减少了构建项目所需的时间。详细看大佬的博客[2020/06/08/migrating-spring-boot](https://spring.io/blog/2020/06/08/migrating-spring-boot-?spm=a2c6h.12873639.0.0.626627832xfDbM)

b) **Language**: 开发语言，Kotlin<br>
c) **Spring Boot**: 选择Spring Boot的版本，选择最新的稳定版 2.3.2<br>
d) **Project Metadata**: 项目的基本配置，可以根据自己的情况填写<br>
e) **Dependencies**: 选择要加入的Spring Boot组件, 这里选择 Spring Web, 可以写Restful接口<br>
例如：
![配置示例](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-spring-initializr.jpg)

2. 点击 **GENERATE** 会下载一个压缩包，名字和Artifact利配置的一样
3. 解压下载下来的springbootdemo.zip, 并用编译器以Gradle方式导入(比如IntelliJ IDEA)：
Import Project(或者File -> Open) -> Import project from external model -> Gradle，然后一路Next。

如果使用的额是UntelliJ IDEA, 可以在IDE中直接创建Spring Boot的项目：
File -> New -> Spring Initializr, 后面配置跟官网差不多，
![配置示例](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-spring-initializr-idea.jpg) 
Dependencies选择 Developer Tools -> Web -> Spring Web
![配置示例](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-spring-initializr-idea2.jpg)
点击Next，配置工程目录，Finish就完成项目的创建了。

>**ps: 可能遇到的问题**:<br>
>**Cannot inline bytecode built with JVM target 1.8 into bytecode that is being built with JVM target 1.6**，可以这样操作:<br>
`IntelliJ preferences` > `Build, Execution, Deployment` > `Compiler` > `Kotlin Compiler`<br>
`Target JVM version` 改为 1.8， 点击 `Apply`。 就解决了。


### 项目结构
![项目结构](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-spring-project-catalog.jpg)
由上图所示，Spring Boot的基础结构一共三个文件：
- **src/main/kotlin**: SpringdemoApplication(程序的入口)
- **src/main/resources**: application.properties（程序的配置）
- **src/test/kotlin**: SpringdemoApplicationTests(单元测试入口)

打开 **build.gradle.kts**，看一下Spring Boot有哪些依赖：
![项目结构](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-spring-project-buildgradle.jpg)

### 试着编写一个Http接口
创建src/main/java/com.onetree.springbootdemo下建个controller和model的目录 
在model目录添加 **PersonModel** 类作为数据模型，编写如下：
```kotlin
package com.onetree.springbootdemo.model

class PersonModel {
    var name: String = ""

    constructor() {}

    constructor(name: String) {
        this.name = name
    }
}
```

在该包名下创建 **PersonController** 类作为配置restful接口，编写如下：
>注意：springboot的启动的Application必须放在controller类的目录外面，要不然扫描不到

```kotlin
package com.onetree.springbootdemo.controller

import com.onetree.springbootdemo.model.PersonModel
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.PostMapping
import org.springframework.web.bind.annotation.RequestParam
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.PathVariable
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RestController

@RestController
@RequestMapping("person")
class PersonController {

    @GetMapping("welcome")
    fun gay(): String {
        return "Welcome Gay"
    }

    @GetMapping("find/{id}")
    fun findById(@PathVariable id: Long?): String {
        return if (id == 0L) "Alien" else "Tom"
    }

    @GetMapping("findByName")
    fun findByName(@RequestParam name: String?): String {
        return if (name == null) "Alien" else "Jack"
    }

    @PostMapping("add")
    fun add(@RequestBody person: PersonModel): String {
        return if (person == null) "person出错，添加失败" else "添加成功"
    }

}
```
点击Idea里的`run`按钮或者在工程根目录执行命令： `./mvnw spring-boot:run`来运行项目
用postman等工具请求 `http://localhost:8080/person/welcome`, 可以看到返回的 `Welcome Gay`
其他接口也用类似的请求方式

Get也可以直接执行命令：
```shell
curl localhost:8080/person/welcome
```
如果是Post请求，可以执行：
```shell
curl -H "Content-Type:application/json" -X POST -d'{"name": "lili"}' localhost:8080/person/add
```
都可以成功看到结果。

### 编写单元测试用例
这里引入一个日志的注解来简化日志调用。
1. 在build.gradle.kts里加入如下代码导入依赖

```groovy
dependencies {
	...
	compileOnly("org.projectlombok:lombok")
	testCompileOnly("org.projectlombok:lombok")
	annotationProcessor("org.projectlombok:lombok")
	testAnnotationProcessor("org.projectlombok:lombok")
}
```

如果注解@Slf4j注入后找不到变量log，需要IDEA安装lombok插件，
Prefences -> Plugins -> Marketplace 里搜索 **lombok**插件，如下图
![](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-spring-Slf4j.jpg)
完成安装后重启IDEA

写个单例测试上面的`person/findByName`等接口：
```kotlin
package com.onetree.springbootdemo

import org.junit.jupiter.api.Test
import org.springframework.boot.test.context.SpringBootTest
import com.fasterxml.jackson.databind.ObjectMapper
import com.onetree.springbootdemo.model.PersonModel
import jdk.nashorn.internal.runtime.regexp.joni.Config.log
import lombok.extern.slf4j.Slf4j
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders
import org.springframework.test.web.servlet.MockMvc
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc
import org.springframework.http.MediaType
import org.springframework.test.web.servlet.result.MockMvcResultMatchers.status


@SpringBootTest
@AutoConfigureMockMvc
@Slf4j
internal class SpringdemoApplicationTests {

	@Autowired
	private val mock: MockMvc? = null

	@Test
	@Throws(Exception::class)
	fun findById() {
		val mvcResult = mock!!.perform(
				MockMvcRequestBuilders.get("/person/find/2")
						.characterEncoding("UTF-8")
		)
				.andExpect(status().isOk())
				.andReturn()
		val response = mvcResult.response
		response.setCharacterEncoding("UTF-8")
		val content = response.contentAsString
		log.printf("findById Result: {}", content)
	}

	@Test
	@Throws(Exception::class)
	fun findByName() {
		val mvcResult = mock!!.perform(
				MockMvcRequestBuilders.get("/person/findByName")
						.characterEncoding("UTF-8")
						.param("name", "cat")
		)
				.andExpect(status().isOk())
				.andReturn()
		val response = mvcResult.response
		response.setCharacterEncoding("UTF-8")
		val content = response.contentAsString
		log.printf("findByName Result: {}", content)
	}

	@Test
	@Throws(Exception::class)
	fun add() {
		val mvcResult = mock!!.perform(
				MockMvcRequestBuilders.post("/person/add")
						.contentType(MediaType.APPLICATION_JSON)
						.characterEncoding("UTF-8")
						.content(asJsonString(PersonModel("Lily")))
		)
				.andExpect(status().isOk())
				.andReturn()
		val response = mvcResult.response
		response.setCharacterEncoding("UTF-8")
		val content = response.contentAsString
		log.printf("add Result: {}", content)
	}

	companion object {

		fun asJsonString(obj: Any): String {
			try {
				return ObjectMapper().writeValueAsString(obj)
			} catch (e: Exception) {
				throw RuntimeException(e)
			}

		}
	}
}
```

在 findByName() 上运行 `Run Test`, 看到日志：
`[           main] c.o.s.SpringdemoApplicationTests         : findByName Result: Jack`
