---
layout: post
title:  "Maven Central Publish Grdle Plugin"
date:   2024-12-21 19:01:00 +0800
---

## 1 maven & sonatype

Maven是Apache主导和维护的一个工具：[maven.apache.org](https://maven.apache.org/)。

>
Apache Maven is a software project management and comprehension tool. Based on the concept of a project object model (POM), Maven can manage a project's build, reporting and documentation from a central piece of information.

[Maven Central Repository](https://central.sonatype.com/)是由[Sonatype](https://www.sonatype.com/)提供的、向开发者开放的、免费的Maven Repository。

>
The Central Repository is provided by Sonatype to the public as a community service. There is no charge to use the Central Repository, and Sonatype pays all the costs for hosting, bandwidth, etc. The components made available for download via Central are licensed to users by the developers who posted them and are subject to the terms and conditions of the applicable licenses accompanying such components.

## 2 Maven Central Repository的Publish文档

### [Central Portal](https://central.sonatype.org/register/central-portal/)

- 将构建publish到Maven Central Repository，需要先有一个账号。在[Center Portal](https://central.sonatype.com/)注册账号可以使用邮箱、Google账号、Github账号。

- 注册成功后，需要在Central Protal的操作页面中[生成User Token](https://central.sonatype.org/publish/generate-portal-token/)，它们将在上传构件的过程中作为用户在Maven Central Repository的**用户名、密码**。

### [Namespace](https://central.sonatype.org/register/namespace/)

- 在Maven的生态中，Namespace就是构件依赖坐标三个组成部分中的groupId。其格式与Java报名相同，如com.example。

- Namespace其实是对一个域名的翻转，如com.example是对域名example.com的翻转。如果添加一个com.example的Namespace，则需要通过与域名example.com相关的DNS Text记录文件进行**验证**（用户对域名的占有情况）。

- 如果使用Github账号进行账号注册，则将自动生成一个**已通过验证**的Namespace：io.github.<github username>。

>
As part of your GitHub subscription, GitHub provides you with a github.io domain that reflects your username and allows you to publish GitHub Pages under that domain. Because of this, Sonatype can, in most cases, automatically verify and provision publishing access to a namespace that looks like io.github.<your GitHub username>.

### [Why do we have Requirements](https://central.sonatype.org/publish/requirements/)

对于上传到Maven Central Repository的构件，要求其包含以下内容：

- sources.jar、javadoc.jar：便于构件的用户更高效地使用它。
  
  ```
  example-application-1.4.7.pom
  example-application-1.4.7.jar
  example-application-1.4.7-sources.jar
  example-application-1.4.7-javadoc.jar
  ```

- .md5、.sha1：各个文件内容的校验和，前者基于MD5取得，后者在前者的基础再进行加密。它们用于保证文件内容不被篡改。
  
  ```
  example-application-1.4.7.pom.md5
  example-application-1.4.7.pom.sha1
  example-application-1.4.7.jar.md5
  example-application-1.4.7.jar.sha1
  example-application-1.4.7-sources.jar.md5
  example-application-1.4.7-sources.jar.sha1
  example-application-1.4.7-javadoc.jar.md5
  example-application-1.4.7-javadoc.jar.sha1
  ```

- .asc：通过GPG对文件进行签名。它们用于保证构件的来源确系构件的发布者。
  
  ```
  example-application-1.4.7.pom.asc
  example-application-1.4.7.jar.asc
  example-application-1.4.7-sources.jar.asc
  example-application-1.4.7-javadoc.jar.asc
  ```

pom文件的需要包含的内容：

- groupId、artifactId、version：构件坐标。

- name、description、url：构件信息。

- licenses：开源许可信息。

- developers：开发者信息。

- scm：源码信息。

### [GPG](https://central.sonatype.org/publish/requirements/gpg/)

[GPG](https://gnupg.org/index.html)是一个免费的、用于加密和签名的工具。

>
GnuPG is a complete and free implementation of the OpenPGP standard as defined by RFC4880 (also known as PGP). GnuPG allows you to encrypt and sign your data and communications; it features a versatile key management system, along with access modules for all kinds of public key directories. GnuPG, also known as GPG, is a command line tool with features for easy integration with other applications.

[下载](https://gnupg.org/download/index.html)、安装并配置环境变量后，使用下列cmd命令进行操作：

- 查看版本以验证可用性：
  
  ```
  gpg --version
  ```

- 生成密钥对：
  
  ```
  gpg --gen-key
  ```
  
  在此过程中，需要输入用户名、邮箱和**密码**。执行成功后显示：
  
  ```
  pub   rsa3072 2021-06-23 [SC] [expires: 2023-06-23]
        CA925CD6C9E8D064FF05B4728190C4130ABA0F98
  uid                      Central Repo Test <central@example.com>
  sub   rsa3072 2021-06-23 [E] [expires: 2023-06-23]
  ```
  
  pub的第二行即是密钥对的标识，在使用密钥时，以其后八位（30ABA0F9）作为其**id**。

- 上传公钥，进而其他用户可以取得公钥并对.asc进行解密：
  
  ```
  gpg --keyserver keyserver.ubuntu.com --send-keys CA925CD6C9E8D064FF05B4728190C4130ABA0F98
  ```

- 导出私钥，后续上传构件的过程进行签名，进而生成.asc：
  
  ```
  gpg --export-secret-keys --armor CA925CD6C9E8D064FF05B4728190C4130ABA0F98
  ```
  
  导出的**私钥文件**位于：~/Users/xx/.gnupg/private-keys.asc。

### [Immutability](https://central.sonatype.org/publish/requirements/immutability/)

发布到Maven Central Repository的构件不允许删除、修改。

>
In order to provide reliable access to open source components we do not remove or modify components once they are publicly available.

### [Publishing via the Central Protal](https://central.sonatype.org/publish-ea/publish-ea-guide/)

- 在将构件上传到Maven Central Repository后，Center Protal会对构件进行验证。亦即，查看构件的内容是否满足要求。

- 构件验证成功后，即可进行发布。

### [Gradle](https://central.sonatype.org/publish/publish-portal-gradle/)

目前，还没有Gradle官方提供的上传构件至Maven Central Repository的插件。不过，有一些社区实现版。

### [Publishing via OSSRH](https://central.sonatype.org/publish/publish-guide/)

OSSRH用于支持构件的阶段性发布。

## 3 Gradle Maven Publish Plugin

[Gradle Maven Publish Plugin](https://vanniktech.github.io/gradle-maven-publish-plugin/central/)是上传构件至Maven Central Repository的Gradle插件社区实现版之一。

该插件的API对Maven Central Repository中部分实体的对应关系如下：

- MavenCentral的几个地址(定义在SonatypeHost)中，DEFAULT是旧版MavenCentral，S01是OSSRH，CENTRAL_PORTAL即是 Central Portal。
  
  ```
  mavenPublishing {
    publishToMavenCentral(SonatypeHost.DEFAULT)
    // or when publishing to https://s01.oss.sonatype.org
    publishToMavenCentral(SonatypeHost.S01)
    // or when publishing to https://central.sonatype.com/
    publishToMavenCentral(SonatypeHost.CENTRAL_PORTAL)
  
    signAllPublications()
  }
  ```

- gradle.properties文件中各个属性的含义：
  
  ```
  # Center Protal中User Token的用户名、密码（而不是Central Portal的登录用户名、密码）
  mavenCentralUsername=username
  mavenCentralPassword=the_password
  
  # GPG生成的密钥对的标识(末8位)与密码
  signing.keyId=12345678
  signing.password=some_password
  # GPG生成的私钥文件路径(.asc和.gpg文件均可)
  signing.secretKeyRingFile=/Users/yourusername/.gnupg/secring.gpg
  ```