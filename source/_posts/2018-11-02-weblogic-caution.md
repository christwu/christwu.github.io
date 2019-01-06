---
layout: post
title: Weblogic踩坑记录
category: 经验
tags:
- Weblogic
---
本文记录我在部署Weblogic时遇到的各种坑。其中JDK为1.6，Weblogic版本11g。
<!-- more -->
<style>
#post-content table {
	word-break: break-all;
}

code {
	word-break: break-all;
}
</style>

# 开发阶段
## 准备测试区
务必准备一个和生产环境架构接近的测试环境，而且开发环境、测试环境和生产环境的JDK与中间件版本应该保持一致，如果条件不够，至少测试环境和生产环境要一致。开发时用某个版本JDK和Tomcat，部署到生产环境时用另一个版本的JDK和Weblogic，这样很容易遭遇意外。

举一些例子，以下几个就属于Tomcat上面测不出来，挪到Weblogic上面就可能会暴露出来的错误，好在下表这些错误通过修改Weblogic服务器启动参数就能在一定程度上解决了：

| 场景							| 错误信息							| 启动参数
|------------------------------|----------------------------------|-----------------------------
| 生成图片						| java.awt.HeadlessException		| -Djava.awt.headless=true
| 访问HTTPS网站					| javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: No trusted certificate found exception | -DUseSunHttpHandler=true
| Apache CXF提供WebService服务	| javax.xml.ws.soap.SOAPFaultException: Cannot create a secure XMLInputFactory | -Dorg.apache.cxf.stax.allowInsecureParser=1

## 路径问题
假设程序部署在服务器的`/home/weblogic/project`中，Weblogic安装在`/u01/Oracle/Middleware`下面，而且程序会动态生成文件，实际上文件会放在类似于`/u01/Oracle/Middleware/user_projects/domains/base_domain/servers/app_server1/stage/project`的对应位置中，而且激活更新时候文件会丢失，需要重新生成。

避免使用中文文件名和中文路径，以免因字符编码问题导致部署或升级失败。举个例子，如果文件里有中文名，打包并解压之后文件名变成了乱码，那么更新的时候Weblogic会提示`Error occurred while downloading files from admin server for deployment request "xxx,xxx,xxx". Underlying error is: "null"`。

## 数据源名称
假如JNDI数据源名称为dataSource，在Tomcat中运行时，需要写成`java:comp/env/dataSource`，但是在Weblogic中运行时要把“java:comp/env/”去掉，直接写成dataSource。

## 集群Session丢失问题
生产环境通常会架设集群，通过负载均衡进行访问。如果负载均衡未按照Cookie进行分配，或者分配策略不完全正确，那么这样的话很可能会存在串Session的问题，例如登录成功之后进的是节点1，稍微做点操作后默默地跳到了节点3，导致会话丢失，系统提示重新登录。因为平时开发不会去使用负载均衡，所以可能注意不到这个问题。

这个问题可以通过以下几种方法解决：
1. 正确地配置负载均衡，保证同一会话（JSESSIONID）的流量只分配到同一节点上；
2. 使用Weblogic的“会话复制”功能（这个比较正统）；
3. 通过Redis等实现会话共享（比较复杂，而且Weblogic不是没有相关功能，不推荐）。

会话复制的操作方法可以用Google搜索。

{% note default %}
由于我们项目比较特殊，所以使用的方法和以上三种均不相同：用户认证和会话控制由集成在上游的系统管理，请求通过上游的反向代理传过来，用户信息保存在特定HTTP Header中，而我们自己项目内的用户验证在Filter中进行，一旦Session丢掉可以直接根据Header信息重建。
{% endnote %}

# 部署阶段
## 主机名与hosts
给服务器设置一个固定IP和固定的主机名，然后将服务器的IP与主机名加入到`/etc/hosts`中。对于Oracle厂的产品，即便后续设置用不到主机名，设置好之后也可以避免一些不必要的麻烦。

## 防火墙配置
默认情况下，管理控制台的端口是7001，应用是7003，节点管理器是5556，初次部署时需要注意让防火墙放行这三个端口。

为了安全，需要仔细控制防火墙的放行范围，不要让7001和5556暴露到互联网上面。另外不要把应用部署到AdminServer上面，否则封锁7001端口之后应用就无法访问了。

## 建议进行网络测试
在开始部署之前，建议在应用服务器上测试一下能否访问数据库等资源。如果网络不通，那么改Weblogic设置时可能会无响应，卡了半天之后才蹦出一句`The Network adapter could not establish the connection`，白白耽误时间。

如果系统没装telnet，可以简单地测试一下能不能ping通，或者通过以下Java程序来测一下端口通不通（有些机房可能会出现IP能ping通但是端口被拦截的情况）：

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.Socket;

public class IpTest {
    public static void main(String[] args){
        Socket connect = new Socket();
        try {
            String ip = "";
            int port = 0;
            int timeout = 100;

            if (args.length < 2) {
                System.out.println("用法：java IpTest <IP地址> <端口号> [超时时间]");
                return;
            } else {
                ip = args[0];
                port = Integer.parseInt(args[1]);
            }
            if (args.length > 2){
                timeout = Integer.parseInt(args[2]);
            }
            System.out.println("正在连接 " + ip + ":" + port);

            InetSocketAddress addr = new InetSocketAddress(ip, port);
            connect.connect(addr, timeout);

            if (connect.isConnected()) {
                System.out.println("连接成功");
            } else {
                System.out.println("连接失败");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                connect.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

使用方法：

```
javac IpTest.java
java IpTest 10.15.2.9 1521
```

## 修改java.security
Java 6存在一个关于随机数的bug，如果不Hack，在Linux系统下面Weblogic建域和启动时需要等待很长时间，因此建议装完Java之后立刻去修改java.security。

假设$JAVA_HOME为`/opt/jdk1.6.0_145`，也就是说JDK装在了这个地方，那么需要修改`$JAVA_HOME/jre/lib/security/java.security`文件，找到

```
securerandom.source=file:/dev/urandom
```

修改成

```
securerandom.source=file:/dev/./urandom
```

之后建域和起停之类操作就不需要再等待十多分钟了。

## 如果通过控制台启动服务器时起不来
如果Weblogic各节点已正确设置，各服务器的防火墙已经开放5556端口，NodeManager也已经启动，但是仍然无法通过控制台启动节点，提示“不兼容的状态”，而且在startNodeManager.sh的输出中出现`javax.net.ssl.SSLKeyException: [Security:090482]BAD_CERTIFICATE alert was received from ...`：

### 方法一：重新设置证书
在集群的每台服务器上面设置一下证书（如果执行第一行命令就报错的话，请把$WL_HOME换成实际安装目录）：

```bash
. $WL_HOME/server/bin/setWLSEnv.sh
cd $WL_HOME/server/lib
java utils.CertGen -keyfilepass DemoIdentityPassPhrase -certfile newcert -keyfile newkey
java utils.ImportPrivateKey -keystore DemoIdentity.jks -storepass DemoIdentityKeyStorePassPhrase -keyfile newkey.pem -keyfilepass DemoIdentityPassPhrase -certfile newcert.pem -alias demoidentity
```

### 方法二：不使用SSL
除此以外，还有一种做法是不使用SSL通信以避免出现证书问题：修改`$WL_HOME/common/nodemanager/nodemanager.properties`文件，将其中的

```
SecureListener = true
```

改成

```
SecureListener = false
```

然后重新启动NodeManager。另外在控制台上建立“计算机”时，类型就不再是“SSL”，而是“普通”（Plain）。

## 组建集群
我们并不需要每一台机器都执行一遍建域之类的操作。只要各服务器JDK路径和Weblogic安装路径一致，那么在一台机器上面把Weblogic的各种参数都配置好，然后将user_projects目录打包（或者干脆把整个Middleware打包），复制到其他各服务器上面解压就差不多了。

如果NodeManager使用SSL，解压完成后，前文提到的“NodeManager证书”还是要在每台服务器上操作一遍。

### （待研究）服务器监听地址不要贸然写成0.0.0.0
虽然0.0.0.0也是一个有效的监听地址，但是在组建集群时不要贸然地写成0.0.0.0，否则AdminServer与各节点之间的通信会出现问题，例如服务器状态变成UNKNOWN，或者部署应用无响应。具体原因和解决方法待研究。

## 增加线程数
如果预计并发数比较高，可以增加应用线程池的线程数。修改config.xml配置文件（例如`/u01/Oracle/Middleware/user_projects/base_domain/config/config.xml`）。假如应用服务器叫app_server1，那么需要找到类似以下的代码

```xml
<server>
    <name>app_server1</name>
    ...
```

修改成

```xml
<server>
    <name>app_server1</name>
    <self-tuning-thread-pool-size-min>1000</self-tuning-thread-pool-size-min>
    <self-tuning-thread-pool-size-max>1000</self-tuning-thread-pool-size-max>
    ...
```

需要注意的是，如果数值改得太大，启动时可能会出现以下错误：

```
[ERROR][thread ] Could not start thread [ACTIVE] ExecuteThread: '977' for queue: 'weblogic.kernel.Default (self-tuning)'. Resource temporarily unavailable


Attempting to allocate 8192M bytes

There is insufficient native memory for the Java
Runtime Environment to continue.

Possible reasons:
  The system is out of physical RAM or swap space
  In 32 bit mode, the process size limit was hit

Possible solutions:
  Reduce memory load on the system
  Increase physical memory or swap space
  Check if swap backing store is full
  Use 64 bit Java on a 64 bit OS
  Decrease Java heap size (-Xmx/-Xms)
  Decrease number of Java threads
  Decrease Java thread stack sizes (-Xss)
  Disable compressed references (-XXcompressedRefs=false)

java.lang.OutOfMemoryError: Resource temporarily unavailable in tsStartJavaThread (lifecycle.c:1097).
```

报错信息看起来像是内存不足，实际上不是内存不足，而是Linux系统默认情况下限制了最大线程数，需要进行修改。

如果使用CentOS 6或RHEL6，可以按照以下方法进行修改：

首先修改/etc/security/limits.conf，加入
```
* hard nofile 1048576
* soft nofile 1048576
```

然后修改/etc/security/limits.d/90-nproc.conf，加入
```
weblogic   soft    nproc     655350
```

最后修改/etc/pam.d/login，加入
```
session required /lib64/security/pam_limits.so
```

修改完成后不要马上退出shell，要开个新窗口测试各账号能否登进去。如果登不进去（例如提示could not open session），说明参数改得太大，需要适当往小调。

另外如果线程数改大了，内存也应当适当调大，因为每个线程都会占一些内存空间。

# 运维阶段

## 应用升级
改完文件之后要在控制台的“部署”里面进行更新，否则内容不会生效。改静态文件也是。

每次更新的时候，JVM会把Class信息保存到内存的永久保留区域中，而旧的内容不会释放。如果Weblogic启动参数中的-XX:MaxPermSize比较小，那么更新几次可能就会卡死挂掉，而且应用日志会显示`java.lang.OutOfMemoryError: PermGen space`。在这种情况下，把Weblogic里面的服务器停掉然后再启动一次就好了。

在开始更新到更新结束，应用会出现短暂的中断，因此要注意选择合适的时间进行操作。另外在业务繁忙时进行更新，不但会影响用户，而且容易因为Weblogic繁忙而导致更新无响应或失败。

建议编写一份升级检查表，将准备工作、实施工作和收尾工作全部列入，以免因漏项导致升级出现问题。

如果更新时遭遇`Error occurred while downloading files from admin server for deployment request "xxx,xxx,xxx". Underlying error is: "null"`，那么需要去检查AdminServer.log的日志（例如`/u01/Oracle/Middleware/user_projects/base_domain/servers/AdminServer/logs/AdminServer.log`）。我碰到过两种会出现这种错误的情况，一种是文件名乱码，另一种是文件权限不正确（例如服务以weblogic用户运行，但是文件所有者是root，weblogic访问不了）。假如权限不正确，可以`chown -R weblogic:weblogic 存放程序的目录`。

我们部分应用服务器安装了防篡改软件，如果升级之前忘记关闭防护功能，在控制台上激活时会报错，提示无法“remove staged files”。

{% note info %}
建议在系统中加入可配置参数的系统公告功能，这样在升级或者维护之前可以通知用户，使用户及时做好准备。
{% endnote %}

## 重启服务器
如果只是重启应用节点，操作就比较简单，直接在控制台上操作即可。不过，在Weblogic控制台重启服务的时候不要点完“启动”就不管了，一定要等到状态显示为“RUNNING”之后再收工。如果变成“ADMIN”，那么可能是有报错，需要去应用服务器日志检查一下。若确认不耽误事（例如`java.lang.NoClassDefFoundError: com/ctc/wstx/stax/WstxInputFactory`）那么在控制台点一下“恢复”就好了。

如果不慎关掉AdminServer，或者打算整个重启，那么操作会比较麻烦，大体上要按以下思路操作：
1. 在管理节点上启动AdminServer（startWebLogic.sh）。
2. 在每个节点上都启动一遍NodeManager（startNodeManager.sh）。
3. 进入管理控制台，启动各应用节点。
4. 启动完成后确认“部署”里面各应用是否启动。

## 打补丁
安装补丁之前，需要检查bsu.sh文件（例如`/u01/Oracle/Middleware/utils/bsu/bsu.sh`），将其中的最大内存Xmx改大些，例如-Xmx2048m。因为打补丁实在太慢，等了半天出来的却是java.lang.OutOfMemoryError的话实在太憋屈。

打补丁对业务影响不大，可以随时操作，但是打完之后需要重启Weblogic才能生效。

## 搬家
尽量不要搬家，因为Weblogic安装和建域之后会产生很多已经写好了路径的配置文件。即使将它们全部改成新路径，Middleware目录中还有个registry.dat，此文件记录了Weblogic的安装情况，而且已经加密，如果贸然搬家会在升级等方面遇到麻烦。实在需要的话还是建立软链接比较好。

## 监控
建议在服务器上面安装专门的监控软件。像我们项目组那种服务器由用户管理，应用由项目组自行运维的情况，更要有自己的监控程序，这样在出现问题时也好及时定位，以免一塌糊涂。

目前我们项目组自己写了一个监控脚本，在繁忙时期每隔10分钟、空闲时期每隔半小时把Weblogic控制台的一些常用指标记录到日志文件中（通过cron控制），[代码见此](https://github.com/infnan/scripts/blob/master/部署和运维/weblogic-monitor.sh)。
