---
layout:     post
title:      SpringBoot + MongoDB
subtitle:   使用MongoDB数据
description: 使用Spring data REST 创建和检索存储在MongoDB NoSQL数据库里的Person对象
date:        2020-07-24 11:56:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-spring.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-spring.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   server
tags:
    - Spring
    - MongoDB
---

使用Spring data REST 创建和检索存储在MongoDB NoSQL数据库里的Person对象。

## 安装MongoDB
由于现在MongoDB不在官方提供列表里，在装有HomeBrew的Mac OS上添加仓库，执行
```shell
brew tap mongodb/brew
```
接下来就可以安装MongoDB了，执行
```shell
brew install mongodb-community
```
在安装成功以后，你需要启动mongo服务，
```shell
mongod
```

>如果报错：exception in initAndListen: NonExistentPath: Data directory /data/db not found
有两种解决方案：
方案1：在对应位置手动建目录, 执行 `sudo mkdir -p /data/db`
方案2：自定义已经存在的MongoDB数据库路径: 比如部署时这样执行 `mongod --dbpath ~/Works/demo/mongo/db`

## 写Spring boot + MongoDB的Demo

### 1. 创建项目
创建一个Spring boot, 配置Gradle和Kotlin, 来看看怎么从MongoDB存储和检索数据。
创建过程参考博文: [SpringBoot + Kotlin + Gradle学习记录](/spring-boot-learn/)

在项目的`build.gradle.kts`里的加入依赖库并导入
```groovy
dependencies {
    ...
	implementation("org.springframework.boot:spring-boot-starter-data-mongodb")
}
```

### 2.创建模型类文档
创建Customer的模型类文档:
```kotlin
package com.onetree.springbootdemo.model

import org.springframework.data.mongodb.core.mapping.Document
import org.springframework.data.annotation.Id

@Document(collection = "customer")
class Customer(
        @Id var id: String,
        var firstName: String? = null,
        var lastName: String? = null
){
    override fun toString(): String{
        return "{id= ${id}, firstName = ${firstName}, lastName = ${lastName}}"
    }
}
```
>– @Document: 标识要保留到MongoDB的域对象
– @Id: 划分标识符

### 3.实现自定义的MongoRepository
创建CustomerRepository文件
```kotlin
package com.onetree.springbootdemo.repo

import com.onetree.springbootdemo.model.Customer
import org.springframework.data.mongodb.repository.MongoRepository

interface CustomerRepository : MongoRepository<Customer, String> {

    fun findByFirstName(firstName: String): List<Customer>
    fun findByLastName(lastName: String): List<Customer>

}
```
–打开application.properties，添加配置来连接Mongo Server：
```groovy
spring.data.mongodb.database=jsa_mongodb
spring.data.mongodb.port=27017
```

### 4.实现应用端
在 SpringbootdemoApplication 里加一些输出
```kotlin
package com.onetree.springbootdemo

import org.springframework.boot.autoconfigure.SpringBootApplication
import com.onetree.springbootdemo.repo.CustomerRepository
import org.springframework.boot.CommandLineRunner
import com.onetree.springbootdemo.model.Customer
import org.springframework.boot.SpringApplication
import org.springframework.context.annotation.Bean

@SpringBootApplication
class SpringbootdemoApplication {

	@Bean
	fun doProcess(customerRepo: CustomerRepository) = CommandLineRunner {
		// clear
		customerRepo.deleteAll()

		/**
		 * Save Entities
		 */
		println("Save customers! >>>>>>>>")
		// save an Entity
		val peter = Customer("001", "Peter", "Smith")
		customerRepo.save(peter)

		// save a List Entity
		val custs = listOf(Customer("002", "Mary", "Smith"), Customer("003", "Lauren", "Smith"), Customer("004", "Peter", "Smith"))
		customerRepo.saveAll(custs)

		/**
		 * Find Entities
		 */
		println("Find customers has name is 'Peter'! >>>>>>>>")
		val peters = customerRepo.findByFirstName("Peter")
		// -> Show result
		peters.forEach{println(it)}

		/**
		 * Update an Entity
		 */
		println("Rename a customer which has name is 'Peter' to 'Jack'! >>>>>>>>")
		val jack = peters.get(0)
		jack.firstName = "Jack"
		customerRepo.save(jack)

		/**
		 * Delete an Entity
		 */
		println("Delete the remain Peter customer! >>>>>>>>")
		customerRepo.delete(peters.get(1))

		/**
		 * Find All customer
		 */
		println("Show All Customers! >>>>>>>>")
		val customers = customerRepo.findAll()
		customers.forEach{println(it)}
	}
}

fun main(args: Array<String>) {
	SpringApplication.run(SpringbootdemoApplication::class.java, *args)
}
```

### 5.部署
通过命令行启动MongoDB服务
```shell
mongod
```

然后就能看到运行的日志了：
```shell
Save customers! >>>>>>>>
Find customers has name is 'Peter'! >>>>>>>>
{id= 001, firstName = Peter, lastName = Smith}
{id= 004, firstName = Peter, lastName = Smith}
Rename a customer which has name is 'Peter' to 'Jack'! >>>>>>>>
Delete the remain Peter customer! >>>>>>>>
Show All Customers! >>>>>>>>
{id= 001, firstName = Jack, lastName = Smith}
{id= 002, firstName = Mary, lastName = Smith}
{id= 003, firstName = Lauren, lastName = Smith}
```