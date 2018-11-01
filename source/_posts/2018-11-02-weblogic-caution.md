---
layout: post
title: Weblogic踩坑记录
category: 经验
tags:
- Weblogic
- 坑
---
本文记录我在部署Weblogic时遇到的各种坑。其中JDK为1.6，Weblogic版本10.3.6。
<!-- more -->
<style>
#post-content table {
	word-break: break-all;
}
</style>

# 开发
## 保持版本一致
开发环境、测试环境和生产环境的JDK与中间件版本应该保持一致，至少测试环境和生产环境要一致。开发时用某个版本JDK和Tomcat，部署到生产环境时用另一个版本的JDK和Weblogic，这样很容易遭遇意外。

## 路径
假设程序部署在服务器的/home/weblogic/project中，Weblogic安装在/u01/Oracle/Middleware下面，而且程序会动态生成文件，实际上文件会放在类似于/u01/Oracle/Middleware/user_projects/domains/base_domain/servers/app_server1/stage/project的对应位置中。

## 数据源
假如JNDI数据源名称为dataSource，在Tomcat中运行时，需要写成java:comp/env/dataSource，但是在Weblogic中运行时要把“java:comp/env/”去掉，直接写成dataSource。

## Session
生产环境通常会架设集群，通过负载均衡进行访问，这样的话很可能存在串Session的问题，例如登录成功之后稍微做点操作会话就丢失了。因为平时开发不会去使用负载均衡，所以可能注意不到这个问题。

部署时要注意，要么在Weblogic上面配置共享Session，要么注意负载均衡策略，同一会话时应当将流量分配到同一服务器上面。

# 部署
## 防火墙
默认情况下，管理控制台的端口是7001，应用是7003，节点管理器是5556，初次部署时需要注意让防火墙放行这三个端口。

为了安全，需要仔细控制防火墙的放行范围，不要让7001和5556暴露到互联网上面。另外不要把应用部署到AdminServer上面，否则封锁7001端口之后应用就无法访问了。

## 修改java.security
Java 6存在一个关于随机数的bug，如果不Hack，Weblogic建域和启动时需要等待很长时间，因此建议装完Java之后立刻去修改java.security。

假设JDK安装在/opt/jdk1.6.0_145下面，则需要修改/opt/jdk1.6.0_145/jre/lib/security/java.security文件，找到

```
securerandom.source=file:/dev/urandom
```

修改成

```
securerandom.source=file:/dev/./urandom
```

之后建域和起停之类操作就不需要再等待十多分钟了。

## 无法通过控制台启动服务器
如果Weblogic各节点已正确设置，各服务器的防火墙已经开放5556端口，Nodemanager也已经启动，但是仍然无法通过控制台启动节点，提示“不兼容的状态”，而且在Nodemanager的输出中出现“javax.net.ssl.SSLKeyException: [Security:090482]BAD_CERTIFICATE alert was received from ...”，那么需要在每个服务器上面设置一下证书：

```bash
. $WL_HOME/server/bin/setWLSEnv.sh
cd $WL_HOME/server/lib
java utils.CertGen -keyfilepass DemoIdentityPassPhrase -certfile newcert -keyfile newkey
java utils.ImportPrivateKey -keystore DemoIdentity.jks -storepass DemoIdentityKeyStorePassPhrase -keyfile newkey.pem -keyfilepass DemoIdentityPassPhrase -certfile newcert.pem -alias demoidentity
```

## 组建集群
我们并不需要每一台机器都执行一遍建域之类的操作。只要在一台机器上面把Weblogic的各种参数都配置好，然后将Middleware目录打包，复制到其他各服务器上面解压就差不多了。

解压完成后，前文提到的“Nodemanager证书”还是要在每台服务器上操作一遍。

## 一些报错与处理
以下问题可以通过修改Weblogic启动参数解决。

| 场景							| 错误信息							| 启动参数
|------------------------------|----------------------------------|-----------------------------
| 生成图片						| java.awt.HeadlessException		| -Djava.awt.headless=true
| 访问HTTPS网站					| javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: No trusted certificate found exception | -DUseSunHttpHandler=true
| Apache CXF提供WebService服务	| javax.xml.ws.soap.SOAPFaultException: Cannot create a secure XMLInputFactory | -Dorg.apache.cxf.stax.allowInsecureParser=1

# 运维

## 升级
改完文件之后要在控制台的“部署”里面进行更新，否则内容不会生效。改静态文件也是。

每次更新的时候，JVM会把Class信息保存到内存的永久保留区域中。如果Weblogic启动参数中的-XX:MaxPermSize比较小，那么更新几次可能就会卡死挂掉，而且应用日志会显示“java.lang.OutOfMemoryError: PermGen space”。在这种情况下，把Weblogic里面的服务器停掉然后再启动一次就好了。

在开始更新到更新结束，应用会出现短暂的中断，因此要注意选择合适的时间进行操作。

## 重启
在Weblogic控制台重启服务的时候不要点完“启动”就不管了，一定要等到状态显示为“RUNNING”之后再闪人。如果变成“ADMIN”，那么可能是有报错，若确认不耽误事那么在控制台点一下“恢复”就好了。

## 打补丁
安装补丁之前，需要检查bsu.sh文件（例如/opt/Oracle/Middleware/utils/bsu/bsu.sh），将其中的最大内存Xmx改大些，例如-Xmx2048m，否则打补丁时可能会报java.lang.OutOfMemoryError，耽误时间。

打补丁可以随时操作，但是打完之后需要重启Weblogic才能生效。

## 搬家
尽量不要搬家，因为Weblogic安装和建域之后会产生很多已经写好了路径的配置文件。即使将它们全部改成新路径，Middleware目录中还有个registry.dat，此文件记录了Weblogic的安装情况而且已经加密，如果贸然搬家会在升级等方面遇到麻烦。实在需要的话还是建立软链接比较好。
