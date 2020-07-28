# Spring

## 一、课程内容介绍

1、Spring 框架概述 
2、IOC 容器
（1）IOC 底层原理
（2）IOC 接口（BeanFactory，ApplicationContext）
（3）IOC 操作 Bean 管理（基于 xml）
（4）IOC 操作 Bean 管理（基于注解）
3、Aop
4、JdbcTemplate
5、事务管理
6、Spring5 新特性

## 二、Spring5 框架概述

1、Spring 是轻量级的开源的 JavaEE 框架
2、Spring 可以解决企业应用开发的复杂性
3、Spring 有两个核心部分：IOC 和 Aop
（1）IOC：控制反转，把创建对象过程交给 Spring 进行管理
（2）Aop：面向切面，不修改源代码进行功能增强
4、Spring 特点
（1）方便解耦，简化开发
（2）Aop 编程支持
（3）方便程序测试
（4）方便和其他框架进行整合
（5）方便进行事务操作
（6）降低 API 开发难度

### 1、IOC（概念和原理）

1、什么是 IOC
（1）控制反转，把对象创建和对象之间的调用过程，交给 Spring 进行管理
（2）使用 IOC 目的：为了耦合度降低
（3）做**入门案例**就是 IOC 实现
2、IOC 底层原理
（1）xml 解析、工厂模式、反射
3、画图讲解 IOC 底层原理

![](Spring.assets/图2.png)

### 2、IOC（BeanFactory ApplicationContext接口）

1、IOC 思想基于 IOC 容器完成，IOC 容器底层就是对象工厂
2、Spring 提供 IOC 容器实现两种方式：（两个接口）
（1）BeanFactory：IOC 容器基本实现，是 Spring 内部的使用接口，不提供开发人员进行使用
*加载配置文件时候不会创建对象，在获取对象（使用）才去创建对象
（2）ApplicationContext：BeanFactory 接口的子接口，提供更多更强大的功能，一般由开发人
员进行使用
*加载配置文件时候就会把在配置文件对象进行创建
3、ApplicationContext 接口有实现类（Ctrl+h）

```java
ClassPathXmlApplicationContext
FileSystemXmlApplicationContext
```

### 3、IOC	操作 Bean 管理

1、什么是 Bean 管理
（0）Bean 管理指的是两个操作
（1）Spring 创建对象
（2）Spirng 注入属性
2、Bean 管理操作有两种方式
（1）基于 xml 配置文件方式实现
（2）基于注解方式实现

#### 1、IOC 操作 Bean 管理（基于 xml 方式）

- 基于 xml 方式创建对象

  （1）在 spring 配置文件中，使用 bean 标签，标签里面添加对应属性，就可以实现对象创建
  （2）在 bean 标签有很多属性，介绍常用的属性
  * id 属性：唯一标识
  * class 属性：类全路径（包类路径）

  （3）创建对象时候，默认也是执行无参数构造方法完成对象创建

- 基于 xml 方式注入属性
  （1）DI：依赖注入，就是注入属性

  - ***IOC和DI的区别：DI是IOC中的一种具体实现，表示依赖注入，需要在创建对象的基础之上完成***

    1. 使用 set 方法进行注入

       - 创建类，定义属性和对应的 set 方法

       - 在 spring 配置文件配置对象创建，配置属性注入

         ```xml
         <!--2 set 方法注入属性-->
         <bean id="book" class="com.atguigu.spring5.Book">
          	<!--使用 property 完成属性注入
         	 name：类里面属性名称
         	 value：向属性注入的值
              -->
              <property name="bname" value="易筋经"></property>
              <property name="bauthor" value="达摩老祖"></property>
         </bean>
         ```

    2. 使用有参数构造进行注入

       - 创建类，定义属性，创建属性对应有参数构造方法

       - 在 spring 配置文件中进行配置

         ```xml
         <!--3 有参数构造注入属性-->
         <bean id="orders" class="com.atguigu.spring5.Orders">
              <constructor-arg name="oname" value="电脑"></constructor-arg>
              <constructor-arg name="address" value="China"></constructor-arg>
         </bean>
         ```

    3. p 名称空间注入（了解）