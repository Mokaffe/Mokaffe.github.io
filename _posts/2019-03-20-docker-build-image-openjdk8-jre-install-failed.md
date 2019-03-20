---
layout:     post
title:      build image找不到设置的openjdk8-jre版本
subtitle:   openjdk8-jre-8.201.08-r0 bug
date:       2019-03-20
author:     Mokaffe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Docker
---

### Pipeline 红过夜的一次记录
2019年03月18日`周一`晚上提交代码，pipeline 挂了。
![error image.png](https://upload-images.jianshu.io/upload_images/5013098-68fb44957e314599.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
根据图上的信息，我们知道了Dockerfile中设置的`apk add --no-cache 'openjdk8-jre=8.191.12-r0'` 没有找到安装包，提示说可以使用` openjdk8-jre-8.201.08-r0[openjdk8-jre]` 这个版本。相应的也需要将上一步中`ENV JAVA_VERSION 8u191`设置为对应的版本。

![使用提示信息中的版本.png](https://upload-images.jianshu.io/upload_images/5013098-3b0fb627ff7feb4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
此时，按照提示更改后，成功的build image。
但是，在第二个运行测试的Job中，pipeline又挂了。
Dockerfile中的java环境就是为SonarQube设置的，现在发现Sonar不能正常运行，脑子里想到了一些问题：
- Sonar需要的Java版本必须是固定的？
- 为什么下载不到之前设置的8u191 openjdk 版本？
- 如何让pipeline正常运行通过？

![sonar-scanner 运行失败.png](https://upload-images.jianshu.io/upload_images/5013098-c40a5f49cafa8df5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Dockerfile 信息
因为项目的原因，使用的是node-alpine为基础镜像，然后安装了SonarQube进行代码分析。
安装docker-sonar-scanner Dockerfile的参考github repo  [newtmitch](https://github.com/newtmitch)/**[docker-sonar-scanner](https://github.com/newtmitch/docker-sonar-scanner)**


``` docker
FROM node:8.12.0-alpine
WORKDIR /app
RUN cd /app \
    && apk update \
    && apk add --no-cache curl unzip

# Porperty for JDK
# Default to UTF-8 file.encoding
ENV LANG C.UTF-8
RUN { \
		echo '#!/bin/sh'; \
		echo 'set -e'; \
		echo; \
		echo 'dirname "$(dirname "$(readlink -f "$(which javac || which java)")")"'; \
	} > /usr/local/bin/docker-java-home \
	&& chmod +x /usr/local/bin/docker-java-home

ENV JAVA_HOME /usr/lib/jvm/java-1.8-openjdk/jre
ENV PATH $PATH:/usr/lib/jvm/java-1.8-openjdk/jre/bin:/usr/lib/jvm/java-1.8-openjdk/bin
ENV JAVA_VERSION 8u201

RUN set -x \
	&& apk add --no-cache openjdk8-jre=8.201.08-r0 \
	&& [ "$JAVA_HOME" = "$(docker-java-home)" ]

# Install sonar-scanner
RUN curl --insecure -o ./sonarscanner.zip -L https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.0.3.778-linux.zip && \
	unzip sonarscanner.zip && \
	rm sonarscanner.zip && \
	mv sonar-scanner-3.0.3.778-linux sonar-scanner

ENV SONAR_RUNNER_HOME=/app/sonar-scanner
ENV PATH $PATH:/app/sonar-scanner/bin

COPY ./sonar-project.properties /app/sonar-scanner/conf/sonar-scanner.properties

#   ensure Sonar uses the provided Java for musl instead of a borked glibc one
RUN sed -i 's/use_embedded_jre=true/use_embedded_jre=false/g' /app/sonar-scanner/bin/sonar-scanner

```
### 解决问题的过程

###### openjdk8-jre版本问题分析
对于没有玩过linux的我，用apk 安装的包是从哪里找到的，需要搞清楚。
![image.png](https://upload-images.jianshu.io/upload_images/5013098-7ed5f1e670462748.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
由这个图直观的看到：
第一个网站是`openjdk8-jre`，显示是5天前进行了更新，已经是`8.201.08-r0`版本了，
第二个网站是`openjdk-jre-base`，看上去是还有`8.191.12-r0` 版本，但是点击进去也是`8.201.08-r0`版本了。
![image.png](https://upload-images.jianshu.io/upload_images/5013098-d4d23e7eb3917e8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在[Alpine Linux Package](https://pkgs.alpinelinux.org/packages) 里面搜索`openjdk8-jre`,发现在2019年03月14日更新为`8.201.08-r0`版本，而且找不到旧版本`8.191.12-r0`了。
所以build image的时候 pipeline在安装旧版本的openjdk8-jre的时候挂了。

![image.png](https://upload-images.jianshu.io/upload_images/5013098-f85e0b18c2c69a76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### Sonar-scanner 运行失败问题分析
根据详细的Error信息，可以知道是
```
java.lang.ExceptionInInitializerError  
Caused by: java.security.ProviderException: Could not initialize NSS
Caused by: java.io.FileNotFoundException: /usr/lib/libnss3.so
```
看到这个错误信息一脸懵逼，也不知道是为什么，最后在github上找到了一个类似的issue，是关于docker openjdk的，issue 创建时间是3天前，大概是2019年03月17日[docker-library / openjdk / issue - Missing libnss3.so after updating openjdk:8-jdk-alpine image](https://github.com/docker-library/openjdk/issues/289)
![image.png](https://upload-images.jianshu.io/upload_images/5013098-7309d8762237c99e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
issue的评论中，有人说自己手动在dockerfile中install nss可以觉得这个问题，尝试了一下，也解决了我所面临的问题，sonar-scanner可以正常运行了。
``` docker
RUN apk add --no-cache nss
```
有人直接po出了上游问题，是在Alpine Linux中提出的。
> Looks like someone filed a bug upstream about this [https://bugs.alpinelinux.org/issues/10126](https://bugs.alpinelinux.org/issues/10126)



### 思考
早上和TL说在整理这篇文章，TL说这个不是正确的解决方式，因为如果openjdk8-jre的版本又突然间本改变呢？我们又会重复面临这个问题。从长远角度考虑，应该是要选择长期稳定的jdk版本才是解决之道。


### 附上详细的Error信息：
``` java
ERROR: Error during SonarQube Scanner execution
java.lang.ExceptionInInitializerError
	at sun.security.ssl.SSLSessionImpl.<init>(SSLSessionImpl.java:188)
	at sun.security.ssl.SSLSessionImpl.<init>(SSLSessionImpl.java:152)
	at sun.security.ssl.SSLSessionImpl.<clinit>(SSLSessionImpl.java:79)
	at sun.security.ssl.SSLSocketImpl.init(SSLSocketImpl.java:598)
	at sun.security.ssl.SSLSocketImpl.<init>(SSLSocketImpl.java:566)
	at sun.security.ssl.SSLSocketFactoryImpl.createSocket(SSLSocketFactoryImpl.java:110)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.connection.RealConnection.connectTls(RealConnection.java:256)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.connection.RealConnection.establishProtocol(RealConnection.java:237)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.connection.RealConnection.connect(RealConnection.java:148)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.connection.StreamAllocation.findConnection(StreamAllocation.java:186)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.connection.StreamAllocation.findHealthyConnection(StreamAllocation.java:121)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.connection.StreamAllocation.newStream(StreamAllocation.java:100)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.connection.ConnectInterceptor.intercept(ConnectInterceptor.java:42)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.http.RealInterceptorChain.proceed(RealInterceptorChain.java:92)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.http.RealInterceptorChain.proceed(RealInterceptorChain.java:67)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.cache.CacheInterceptor.intercept(CacheInterceptor.java:93)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.http.RealInterceptorChain.proceed(RealInterceptorChain.java:92)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.http.RealInterceptorChain.proceed(RealInterceptorChain.java:67)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.http.BridgeInterceptor.intercept(BridgeInterceptor.java:93)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.http.RealInterceptorChain.proceed(RealInterceptorChain.java:92)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.http.RetryAndFollowUpInterceptor.intercept(RetryAndFollowUpInterceptor.java:120)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.http.RealInterceptorChain.proceed(RealInterceptorChain.java:92)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.internal.http.RealInterceptorChain.proceed(RealInterceptorChain.java:67)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.RealCall.getResponseWithInterceptorChain(RealCall.java:179)
	at org.sonarsource.scanner.api.internal.shaded.okhttp.RealCall.execute(RealCall.java:63)
	at org.sonarsource.scanner.api.internal.ServerConnection.callUrl(ServerConnection.java:113)
	at org.sonarsource.scanner.api.internal.ServerConnection.downloadString(ServerConnection.java:98)
	at org.sonarsource.scanner.api.internal.Jars.getBootstrapIndex(Jars.java:96)
	at org.sonarsource.scanner.api.internal.Jars.getScannerEngineFiles(Jars.java:76)
	at org.sonarsource.scanner.api.internal.Jars.download(Jars.java:70)
	at org.sonarsource.scanner.api.internal.JarDownloader.download(JarDownloader.java:39)
	at org.sonarsource.scanner.api.internal.IsolatedLauncherFactory$1.run(IsolatedLauncherFactory.java:75)
	at org.sonarsource.scanner.api.internal.IsolatedLauncherFactory$1.run(IsolatedLauncherFactory.java:71)
	at java.security.AccessController.doPrivileged(Native Method)
	at org.sonarsource.scanner.api.internal.IsolatedLauncherFactory.createLauncher(IsolatedLauncherFactory.java:71)
	at org.sonarsource.scanner.api.internal.IsolatedLauncherFactory.createLauncher(IsolatedLauncherFactory.java:67)
	at org.sonarsource.scanner.api.EmbeddedScanner.doStart(EmbeddedScanner.java:218)
	at org.sonarsource.scanner.api.EmbeddedScanner.start(EmbeddedScanner.java:156)
	at org.sonarsource.scanner.cli.Main.execute(Main.java:74)
	at org.sonarsource.scanner.cli.Main.main(Main.java:61)
Caused by: java.security.ProviderException: Could not initialize NSS
	at sun.security.pkcs11.SunPKCS11.<init>(SunPKCS11.java:223)
	at sun.security.pkcs11.SunPKCS11.<init>(SunPKCS11.java:103)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at sun.security.jca.ProviderConfig$2.run(ProviderConfig.java:224)
	at sun.security.jca.ProviderConfig$2.run(ProviderConfig.java:206)
	at java.security.AccessController.doPrivileged(Native Method)
	at sun.security.jca.ProviderConfig.doLoadProvider(ProviderConfig.java:206)
	at sun.security.jca.ProviderConfig.getProvider(ProviderConfig.java:187)
	at sun.security.jca.ProviderList.getProvider(ProviderList.java:233)
	at sun.security.jca.ProviderList.getIndex(ProviderList.java:263)
	at sun.security.jca.ProviderList.getProviderConfig(ProviderList.java:247)
	at sun.security.jca.ProviderList.getProvider(ProviderList.java:253)
	at java.security.Security.getProvider(Security.java:503)
	at sun.security.ssl.SignatureAndHashAlgorithm.<clinit>(SignatureAndHashAlgorithm.java:415)
	... 40 more
Caused by: java.io.FileNotFoundException: /usr/lib/libnss3.so
	at sun.security.pkcs11.Secmod.initialize(Secmod.java:193)
	at sun.security.pkcs11.SunPKCS11.<init>(SunPKCS11.java:218)
	... 56 more
ERROR: 
ERROR: Re-run SonarQube Scanner using the -X switch to enable full debug logging.
```
