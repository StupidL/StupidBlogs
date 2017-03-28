# 第一次体验SpringBoot  

因为毕设，最近觉得有必要了解下后端开发。通过网上找资料，发现现在的Spring框架很流行，所以决定尝试一下，以便了解Spring。  

## 准备工作  
* Java开发环境  
* Gradle构建工具  
* 云主机  

我的云主机是腾讯云提供的，安装的操作系统为Ubuntu16.04 x64，主机的开发环境如下：  

```bash  
ubuntu@VM-252-55-ubuntu:~$ java -version
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
ubuntu@VM-252-55-ubuntu:~$
```  

```bash  
ubuntu@VM-252-55-ubuntu:~$ gradle -v

------------------------------------------------------------
Gradle 3.4.1
------------------------------------------------------------

Build time:   2017-03-03 19:45:41 UTC
Revision:     9eb76efdd3d034dc506c719dac2955efb5ff9a93

Groovy:       2.4.7
Ant:          Apache Ant(TM) version 1.9.6 compiled on June 29 2015
JVM:          1.8.0_121 (Oracle Corporation 25.121-b13)
OS:           Linux 4.4.0-66-generic amd64

ubuntu@VM-252-55-ubuntu:~$  
```  

## 实战  
官方[Spring.io](https://spring.io/guides/gs/rest-service/#scratch) 的教程也友好，所以你也可以直接按照教程来体验。教程还有Maven相关的内容，因为我只是用Gradle，所以这篇博客都是用Gradle进行构建的，对Maven感兴趣的同学可以去上述网址学习。废话到此为止。  

### 创建项目  

```bash  
ubuntu@VM-252-55-ubuntu:~/JavaProjects$ mkdir demo && cd demo
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ ls
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$
```  
上面创建了项目 **demo** ，接下来要创建常见的文件结构目录：  
```bash  
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ mkdir -p src/main/java/me/stupidme/demo
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ tree .
.
`-- src
    `-- main
        `-- java
            `-- me
                `-- stupidme
                    `-- demo

6 directories, 0 files
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$  
```  

因为项目使用Gradle构建，所以需要在根目录创建 **build.gradle** 文件，内容如下：  
```bash  
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ cat build.gradle
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.2.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

jar {
    baseName = 'demo'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile('org.springframework.boot:spring-boot-starter-test')
}

ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$  
```  

以上的内容什么意思就不需要详细解释了，最主要的就是引入了spring依赖。关于Gradle的基本使用，请参考我的另一篇博客[使用Gradle构建Java项目](http://123.206.221.113/article/build-java-projects-with-gradle) 。  

接下来就是写代码了。

### 编写代码  

我想要达到这样一个效果：  
 当我在浏览器访问 http://123.206.221.113/hello 的时候，能够看到以下的内容：  
```bash  
{
	"id" : 1,
	"content" : "Hello, World!"
}  
```  

我们将上面的内容，抽象成为一个POJO——在包下创建一个 **Greeting.java** 文件，内容如下：  
```java  
package me.stupidme.demo;

public class Greeting {
	private Lon id;
	private String content;

	public Greeting(long id, String content) {
		this.id = id;
		this.content = content;
	}

	public long getId() {
		return id;
	}

	public String getContent() {
		return content;
	}
}  
```  
Spring会使用Jackson这个库，将Greeting类映射成为JSON文件。  

当浏览器发送请求的时候，服务端需要处理请求，那么Spring如何处理呢？答案是新建一个控制器类，使用 **@RestController** 注解。我们在包内创建一个 **GreetingController.java** 文件，内容为：  

```java  
package me.stupidme.demo;

import java.util.concurrent.atomic.AtomicLong;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @RequestMapping("/hello")
    public Greeting greeting(@RequestParam(value="name", defaultValue="World") String name) {
        return new Greeting(counter.incrementAndGet(), String.format(template, name));
    }
}  
```  


接下来，我们需要是项目运行起来，编写 **Application.java** 文件：  

```java  
package me.stupidme.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}  
```  

此时，我们的目录结构为：  

```bash  
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ vim src/main/java/me/stupidme/demo/Greeting.java
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ vim src/main/java/me/stupidme/demo/GreetingController.java
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ vim src/main/java/me/stupidme/demo/Application.java
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ tree .
.
|-- build.gradle
`-- src
    `-- main
        `-- java
            `-- me
                `-- stupidme
                    `-- demo
                        |-- Application.java
                        |-- Greeting.java
                        `-- GreetingController.java

6 directories, 4 files
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ 
```    

### 使用Gradle构建项目  
接下来，构建项目，使用Gradle：  

```bash  
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ gradle wrapper --gradle-version 3.4.1
Starting a Gradle Daemon (subsequent builds will be faster)
:wrapper UP-TO-DATE

BUILD SUCCESSFUL

Total time: 10.085 secs
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ ./gradlew build
:compileJava
:processResources NO-SOURCE
:classes
:findMainClass
:jar
:bootRepackage
:assemble
:compileTestJava NO-SOURCE
:processTestResources NO-SOURCE
:testClasses UP-TO-DATE
:test NO-SOURCE
:check UP-TO-DATE
:build

BUILD SUCCESSFUL

Total time: 4.048 secs
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$  
```  

看下我们的目录结构：  

```bash  
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ tree .
.
|-- build
|   |-- classes
|   |   `-- main
|   |       `-- me
|   |           `-- stupidme
|   |               `-- demo
|   |                   |-- Application.class
|   |                   |-- Greeting.class
|   |                   `-- GreetingController.class
|   |-- libs
|   |   |-- demo-0.1.0.jar
|   |   `-- demo-0.1.0.jar.original
|   `-- tmp
|       |-- compileJava
|       `-- jar
|           `-- MANIFEST.MF
|-- build.gradle
|-- gradle
|   `-- wrapper
|       |-- gradle-wrapper.jar
|       `-- gradle-wrapper.properties
|-- gradlew
|-- gradlew.bat
`-- src
    `-- main
        `-- java
            `-- me
                `-- stupidme
                    `-- demo
                        |-- Application.java
                        |-- Greeting.java
                        `-- GreetingController.java

18 directories, 14 files
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$  
```  

### 运行看效果  

首先，我们运行 **jar** 文件：  

```bash  
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ java -jar build/libs/demo-0.1.0.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.2.RELEASE)

2017-03-28 22:58:48.503  INFO 2514 --- [           main] me.stupidme.demo.Application             : Starting Application on localhost with PID 2514 (/home/ubuntu/JavaProjects/demo/build/libs/demo-0.1.0.jar started by ubuntu in /home/ubuntu/JavaProjects/demo)
2017-03-28 22:58:48.512  INFO 2514 --- [           main] me.stupidme.demo.Application             : No active profile set, falling back to default profiles: default
2017-03-28 22:58:48.763  INFO 2514 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@41906a77: startup date [Tue Mar 28 22:58:48 CST 2017]; root of context hierarchy
2017-03-28 22:58:53.095  INFO 2514 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 8080 (http)
...  

```  

你会看到以上信息，说明已经启动成功了。从上面的一大串日志中，我们可以发现，现在使用的容器是Tomcat，端口为8080。  

接下来，就可以在浏览器访问 http://123.206.221.113/hello 了，效果如下：  

![one](http://123.206.221.113:80/upload/2017/03/u4nasscpgogi1qnt65eqilkmmo.PNG)  
![two](http://123.206.221.113:80/upload/2017/03/thb7ejhu4mjsiqp50bhll2n0el.PNG)  

刷新几次，会发现 **id** 值递增。  

在云主机上当然也可以看看效果：  

```bash  
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ curl http://localhost:8080/hello
{"id":4,"content":"Hello, World!"}
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ curl http://localhost:8080/hello
{"id":5,"content":"Hello, World!"}
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ curl http://localhost:8080/hello
{"id":6,"content":"Hello, World!"}
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$ curl http://localhost:8080/hello
{"id":7,"content":"Hello, World!"}
ubuntu@VM-252-55-ubuntu:~/JavaProjects/demo$  
```  
#### 备注  
* 123.206.221.113是我云主机的公网ip地址  
* 服务端口使用的是8080

至此，就是我第一次体验SpringBoot所做的工作了。感觉有了SpringBoot确实很方便，告别了传统的一大堆xml配置文件，取而代之的是注解，我们可以使用纯粹的Java代码构建一个应用。而且不需要处理类似Android烦人的UI界面，感觉后端开发的体验还是不错的。  
