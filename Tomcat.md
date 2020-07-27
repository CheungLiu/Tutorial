## Tomcat

通过配置tomcat，在服务器上下载文件，环境信息：

```
OS：Ubuntu16.04
JDK：1.8.0_252
Tomcat: 8.5.56
```

[TOC]

### 一、下载 Tomcat

1. 到Apache Tomcat®官网，选择core中的tar.gz包[下载](https://tomcat.apache.org/download-80.cgi)，通过winSCP将包传到服务器

2. 将文件解压，赋予访问更改权限

   ```
   sudo chmod 755 -R apache-tomcat-8.5.56
   ```

3. 进入到apache-tomcat-8.5.56/bin修改启动脚本

   ```
   sudo vim startup.sh
   ```

   在最后行插入

   ```
   #set java environment
   export JAVA_HOME=/(根路径).../JDK
   export JRE_HOME=${JAVA_HOME}/jre
   export CLASSPATH=.:%{JAVA_HOME}/lib:%{JRE_HOME}/lib
   export PATH=${JAVA_HOME}/bin:$PATH
   
   #tomcat
   export TOMCAT_HOME=/（根路径）.../apache-tomcat-8.5.56
   ------------------------------------------------------
   //以.9的为例
   #set java environment
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
   export JRE_HOME=${JAVA_HOME}/jre
   export CLASSPATH=.:%{JAVA_HOME}/lib:%{JRE_HOME}/lib
   export PATH=${JAVA_HOME}/bin:$PATH
   
   #tomcat
   export TOMCAT_HOME=/home/lc/software/apache-tomcat-8.5.56
   ```

4. 如果tomcat默认的8080端口被占用，需要进入apache-tomcat-8.5.56/conf修改server.xml中的Connector

   ```
   <Connector port="端口号" protocol="HTTP/1.1"
                  connectionTimeout="20000"
                  redirectPort="8443" />
   ```

   ***在将端口放在防火墙外面，设置转发路由***

5. 启动tomcat

   ```
   sudo ./startup.sh
   //启动成功
   Using CATALINA_BASE:   /home/lc/software/apache-tomcat-8.5.56
   Using CATALINA_HOME:   /home/lc/software/apache-tomcat-8.5.56
   Using CATALINA_TMPDIR: /home/lc/software/apache-tomcat-8.5.56/temp
   Using JRE_HOME:        /usr/lib/jvm/java-8-openjdk-amd64/jre
   Using CLASSPATH:       /home/lc/software/apache-tomcat-8.5.56/bin/bootstrap.jar:/home/lc/software/apache-tomcat-8.5.56/bin/tomcat-juli.jar
   Tomcat started.
   ```

6. 关闭tomcat

   同样需要在bin下的shutdown.sh对应位置添加信息

   ```
   #set java environment
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
   export JRE_HOME=${JAVA_HOME}/jre
   export CLASSPATH=.:%{JAVA_HOME}/lib:%{JRE_HOME}/lib
   export PATH=${JAVA_HOME}/bin:$PATH
   
   #tomcat
   export TOMCAT_HOME=/home/lc/software/apache-tomcat-8.5.56
   ```

   执行

   ```
   sudo ./shutdown.sh
   ```

7. 开机自动启动tomcat

   需要编辑文件/etc/rc.local，这里存放着开机自启动的程序。（配置在/etc/profile和/etc/bash.bashrc文件中的内容是当有用户登录时才起作用，这不符合tomcat需要启动的实际情况）

   在最后一行之前加入如下信息：（配置你自己的tomcat的startup.sh文件的路径）

   ```
   #set java environment
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
   export JRE_HOME=${JAVA_HOME}/jre
   export CLASSPATH=.:%{JAVA_HOME}/lib:%{JRE_HOME}/lib
   export PATH=${JAVA_HOME}/bin:$PATH
   
   /home/lc/software/apache-tomcat-8.5.56/bin/startup.sh
   ```

   执行命令reboot重启系统，完成，通过ip:端口就能访问Apache Tomcat

### 二、通过 url 访问服务器文件

1. 在apache-tomcat-8.5.56t的webapps下创建***文件夹***
2. 将要下载的文件放入该***文件夹***
3. 重启tomcat
4. 访问ip:端口/文件夹/文件