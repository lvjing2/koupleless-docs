---
title: 常见问题列表
date: 2024-01-25T10:28:32+08:00
weight: 10
---

## 使用问题

### arklet 安装问题
#### 现象

通过 go install 无法安装 arkctl，执行命令如：

```shell
go install koupleless.alipay.com/koupleless/v1/arkctl@latest
```

报错信息如下：

```text
go: koupleless.alipay.com/koupleless/v1/arkctl@latest: module koupleless.alipay.com/koupleless/v1/arkctl: Get "https://proxy.golang.org/koupleless.alipay.com/koupleless/v1/arkctl/@v/list": dial tcp 142.251.42.241:443: i/o timeout
```

#### 解决方式

arkctl 是作为 koupleless 子目录的方式存在的，所以没法直接 go get，可以从这下面下载执行文件, 请参考[安装 arkctl](https://github.com/koupleless/arkctl/releases/tag/arkctl-release-0.2.0)

## 模块构建问题

### maven 版本太低
#### 现象
构建时，

- 报错 Unable to parse configuration of mojo com.alipay.sofa:sofa-ark-maven-plugin:.*:repackage for parameter excludeArtifactIds
- 报错 com.google.inject.ProvisionException: Unable to provision, see the following errors:
- 报错 Error injecting: private org.eclipse.aether.spi.log.Logger org.apache.maven.repository.internal.DefaultVersionRangeResolver.logger
- 报错 Caused by: java.lang.IllegalArgumentException: Can not set org.eclipse.aether.spi.log.Logger field org.apache.maven.repository.internal.DefaultVersionRangeResolver.logger to org.eclipse.aether.internal.impl.slf4j.Slf4jLoggerFactory

#### 原因
maven 版本太低

#### 解决方式
需升级到 3.6.1 及以上

## 配置问题
### application.properties 配置
#### 现象
spring.application.name must be configured

#### 原因
application.properties 里未配置 spring.application.name

#### 解决方式
application.properties 里配置 spring.application.name

### SOFABoot 基座或模块启动因 AutoConfiguration 失败
#### 现象
报错 The following classes could not be excluded because they are not auto-configuration classes: org.springframework.boot.actuate.autoconfigure.startup.StartupEndpointAutoConfiguration

#### 原因
SOFABoot 正确引入需要同时引入 spring-boot-actuator-autoconfiguration，因为 [sofa-boot 里通过代码定义](https://github.com/sofastack/sofa-boot/blob/82d0ca388b433ac18fb44704e2f2b280fda1b760/sofa-boot-project/sofa-boot/src/main/java/com/alipay/sofa/boot/env/SofaBootEnvironmentPostProcessor.java#L88)了 spring.exclude.autoconfiguration = `org.springframework.boot.actuate.autoconfigure.startup.StartupEndpointAutoConfiguration`, 当启动时找不到该类时就会报错。

#### 解决方式
基座或模块里引入 sprign-boot-actuator-autoconfiguration

### 模块发rest服务webContextPath冲突
#### 现象
报错 org.springframework.context.ApplicationContextException: Unable to start web server; nested exception is java.lang.IllegalArgumentException: Child name xxx is not unique

#### 原因
webContextPath 冲突

#### 解决方式
检查其他模块是否设置相同的webContextPath

### jvm参数配置错误
#### 现象
报错 Error occurred during initialization of VM

#### 原因
报错 Error occurred during initialization of VM，一般是jvm参数配置有问题

#### 解决方式
用户侧检查 jvm 参数配置

## 运行时问题
### koupleless 依赖缺失
#### 现象
- 模块安装时，报错 com.alipay.sofa.ark.exception.ArkLoaderException: \[ArkBiz Loader\] module1:1.0-SNAPSHOT : can not load class: com.alipay.sofa.koupleless.common.spring.KouplelessApplicationListener

#### 原因
koupleless 依赖缺失

#### 解决方式
 请在模块里面添加如下依赖：
```xml
<dependency>
    <groupId>com.alipay.sofa.koupleless</groupId>
    <artifactId>koupleless-app-starter</artifactId>
    <version>${koupleless.runtime.version}</version>
</dependency>
```
或者升级 koupleless 版本到最新版本

### koupleless 版本较低
#### 现象
- 模块安装报错 Master biz environment is null
- 模块静态合并部署无法从指定的目录里找到模块包

#### 解决方式
升级 koupleless 版本到最新版本
```xml
<dependency>
    <groupId>com.alipay.sofa.koupleless</groupId>
    <artifactId>koupleless-app-starter</artifactId>
    <version>${最新版本号}</version>
</dependency>
```

### 类缺失
#### 现象

- 报错 java.lang.ClassNotFoundException
- 报错 java.lang.NoClassDefFoundError

#### 原因
模块/基座 找不到类

#### 解决方式
请根据 模块类缺失 和 基座类缺失 进行排查。
### 模块类缺失
#### 现象
ArkBiz Loader.*can not load class

#### 原因
模块里缺失对应类的依赖!

#### 解决方式
检查模块里是否包含该类的依赖，如果没有，请增加相应的依赖。
### 基座类缺失
#### 现象
ArkLoaderException: Post find class XXX occurs an error via biz ClassLoaderHook

#### 原因
设置委托给基座加载的类，但基座里找不到对应类的依赖，或者依赖的版本不对。

#### 解决方式
基座添加相应依赖，或者修改依赖版本。
### 模块依赖的类存在多个不同的来源
#### 现象

- 报错 java.lang.LinkageError
- 报错 java.lang.ClassCastException
- 报错 previously initiated loading for a different type with name

#### 原因
该类在基座和模块间引入了多个相同的依赖，加载到的类可能来自于不同 ClassLoader

#### 解决方式
在模块主 pom 中，把类所在的包设置为 provided。（做好模块瘦身和基座与模块间的依赖管理。）
### 方法缺失
#### 现象
java.lang.NoSuchMethodError

#### 原因
报出java.lang.NoSuchMethodError，可能存在存在jar包冲突、依赖未加载的情况

#### 解决方式
请检查是否存在jar包冲突或依赖未加载。
### 模块直接使用基座数据源
#### 现象
No operation is allowed after dataSource is closed

#### 原因
模块里直接使用了基座里的 dataSource，模块卸载导致基座 dataSource 关闭

#### 解决方式
dataSource 已经关闭，确认模块里是否通过 bean 获取方式直接使用了基座里的 dataSource

### Bean 配置问题
#### 现象

- 报错 org.springframework.beans.factory.parsing.BeanDefinitionParsingException: Configuration problem: Invalid bean definition with name
- 报错 java.lang.IllegalArgumentException: JVM Reference
- 报错 Error creating bean with name
- 报错 BeanDefinitionStoreException: Invalid bean definition with name
- 报错 org.springframework.beans.FatalBeanException: Bean xx has more than one interface
- 报错 No qualifying bean of type
- 报错 BeanDefinitionStoreException: Invalid bean definition with name

#### 原因
工程中 bean 配置问题

#### 解决方式

1. xml中是否错误配置了 bean, 或者依赖存在问题 2. bean初始化/Bean定义异常, 请检查业务逻辑

### Spring Bean 重复定义
#### 现象
There is already xxx bean

#### 原因
业务编码问题：bean 重复定义

#### 解决方式
检查业务侧代码
### XML 配置问题
#### 现象
Error parsing XPath XXX Cause: java.io.IOException: Could not find resource

#### 原因
XML文件解析失败, 无法找到对应的依赖配置

#### 解决方式
请用户排查解析失败问题
### JMX 配置问题
#### 现象
org.springframework.jmx.export.UnableToRegisterMBeanException: Unable to register MBean

#### 原因
JMX 需要手动配置应用名称

#### 解决方式
给基座加 -Dspring.jmx.default-domain=${spring.application.name} 启动参数
### 依赖配置
#### 现象
Dependency satisfaction failed XXX java.lang.NoClassDefFoundError

#### 原因
jar包依赖问题, class not found

#### 解决方式
请检查 jar 包依赖, 工程中是否依赖了错误 jar 包, 进行修正
### SOFA JVM Service 查找失败
#### 现象

- 报错 can not find corresponding jvm service
- 报错 JVM Reference XXX can not find the corresponding JVM service

#### 原因
JVM Reference 依赖的JVM 服务未找到

#### 解决方式
请检查业务代码是否正确，对应的服务是否存在。
### 内存不足
#### 现象

- 报错 Insufficient space for shared memory
- 报错 java.lang.OutOfMemoryError: Metaspace

#### 原因
内存不足或内存溢出

#### 解决方式
请替换或重启机器
### hessian版本冲突
#### 现象
Illegal object reference

#### 原因
hessian版本冲突

#### 解决方式
使用mvn dependency:tree查看依赖树，排掉冲突依赖

### guice 版本较低
现象 报错 Caused by: java.Lang.ClassNotFoundException: com.google.inject.multibindings.Multibinder

![guice_version_incompatibility.png](/docs/faq/imgs/guice_version_incompatibility.png)

#### 原因
用户工程与 koupleless 里 guice 版本不一致，且版本较老

#### 解决方式
升级 guice 版本到较新版本，如

```xml
<dependency>
    <groupId>com.google.inject</groupId>
    <artifactId>guice</artifactId>
    <version>6.0.0</version>
</dependency>
```

### SOFABoot 健康检查失败
#### 现象
HealthCheck Failed

#### 原因
Sofaboot工程HealthCheck失败

#### 解决方式
请用户排查具体失败原因

### 需要模块瘦身
#### 现象
- 报错 java.lang.IllegalArgumentException: Cannot instantiate interface org.springframework.context.ApplicationListener : com.alipay.sofa.koupleless.common.spring.KouplelessApplicationListener

#### 原因
模块应该以provided方式引入 springboot 依赖

#### 解决方式
模块做好瘦身，参考这里：[模块瘦身](/docs/tutorials/module-development/module-slimming)

### 模块与基座共库时，模块启动了基座的逻辑
#### 现象 
例如基座引入了 druid，但是模块里没有引入，按照设计模块应该不需要初始化 dataSource，但是如果遇到模块也初始化了 dataSource，那么该行为是不符合预期的，也可能导致报错。

#### 解决方式

1. 确保模块可以独立构建，也就是可以在模块的目录里执行 `mvn clean package`，并且不会报错
2. 升级 koupleless 版本到最新版本 1.0.0

### 模块启动无法初始化 EnvironmentPostProcessor
#### 现象
模块启动时报这类错误信息 `Unable to instantiate com.baomidou.mybatisplus.autoconfigure.SafetyEncryptProcessor [org.springframework.boot.en
V.EnvironmentPostProcessor]`

#### 解决方式
模块main方法里启动 springboot 时指定 ResourceLoader 的 ClassLoader
```java
SpringApplicationBuilder builder = new SpringApplicationBuilder(Biz1Application.class);
// set biz to use resource loader.
ResourceLoader resourceLoader = new DefaultResourceLoader(
    Biz1Application.class.getClassLoader());
builder.resourceLoader(resourceLoader);
builder.build().run(args);
```

### 基座关闭的时候Tomcat报错

#### 现象
基座关闭时，报错：Unable to stop embedded Tomcat

#### 原因
基座关闭时，Tomcat 有自身关闭逻辑， koupleless 增加了关闭逻辑，导致基座关闭会尝试第二次关闭。该信息只是一个警告，不影响基座的正常关闭。

#### 解决方式
无需处理
