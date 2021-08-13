## SpringBoot是什么 ##
SpringBoot是构建所有基于Spring的应用程序的起点,是为了让您尽快启动和运行而设计的,具有最少的Spring配置.

SpringBoot具有以下特色功能:  

- 创建独立的Spring应用
- 内嵌Tomcat、Jetty或Underow容器（无需部署War包)
- 提供固定的"starter"依赖以简化构建配置
- 自动配置Spring和第三方库
- 提供生产开箱即用的特性,包括衡量指标,健康检查和外部化配置
- 绝对没有代码生成，也不需要XML配置

## 快速开始 ##
如果想快速创建一个SpringBoot项目,我们可以使用Spring Initializr初始化,然后填写项目的基本信息就可以快速创建maven项目
### 准备工作 ###
- IDEA
- JDK 1.8及以上
-  Maven 3.2+ 或者 Gradle 4+

### 初始化 ###
打开IDEA,新建一个project,选择Spring Initializr 
![](https://note.youdao.com/yws/public/resource/13e45ac31576fe901e85f3c64ac06332/xmlnote/DBC4A9F1F67A431684B814B25768E62E/2261)  

填写项目的基本信息,包括项目名,简介,GroupId,AircraftId等  

![](https://note.youdao.com/yws/public/resource/13e45ac31576fe901e85f3c64ac06332/xmlnote/3E217A7051024D549D348331E500750B/2263)

配置依赖,勾选后初始化的项目pom文件会自动生成  

![](https://note.youdao.com/yws/public/resource/13e45ac31576fe901e85f3c64ac06332/xmlnote/0472F06DD9C24CA3854CF5C23DEE668A/2265)  

最后项目的结构如下:  

![](https://note.youdao.com/yws/public/resource/13e45ac31576fe901e85f3c64ac06332/xmlnote/066E13FF51B34A6A858FD88DC7D20A22/2267)

至此,一个简单的web应用初始化完毕,包括Application启动类,application.properties配置文件,测试类,pom文件等.  

#### 1.pom ####
Spring Initializr初始化后会生成一个pom文件,详细如下:  

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
		<modelVersion>4.0.0</modelVersion>
		<parent>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-parent</artifactId>
			<version>2.1.7.RELEASE</version>
			<relativePath/> <!-- lookup parent from repository -->
		</parent>
		<groupId>com.example</groupId>
		<artifactId>hello</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<name>hello</name>
		<description>Demo project for Spring Boot</description>
	
		<properties>
			<java.version>1.8</java.version>
		</properties>
	
		<dependencies>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter</artifactId>
			</dependency>
	
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-devtools</artifactId>
				<scope>runtime</scope>
				<optional>true</optional>
			</dependency>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-test</artifactId>
				<scope>test</scope>
			</dependency>
		</dependencies>
	
		<build>
			<plugins>
				<plugin>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-maven-plugin</artifactId>
				</plugin>
			</plugins>
		</build>
	
	</project>



