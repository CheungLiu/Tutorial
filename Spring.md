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

### 3、IOC	操作 Bean 管理（xml）

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

  ​			id 属性：唯一标识

  ​			class 属性：类全路径（包类路径）

  （3）创建对象时候，默认也是执行无参数构造方法完成对象创建

- 基于 xml 方式注入属性

  **DI：依赖注入，就是注入属性**

  ***IOC和DI的区别：DI是IOC中的一种具体实现，表示依赖注入，需要在创建对象的基础之上完成***

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

#### 2、IOC 操作 Bean 管理（xml 注入其他类型属性）

- 字面量

  - null
  - 属性值包含特殊符号

- 注入属性-外部 bean

  （1）创建两个类 service 类和 dao 类
  （2）***在 service 调用 dao 里面的方法***
  （3）在 spring 配置文件中进行配置

  ```xml
  <!--1 service 和 dao 对象创建-->
  <bean id="userService" class="com.atguigu.spring5.service.UserService">
      <!--注入 userDao 对象
      name 属性：类里面属性名称
      ref 属性：创建 userDao 对象 bean 标签 id 值
      -->
  	<property name="userDao" ref="userDaoImpl"></property>
  </bean>
  <bean id="userDaoImpl" class="com.atguigu.spring5.dao.UserDaoImpl"></bean>
  ```

- 注入属性-内部 bean

  ```xml
  <!--内部 bean-->
  <bean id="emp" class="com.atguigu.spring5.bean.Emp">
      <!--设置两个普通属性-->
      <property name="ename" value="lucy"></property>
      <property name="gender" value="女"></property>
      <!--设置对象类型属性-->
      <property name="dept">
          <bean id="dept" class="com.atguigu.spring5.bean.Dept">
              <property name="dname" value="安保部"></property>
          </bean>
      </property>
  </bean>
  ```

- 注入属性-级联赋值

  (1)第一种写法（后面的bean不同与外部注入bean，级联有value）

  ```xml
  <!--级联赋值-->
  <bean id="emp" class="com.atguigu.spring5.bean.Emp">
  <!--设置两个普通属性-->
      <property name="ename" value="lucy"></property>
      <property name="gender" value="女"></property>
          <!--级联赋值-->
      <property name="dept" ref="dept"></property>
  </bean>
  <bean id="dept" class="com.atguigu.spring5.bean.Dept">
      <property name="dname" value="财务部"></property>
  </bean>
  ```

  （2）第二种写法（需要在Emp中实现dname的get方法）

  ```xml
  <!--级联赋值-->
  <bean id="emp" class="com.atguigu.spring5.bean.Emp">
      <!--设置两个普通属性-->
      <property name="ename" value="lucy"></property>
      <property name="gender" value="女"></property>
      <!--级联赋值-->
      <property name="dept" ref="dept"></property>
      <property name="dept.dname" value="技术部"></property>
  </bean>
  <bean id="dept" class="com.atguigu.spring5.bean.Dept">
      <property name="dname" value="财务部"></property>
  </bean>
  ```

#### 3、IOC 操作 Bean 管理（xml 注入集合属性）

1、注入数组类型属性 

2、注入 List 集合类型属性 

3、注入 Map 集合类型属性

- （1）创建类，定义数组、list、map、set 类型属性，生成对应 set 方法
- （2）在 spring 配置文件进行配置

4、在集合里面设置对象类型值

```xml
<!--创建多个 course 对象-->
<bean id="course1" class="com.atguigu.spring5.collectiontype.Course">
 	<property name="cname" value="Spring5 框架"></property>
</bean>
<bean id="course2" class="com.atguigu.spring5.collectiontype.Course">
 	<property name="cname" value="MyBatis 框架"></property>
</bean>
<!--注入 list 集合类型，值是对象-->
<property name="courseList">
     <list>
         <ref bean="course1"></ref>
         <ref bean="course2"></ref>
     </list>
</property>
```

5、把集合注入部分提取出来

第一步：在 spring 配置文件中引入名称空间 util

第二步：使用 util 标签完成 list 集合注入提取

#### 4、IOC 操作 Bean 管理（FactoryBean）

```
1、Spring 有两种类型 bean，一种普通 bean，另外一种工厂 bean（FactoryBean）
2、普通 bean：在配置文件中定义 bean 类型就是返回类型
3、工厂 bean：在配置文件定义 bean 类型可以和返回类型不一样
	第一步 创建类，让这个类作为工厂 bean，实现接口 FactoryBean
	第二步 实现接口里面的方法，在实现的方法中定义返回的 bean 类型	
```

#### 5、IOC 操作 Bean 管理（bean 作用域）  

1. 在 Spring 里面，设置创建 bean 实例是单实例还是多实例

2. 在 Spring 里面，默认情况下， bean 是单实例对象

3. 如何设置单实例还是多实例

   ```
   （1）在 spring 配置文件 bean 标签里面有属性（scope）用于设置单实例还是多实例
   （2） scope 属性值
       第一个值 默认值， singleton，表示是单实例对象
       第二个值 prototype，表示是多实例对象
   （3） singleton 和 prototype 区别
       第一 singleton 单实例， prototype 多实例
       第二 设置 scope 值是 singleton 时候，加载 spring 配置文件时候就会创建单实例对象
       	设置 scope 值是 prototype 时候，不是在加载 spring 配置文件时候创建 对象，在调用getBean方法时候创建多实例对象
   ```

#### 6、IOC 操作 Bean 管理（bean 生命周期）

1. 生命周期 

   （1）从对象创建到对象销毁的过程

2. bean 生命周期

```
（1）通过构造器创建 bean 实例（无参数构造）
（2）为 bean 的属性设置值和对其他 bean 引用（调用 set 方法）
（3）把 bean 实例传递 bean 后置处理器的方法 postProcessBeforeInitialization
（4）调用 bean 的初始化的方法（需要进行配置初始化的方法）（5）把 bean 实例传递 bean 后置处理器的方法 postProcessAfterInitialization
（6） bean 可以使用了（对象获取到了）
（7）当容器关闭时候，调用 bean 的销毁的方法（需要进行配置销毁的方法）
```

#### 7、IOC 操作 Bean 管理（xml 自动装配）

实际中用的机率很小，为了方便一般采用注解（这里讲的是配置文件 xml 也能做到装配）

1. 什么是自动装配

   根据指定装配规则（属性名称或者属性类型）， Spring 自动将匹配的属性值进行注入

2. 演示自动装配过程

    （1）根据属性名称自动注入

   ```xml
   <bean id="emp" class="com.atguigu.spring5.autowire.Emp" autowire="byName">
   </bean>
   <bean id="dept" class="com.atguigu.spring5.autowire.Dept"></bean>
   ```

   （2）根据属性类型自动注入

   ```xml
   <bean id="emp" class="com.atguigu.spring5.autowire.Emp" autowire="byType">
   </bean>
   <bean id="dept" class="com.atguigu.spring5.autowire.Dept"></bean>
   ```

#### 8、IOC 操作 Bean 管理（外部属性文件)  

1. 直接配置数据库信息

   ```xml
   （1）配置德鲁伊连接池
   （2）引入德鲁伊连接池依赖 jar 包
   <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
       <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
       <property name="url" value="jdbc:mysql://localhost:3306/userDb"></property>
       <property name="username" value="root"></property>
       <property name="password" value="root"></property>
   </bean>
   ```

2. 引入外部属性文件配置数据库连接池

   ```xml
   （1）创建外部属性文件， properties 格式文件，写数据库信息
   （2） 把外部 properties 属性文件引入到 spring 配置文件中
   	 * 引入 context 名称空间
   <!--引入外部属性文件-->
   //自己建立的一个外部配置文件
   <context:property-placeholder location="classpath:jdbc.properties"/>
   <!--配置连接池-->
   <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
       <property name="driverClassName" value="${prop.driverClass}"></property>
       <property name="url" value="${prop.url}"></property>
       <property name="username" value="${prop.userName}"></property>
       <property name="password" value="${prop.password}"></property>
   </bean>
   ```

### 3、IOC 操作 Bean 管理（注解）

