---
layout:     post
title:      Spring Boot学习记录
subtitle:   简化的Spring配置
description: 通过Spring Boot可以轻松地创建和运行独立的，可以商用的应用程序
date:        2020-07-24 11:56:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-spring.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-spring.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   server
tags:
    - Spring Boot
    - server
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
a) **Project**：使用什么构建工具, 这里选择大多数Java比较熟悉的 Maven Project<br>
b) **Language**: 开发语言，Java<br>
c) **Spring Boot**: 选择Spring Boot的版本，选择最新的稳定版 2.3.1<br>
d) **Project Metadata**: 项目的基本配置，可以根据自己的情况填写<br>
e) **Dependencies**: 选择要加入的Spring Boot组件, 这里选择 Spring Web, 可以写Restful接口<br>
例如：
![配置示例](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-spring-initializr.jpg)

2. 点击 **GENERATE** 会下载一个压缩包，名字和Artifact利配置的一样
3. 解压下载下来的springdemo.zip, 并用编译器以Maven方式导入(比如IntelliJ IDEA)：
Import Project(或者File -> Open) -> Import project from external model -> Maven，然后一路Next。

如果使用的额是UntelliJ IDEA, 可以在IDE中直接创建Spring Boot的项目：
File -> New -> Spring Initializr, 后面配置跟官网差不多，
![配置示例](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-spring-initializr-idea.jpg) 
Dependencies选择 Developer Tools -> Web -> Spring Web
![配置示例](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-spring-initializr-idea2.jpg)
点击Next，配置工程目录，Finish就完成项目的创建了。

如果开发环境没有安装Java， 
通过`java --version`查看当前环境JAVA版本情况。由于前面配置的是Java 11
如果没有安装11，可以去这里下载指定版本 [JDK官方下载](https://www.oracle.com/java/technologies/javase-downloads.html) 

### 项目结构
![项目结构](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-spring-project-catalog.jpg)
由上图所示，Spring Boot的基础结构一共三个文件：
- **src/main/java**: SpringdemoApplication(程序的入口)
- **src/main/resources**: application.properties（程序的配置）
- **src/test**: SpringdemoApplicationTests(单元测试入口)

打开 **pom.xml**，看一下Spring Boot有哪些依赖：
![项目结构](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-spring-project-pom.jpg)
由上图所示，pom.xml只要有几个部分：
- **基本配置**: 创建项目时输入的Project Metadata部分，也是项目的基本信息，包括：
    - **modelVersion**: pom模型版本，maven2和3只能为4.0.0
    - **groupId**: 这是项目组的编号，这在组织或项目中通常是独一无二的。 例如，一家银行集团com.company.bank拥有所有银行相关项目。
    - **artifactId**: 这是项目的ID。这通常是项目的名称。 例如，consumer-banking。 除了groupId之外，artifactId还定义了artifact在存储库中的位置。
    - **version**: 这是项目的版本。与groupId一起使用，artifact在存储库中用于将版本彼此分离。 例如：com.company.bank:consumer-banking:1.0，com.company.bank:consumer-banking:1.1
    - **name**: 项目名字
    - **description**: 项目描述
- **依赖配置**: 
    - **parent**: 用于确定父项目的坐标位置。
    - **dependencies**: 项目相关依赖配置，如果在父项目写的依赖，会被子项目引用。
    - **properties**: 用于定义pom常量。
- **构建配置**: 
    - **build**: 默认使用了spring-boot-maven-plugin，配合spring-boot-starter-parent就可以把Spring Boot应用打包成JAR来直接运行。

### 试着编写一个Http接口
创建src/main/java/com.onetree.springdemo下建个controller和model的目录 
在model目录添加 **PersonModel** 类作为数据模型，编写如下：
```java
package com.onetree.springdemo.model;

public class PersonModel {
    public String name;

    public PersonModel() {
    }

    public PersonModel(String name) {
        this.name = name;
    }
}
```

在该包名下创建 **PersonController** 类作为配置restful接口，编写如下：
>注意：springboot的启动的Application必须放在controller类的目录外面，要不然扫描不到

```java
@RestController()
@RequestMapping("person")
public class PersonController {

    @GetMapping("welcome")
    public String gay() {
        return "Welcome Gay";
    }

    @GetMapping("find/{id}")
    public String findById(@PathVariable Long id) {
        if(id == 0) return "Alien";
        return "Tom";
    }

    @GetMapping("findByName")
    public String findByName(@RequestParam String name) {
        if(name == null) return "Alien";
        return "Jack";
    }

    @PostMapping("add")
    public String add(@RequestBody PersonModel pereson) {
        if(person == null) return "person出错，添加失败";
        return "添加成功";
    }

}

```
启动程序，用postman等工具请求 `http://localhost:8080/person/welcome`, 可以看到返回的 `Welcome Gay`
其他接口也用类似的请求方式

### 编写单元测试用例
这里引入一个日志的注解来简化日志调用。
1. 在pom.xml里加入如下代码导入依赖

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

如果注解@Slf4j注入后找不到变量log，需要IDEA安装lombok插件，
Prefences -> Plugins -> Marketplace 里搜索 **lombok**插件，如下图
![](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-spring-Slf4j.jpg)
完成安装后重启IDEA

写个单例测试上面的`person/findByName`等接口：
```java
@SpringBootTest
@AutoConfigureMockMvc
@Slf4j
class SpringdemoApplicationTests {

	@Autowired
	private MockMvc mock;

	@Test
	void findById() throws Exception {
		MvcResult mvcResult = mock.perform(
				MockMvcRequestBuilders.get("/person/find/2")
						.characterEncoding("UTF-8")
		)
				.andExpect(status().isOk())
				.andReturn();
		MockHttpServletResponse response = mvcResult.getResponse();
		response.setCharacterEncoding("UTF-8");
		String content = response.getContentAsString();
		log.info("findById Result: {}", content);
	}

	@Test
	void findByName() throws Exception{
		MvcResult mvcResult = mock.perform(
				MockMvcRequestBuilders.get("/person/findByName")
						.characterEncoding("UTF-8")
						.param("name", "cat")
		)
				.andExpect(status().isOk())
				.andReturn();
		MockHttpServletResponse response = mvcResult.getResponse();
		response.setCharacterEncoding("UTF-8");
		String content = response.getContentAsString();
		log.info("findByName Result: {}", content);
	}

	@Test
	void add() throws Exception {
		MvcResult mvcResult = mock.perform(
				MockMvcRequestBuilders.post("/person/add")
						.contentType(MediaType.APPLICATION_JSON)
						.characterEncoding("UTF-8")
						.content(asJsonString(new PersonModel("Lily")))
		)
				.andExpect(status().isOk())
				.andReturn();
		MockHttpServletResponse response = mvcResult.getResponse();
		response.setCharacterEncoding("UTF-8");
		String content = response.getContentAsString();
		log.info("add Result: {}", content);
	}

	public static String asJsonString(final Object obj) {
		try {
			return new ObjectMapper().writeValueAsString(obj);
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
	}
}
```

在 findByName() 上运行 `Run Test`, 看到日志：
`[           main] c.o.s.SpringdemoApplicationTests         : findByName Result: Jack`
