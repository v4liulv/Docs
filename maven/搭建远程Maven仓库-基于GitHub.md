# 搭建远程Maven仓库-基于GitHub

------------

[TOCM]

[TOC]

## 原理
通过maven-deploy-plugin结合GitHub的site-maven-plugin插件进行本地工程的远程提交工程发布到自定义的GitHub仓库上。

**maven-deploy-plugin**
主要是为了用来将本地工程部署到远程仓库中。

**site-maven-plugin**
集成了与GitHub集成的Maven插件。

## 目的

* 通过远程管理的方式管理本地的工程Jar包依赖。
* 提供需要项目、框架、工具的Maven依赖，而不是死板的Jar包提供。
* 为依赖升级提供了更便捷和更灵活方式。
* 提供他人的远程Maven库依赖下载和使用。

## 搭建流程


### 创建mvn-repo仓库

首先在你的github上创建一个maven-repo仓库，这个最后将作为实际上jar包发布的仓库

**创建方式：**

> 1. GitHub网站上进行创建名为maven-repo仓库.
   如果需要本地化的查看管理此仓库，可以再通过GitHub Desktop的Clone repository到本地.

 > 2. 本地在GitHub的工作文件夹创建maven-repo文件夹，然后通过GitHub Desktop的Add local repository的方式创建GitHub的maven-repo仓库。

**注意**：本地的maven-repo项目不需要提交任何东西到GithHub上面，统一都是由Maven的方式提交。

### 配置本地Maven提交GitHub验证信息
找到本地Maven配置文件**settings.xml**如（比如：D:\maven\apache-maven-3.3.9\conf\settings.xml）,找到***servers***标签标签，添加如下配置:
```xml
    <server>
        <id>github</id>
        <username>github的用户名</username>
        <password>github的密码</password>
    </server>
```
**特别提醒：**如果您的Maven工程是多模块的模式，下面的所有配置最好都配置到全局的父模块的pom.xml文件中。

### 添加发布本地仓库插件
在需要发布的项目中的pom文件中**plugins**标签中加入以下插件：

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-deploy-plugin</artifactId>
	<version>2.8.2</version>
	<configuration>
	<altDeploymentRepository>internal.repo::default::file://${project.build.directory}/mvn-repo</altDeploymentRepository>
	</configuration>
</plugin>
```
然后运行 mvn clean deploy 命令，即可在对应项目中的target/mvn-repo目录下找到本地的jar

### Maven项目添加GitHub的全局服务
在需要发布的Maven项目中的pom文件中**properties**标签中添加如下配置：
```xml
<github.global.server>github</github.global.server>
```

### Maven项目添加GitHub集成
在需要发布的Maven项目的pom.xml文件中**plugins**标签中添加下插件配置：

```xml
<plugin>
	<groupId>com.github.github</groupId>
	<artifactId>site-maven-plugin</artifactId>
	<version>0.12</version>
	<configuration>
	   <!-- GitHub提交的信息 -->
		<message>Maven artifacts for ${project.version}</message>
		<!--提交的目录-->
		<outputDirectory>${project.build.directory}/mvn-repo</outputDirectory>
		<!--将被更新为提交的分支ref, 其中master为GitHub提交的角色-->
		<branch>refs/heads/master</branch>
		<!--子元素将被视为从outputDirectory中包含的模式-->
		<includes>
			<include>**/*</include>
		</includes>
		<!-- <path>com.sinobest.kshfx</path>-->
		<!-- 是否与当前树合并，或者完全替换提交指向的树, 默认为false-->
		<merge>true</merge>
		<!-- 在该站点的根目录下是否总是创造一个.nojekyll文件。如果您的站点包含任何以下划线开头的文件夹，那么该设置应该启用。-->
		<noJekyll>true</noJekyll>
		<!-- 对应github上创建的仓库名称 name -->
		<repositoryName>maven-repo</repositoryName>
		<!-- github 仓库所有者 -->
		<repositoryOwner>您的github的用户名</repositoryOwner>
	</configuration>
	<executions>
		<execution>
			<goals>
				<goal>site</goal>
			</goals>
			<phase>deploy</phase>
		</execution>
	</executions>
</plugin>
```
到此配置以及全部结束.

### 发布Maven依赖到github

再次执行 mvn clean deploy命令即可发布到github上了.

## 远程仓库Maven依赖使用

在自己的项目中使用发布的jar

### 步骤

**pom文件的**repositories**节点中添加对应仓库**

```xml
<repository>
            <id>maven-repo-master</id>
            <url>https://github.com/v4liulv/maven-repo/tree/master</url>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </snapshots>
</repository>
```
**pom文件的**dependencies**节点中添加依赖**
```xml
<repository>
            <id>maven-repo-master</id>
            <url>https://github.com/v4liulv/maven-repo/tree/master</url>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </snapshots>
</repository>
```
**End**









