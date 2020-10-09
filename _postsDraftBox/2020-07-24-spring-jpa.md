---
layout:     post
title:      SpringBoot + JPA
subtitle:   基于Kotlin的SpringBoot的JPA应用
description: 基于Kotlin的SpringBoot的JPA应用
date:        2020-07-24 11:56:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-spring.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-spring.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   server
tags:
    - Spring
    - Cache
---

## 介绍
主要看一下如何在Kotlin中如何使用JPA

项目基本信息如下：
- 基于Spring Data JPA
- 使用Kotlin
- 使用Gradle构建

基础项目的搭建参考：[SpringBoot + Kotlin + Gradle学习记录](/spring-boot-learn/)

## 依赖
首先，我们在项目的`build.gradle.kts`里的加入依赖库并导入
~~~ groovy
dependencies {
    ...
	implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    runtimeOnly("mysql:mysql-connector-java")
}
~~~

>**ps: 可能遇到的问题**:<br>
**org.springframework.boot.autoconfigure.jdbc.DataSourceProperties$DataSourceBeanCreationException: Failed to determine a suitable driver class**, 解决办法是在xxxApplication.kt里@SpringBootApplication后面加上例外：
~~~kotlin
...
@SpringBootApplication(exclude = [DataSourceAutoConfiguration::class])
class SpringbootdemoApplication
...
~~~

## 定义JPA实体
在 com.onetree.springbootdemo.model 下创建学生实体 StudentModel.kt：
~~~kotlin
package com.onetree.springbootdemo.model

import javax.persistence.*

@Entity
data class StudentModel(
        /**
         * 学生id，自动生成
         */
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        val id: Int,

        /**
         * 学生名字,不可为空
         */
        @Column(nullable = false)
        val name: String,

        /**
         * 学生email，可以为空
         */
        @Column(nullable = true)
        val email: String? = null,

        /**
         * 学生手机号码，可以有多个，关联存储在另外一张表中
         */
        @OneToMany(cascade = [CascadeType.ALL], fetch = FetchType.EAGER)
        val phone: List<PhoneModel>? = null
)
~~~

在 com.onetree.springbootdemo.model 下创建手机号码实体 PhoneModel.kt：
~~~kotlin
package com.onetree.springbootdemo.model

import javax.persistence.Column
import javax.persistence.Entity
import javax.persistence.GeneratedValue
import javax.persistence.GenerationType
import javax.persistence.Id

/**
 * 手机号码
 */
@Entity
data class PhoneModel(
        /**
         * 手机号码id，自动生成
         */
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        val id: Int,

        /**
         * 手机号码，不能为空
         */
        @Column(nullable = false)
        val number: String
)
~~~

JPA提供的注解, 比如 `@Entity` , `@Column` 和 `@Id` 等等在这里可以随意使用

## 存储库类
有了实体，我们来定义相关的存储库类：
创建学生 StudentModel 实体对应存储库类 StudentRepository.kt
~~~kotlin
package com.onetree.springbootdemo.repo

import com.onetree.springbootdemo.model.StudentModel
import org.springframework.data.jpa.repository.JpaRepository
import org.springframework.data.jpa.repository.JpaSpecificationExecutor
import org.springframework.stereotype.Repository

@Repository
interface StudentRepository : JpaRepository<StudentModel, Long>, JpaSpecificationExecutor<StudentModel> {
    fun findByNameLike(pattern: String): List<StudentModel>
}
~~~

## 应用程序入口
有了实体和对应的存储库类，我们看看怎么使用它们：
在 SpringbootdemoApplication.kt 中这样调用：
~~~kotlin
package com.onetree.springbootdemo

import com.onetree.springbootdemo.model.PhoneModel
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.CommandLineRunner
import com.onetree.springbootdemo.model.StudentModel
import com.onetree.springbootdemo.repo.StudentRepository
import org.springframework.boot.SpringApplication
import org.springframework.context.annotation.Bean

@SpringBootApplication
class SpringbootdemoApplication {

	@Bean
	fun doProcess(repo: StudentRepository) = CommandLineRunner {
		// 存储一些测试数据
		repo.save(StudentModel(0, "韩梅梅", "mmhan@xxx.com", listOf(
				PhoneModel(0, "13200000000")
		)))
		repo.save(StudentModel(0, "李明", "mli@xxx.com", listOf(
				PhoneModel(0, "13200000000"),
				PhoneModel(1, "13200000001")
		)))
		repo.save(StudentModel(0, "王磊", "lwangu@xxx.com", null))

		// 既然添加了一个Student 李明，搜索下该条记录
		val entity = repo.findByNameLike("李明")[0]
		println("查询到的记录应该是 李明 : $entity")
	}

}

fun main(args: Array<String>) {
	SpringApplication.run(SpringbootdemoApplication::class.java, *args)
}
~~~

## 单元测试