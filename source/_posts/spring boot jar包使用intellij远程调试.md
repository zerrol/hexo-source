---
title: spring boot jar包使用intellij远程调试
categories: Java
tag: Java
---

在开发中难免会遇到需要对已打包好的jar包进行远程调试的工作，我所遇到的形式是因为需要以来特定的shell配置来运行java的代码，因此需要将jar包在特定的shell中运行，然后还需要通过intellij进行调试。面对这种情况其实可以很简单的处理，大致思路是将需要进行调试的代码打包成jar包后，可以在运行该jar包时暴露出一个接口以供idea远程接入就可以完成调试了。

## 配置jar包    
如果并不是使用的sping boot打包的jar包，调试前需要先修改容器的配置，例如Tomcat的话需要先接入容器的jvm
```shell
// bin\startup.bat（.sh）文件，在里面添加
 
// windows
set CATALINA_OPTS="-agentlib:jdwp=transport=dt_socket,address=8888（自定义调试端口）,server=y,suspend=n %CATALINA_OPTS%"
 
// linux
export CATALINA_OPTS="-agentlib:jdwp=transport=dt_socket,address= 8888（自定义调试端口）,server=y,suspend=n $CATALINA_OPTS"
```
如果使用了spring boot的话就相对简单一些，只需要运行jar包是添加一些配置项就可以了。由于我的项目使用了maven进行管理，所以先使用maven打包成jar包。
```shell
mvn clean package
```
```shell
// 在使用java指令启动程序时需要附加额外的参数以开启外部调试，如下
-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8888（自定义调试端口）
```
上面的代码就是指打开debug模式，打开8888端口，用作调试使用。完成命令如下：
```shell 
// 完整的启动指令是类似下面酱的
java -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8888（自定义调试端口） -jar application.jar
```
## 配置intellij
使用上面命令运行jar包，下面要做的就是配置intellij接入8888端口来进行调试，首先是我们需要先点击`Edit Configurations`。

![Edit Configurations](/img/17-6/20-2.png)

这是idea会打开configurations窗口，我按照图片的顺序来进行设置。
先点击`+`号添加一个新的`remote`

![add remote](/img/17-6/20-3.png)

添加好新的`remote`以后我们将它打开，点击配置里面的`Unnamed`。如图，因为我们是对本地的jar包进行远程调试，所以只需要修改`port`即可，将port改为与jar包暴露的端口一致。

![config remote](/img/17-6/20-4.png)

修改完毕点击apply和OK，完成配置。

![set](/img/17-6/20-5.png)

如图将intellij的运行项目改为刚刚设置的`remote`（一般情况下，会自动变为这个）。然后点击debug按钮就可以进行远程调试了。

![result](/img/17-6/20-6.png)

接下来的操作就和在本地调试一样了，打断点，远端JVM会通过网络同步调试信息，和在本地没什么两样，要注意调试的时候和本地一样都是会暂停JVM继续往下执行的。





