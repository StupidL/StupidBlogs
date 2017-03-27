# 使用Gradle构建Java项目  

现在构建Java项目越来越多使用Gradle这个构建工具，如果你是Android Studio的使用者，那一定不会陌生。今天这个教程是针对一般的Java项目的。废话不多说，直接开始吧。  

## 安装JDK  

作为一个Java开发者，这个是配置环境必须的一个步骤，大家也不会陌生，这里就不多说了。  

## 安装Gradle
首先你应该安装Gradle构建工具，Gradle工具的下载安装可以看这里：[gradle-install](https://gradle.org/install)   
配置环境变量之类也不多说。  

## 实战  

```bash  
stupidl@StupidL:~$ mkdir -p Java/HelloJava
stupidl@StupidL:~$ cd Java/HelloJava
stupidl@StupidL:~/Java/HelloJava$ 
```  

以上就创建了一个项目的空目录：在用户stupidl的目录下面，创建了一个存放Java项目的文件夹**Java**，然后在这个文件夹里面创建了我们的项目目录文件夹**HelloJava**。  

接下来，就是创建项目的文件夹了：  

```bash  
stupidl@StupidL:~/Java/HelloJava$ mkdir -p src/main/java/me/stupidme/hellojava  

stupidl@StupidL:~/Java/HelloJava$ tree .
.
└── src
    └── main
        └── java
            └── me
                └── stupidme
                    └── hellojava

6 directories, 0 files  

```  

以上就是我们的项目结构： **src/main/java/** 目录下面放的是Java代码，**me/stupidme/hellojava** 是一个包名，可以改任意合适的包名。  

接下来，我们在包名 **me/stupidme/hellojava/** 下面写两个简单的Java文件:  

```java  
public class HelloWorld {
	public static void main(String[] args) {
		Greeter greeter = new Greeter();
		System.out.println(greeter.sayHello());
	}
}  
```  


```java  
public class Greeter {
	public String sayHello() {
		return "Hello world!";
	}
}  
```  

到此为止，好像都和Gradle没有什么关系。不要着急，Gradle要出场了。  

假设我们写好了项目的所有代码，那么怎么能够轻松的构建整个系统呢？难道要我们自己一个一个 **.java** 编译成字节码 **.class** ？显然是不现实的。Gradle就是这样一个工具，能够轻松将我们写好源代码的项目构建起来。具体做法如下：  

在我们项目的根目录下，创建一个文件 **build.gradle** ，这个脚本就是gradle工具构建项目的依据。我们这个项目的根目录即 **“~/Java/HelloJava/”** 。  

**build.gradle** 文件的内容如下：  

```gradle  
apply plugin: 'java'  
```  

暂时就这句话。它告诉Gradle，这是一个Java项目。  

此时，我们的项目结构是这样的:  

```bash  
stupidl@StupidL:~/Java/HelloJava$ tree .
.
├── build.gradle
└── src
    └── main
        └── java
            └── me
                └── stupidme
                    └── hellojava
                        ├── Greeter.java
                        └── HelloWorld.java

6 directories, 3 files
```  


接下来就可以构建项目了：  

```bash  
stupidl@StupidL:~/Java/HelloJava&  
stupidl@StupidL:~/Java/HelloJava& gradle build  
:compileJava
:processResources UP-TO-DATE
:classes
:jar
:assemble
:compileTestJava UP-TO-DATE
:processTestResources UP-TO-DATE
:testClasses UP-TO-DATE
:test UP-TO-DATE
:check UP-TO-DATE
:build

BUILD SUCCESSFUL

Total time: 1.986 secs  
stupidl@StupidL:~/Java/HelloJava$ tree .
.
├── build
│   ├── classes
│   │   └── main
│   │       └── me
│   │           └── stupidme
│   │               └── hellojava
│   │                   ├── Greeter.class
│   │                   └── HelloWorld.class
│   ├── libs
│   │   └── HelloJava.jar
│   └── tmp
│       ├── compileJava
│       └── jar
│           └── MANIFEST.MF
├── build.gradle
└── src
    └── main
        └── java
            └── me
                └── stupidme
                    └── hellojava
                        ├── Greeter.java
                        └── HelloWorld.java

16 directories, 7 files
stupidl@StupidL:~/Java/HelloJava$  
```  

Gradle的威力当然不止于此，它还能够自动下载依赖包。  
考虑这种情况：你需要一个在mavenCentral等平台的开源项目包，那么传统的做法是下载jar包，或者在maven这个工具里面添加依赖。在Gradle里面，这个步骤更加简单。具体请看以下例子。  



我将 **HelloWorld.java** 文件改成如下：  

```java  
package me.stupidme.hellojava;

import org.joda.time.LocalTime;

public class HelloWorld{
    public static void main(String[] args){
        LocalTime currentTime = new LocalTime();
        System.out.println("The current time is: " + currentTime);

        Greeter greeter = new Greeter();
        System.out.println(greeter.sayHello());
    }
}  
```  
现在我们需要joda-time这个包，但是本地没有。此时，我们可以在根目录的 **build.gradle** 里面配置依赖。  
此时，我们的 **build.gradle** 文件改成如下内容：  

```gradle  
apply plugin: 'java'

repositories{
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies{
    compile "joda-time:joda-time:2.2"
    testCompile "junit:junit:4.12"
}

jar{
    baseName = 'hello-java'
    version = '0.1.0'
}  
```  
首先是添加了mavenCentral仓库，然是说明Java代码相关版本。然后添加了joda-time和junit两个依赖。如果需要其他依赖，则在次处添加即可。最后，说明了编译后的jar包名称。  

我们再次编译，得到的目录结构如下：  

```bash  
stupidl@StupidL:~/Java/HelloJava$ gradle build
:compileJava
:processResources UP-TO-DATE
:classes
:jar
:assemble
:compileTestJava UP-TO-DATE
:processTestResources UP-TO-DATE
:testClasses UP-TO-DATE
:test UP-TO-DATE
:check UP-TO-DATE
:build

BUILD SUCCESSFUL

Total time: 2.205 secs
stupidl@StupidL:~/Java/HelloJava$ tree .
.
├── build
│   ├── classes
│   │   └── main
│   │       └── me
│   │           └── stupidme
│   │               └── hellojava
│   │                   ├── Greeter.class
│   │                   └── HelloWorld.class
│   ├── libs
│   │   ├── hello-java-0.1.0.jar
│   │   └── HelloJava.jar
│   └── tmp
│       ├── compileJava
│       │   └── emptySourcePathRef
│       └── jar
│           └── MANIFEST.MF
├── build.gradle
└── src
    └── main
        └── java
            └── me
                └── stupidme
                    └── hellojava
                        ├── Greeter.java
                        └── HelloWorld.java

17 directories, 8 files
stupidl@StupidL:~/Java/HelloJava$  
```  

接下来，有一个很有用的步骤——生成Gradle Wrappr。这样当别人克隆了这个项目的时候，就不需要安装本版本的gradle工具即刻运行项目了。  

```bash  
stupdl@StupidL:~/Java/HelloJava$  gradle wrapper --gradle-version 3.3  
:wrapper

BUILD SUCCESSFUL

Total time: 2.33 secs
stupidl@StupidL:~/Java/HelloJava$ tree .
.
├── build
│   ├── classes
│   │   └── main
│   │       └── me
│   │           └── stupidme
│   │               └── hellospring
│   │                   ├── Greeter.class
│   │                   └── HelloWorld.class
│   ├── libs
│   │   ├── hello-spring-0.1.0.jar
│   │   └── HelloSpring.jar
│   └── tmp
│       ├── compileJava
│       │   └── emptySourcePathRef
│       └── jar
│           └── MANIFEST.MF
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── src
    └── main
        └── java
            └── me
                └── stupidme
                    └── hellospring
                        ├── Greeter.java
                        └── HelloWorld.java

19 directories, 12 files  
```  

接下来，就可以这样构建项目了：  

```bash  
stupidl@StupdL:~/Java/HelloJava$ ./gradlew build  

```  

它会下载对应上述 **3.3** 版本的Gradle，然后进行构建。  

到目前为止，还有一个问题，那就是我们的项目如何运行。其实也很简单，在build.gradle 文件添加以下内容：  

```bash  
apply plugin: 'application'  

mainClassName = 'me.stupidme.hellojava.HelloWorld'  
```  

接下来就可以运行项目了：  
```bash  
stupidl@StupidL:~/Java/HelloJava$ ./gradlew run  
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:run
The current time is: 16:54:49.677
Hello world!

BUILD SUCCESSFUL

Total time: 9.235 secs
stupidl@StupidL:~/Java/HelloJava$  
```  

Gradle的基本使用就是如此了。欢迎指正错误。谢谢！