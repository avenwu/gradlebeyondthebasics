# FileCollection接口 

<!-- toc -->

在前面的章节中我么已经介绍了一些文件相关的方法，你可能已经发现files和fileTree的返回值的toString()实现并不是太友好。假如这些返回值是ArrayList，我们希望toString()能够打印出他们的内容，但实际上结果并不是这样，而是输出了一些特殊的内容，这个输出结果就是今天要讲的FileCollection接口。

*Example 1-17. FileCollection的默认toString()实现*

```groovy
task copyPoems(type: Copy) { 
    from 'text-files' 
    into 'build/poems'
}
println "NOT HELPFUL:" 
println files(copyPoems)
```

我们执行以下这段代码，看看输出什么内容：

```shell
$ gradle -b file-collection-not-helpful.gradle 
NOT HELPFUL: 
file collection
```

*Example 1-18. 查看FileCollection的更好方法*

```groovy
task copyPoems(type: Copy) { 
    from 'text-files' 
    into 'build/poems'
}
println "HELPFUL:" 
println files(copyPoems).files
```

稍作修改后，再看看输出结果：

```shell
$ gradle -b file-collection-helpful.gradle 
HELPFUL: 
[~/oreilly-gradle-book-examples/file-operations-lab/build/poems]
```

files()返回值并不是List，而是FileCollection。FileCollection在Gradle中经常会出现，比如：SourceSet内容，task的输入输出，Java的classpaths，dependency依赖配置，等等。知道这些必要方法的返回值类型会在日后file相关的编码中获益良多，比如files是否是来自Maven仓库的transitive的JAR依赖，Groovy构建中的源代码，web application的静态资源。我们会探讨FileCollection提供的关键操作。

在介绍这些特性之前，先从一个简单的build文件开始，这个文件设置了一些有意思的文件集合如Example 1-19,我们稍后会陆续添加task来介绍不同的特性。

*Example 1-19. 集成build文件*

```groovy
apply plugin: 'java'
repositories { 
    mavenCentral()
}
dependencies { 
    compile 'org.springframework:spring-context:3.1.1.RELEASE'
}
```

## 转换为Set集合
我们已经知道FileCollection有一个files属性，这个属性返回的是一个Set<File>集合实例，包含了这个FileCollection中的所有文件或目录。现在我们添加一个task来输出这些文件Example 1-20.

*Example 1-20. 输出文件的原生方法*

```groovy
task naiveFileLister { 
    doLast {
         println fileTree('src/main/java').files 
    }
}
```

这个task的输出结果如下：

```shell
$ gradle nFL 
:naiveFileLister 
[~/file-collection-lab/src/main/java/org/gradle/example/ConsoleContentSink.java,
 ~/file-collection-lab/src/main/java/org/gradle/example/Content.java, 
~/file-collection-lab/src/main/java/org/gradle/example/ContentFactory.java, 
~/file-collection-lab/src/main/java/org/gradle/example/ContentRegistry.java, 
~/file-collection-lab/src/main/java/org/gradle/example/ContentSink.java, 
~/file-collection-lab/src/main/java/org/gradle/example/
 DefaultContentFactory.java, ~/file-collection-lab/src/main/java/org/gradle/example/DonneContent.java, 
~/file-collection-lab/src/main/java/org/gradle/example/PoetryEmitter.java, 
~/file-collection-lab/src/main/java/org/gradle/example/ShakespeareContent.java]

 BUILD SUCCESSFUL
```

## 转换为路径字符串

构建过程中经常会有出于不同目的需要操作文件集合，比如需要把文件集合作为参数传给一个系统命令。Java核心插件在处理编译时依赖，就需要用到javac编译器，Java编译器有一个命令行开关，用于指定classpath，这个开关必须以系统平台相关的string一起提供。FileCollection的asPath属性可以吧一个FileCollection转换为系统相关的字符串string。

*Example 1-21. 以字符串路径的形式输出编译时依赖文件*

```groovy
println configurations.compile.asPath
```

输出结果为：

```shell
$ gradle
 ~/.gradle/caches/artifacts-8/filestore/org.springframework/spring-context/ 3.1.1.RELEASE/jar/ecb0784a0712c1bfbc1c2018eeef6776861300e4/spring- context-3.1.1.RELEASE.jar:
~/.gradle/caches/artifacts-8/filestore/org.springframework/spring-asm/ 3.1.1.RELEASE/jar/8717ad8947fcada5c55da89eb474bf053c30e57/spring- asm-3.1.1.RELEASE.jar: 
~/.gradle/caches/artifacts-8/filestore/commons-logging/commons-logging/ 1.1.1/jar/5043bfebc3db072ed80fbd362e7caf00e885d8ae/commons-logging-1.1.1.jar: 
~/.gradle/caches/artifacts-8/filestore/org.springframework/spring-core/ 3.1.1.RELEASE/jar/419e9233c8d55f64a0c524bb94c3ba87e51e7d95/spring- core-3.1.1.RELEASE.jar: 
~/.gradle/caches/artifacts-8/filestore/org.springframework/spring-beans/ 3.1.1.RELEASE/jar/83d0e5adc98714783f0fb7d8a5e97ef4cf08da49/spring- beans-3.1.1.RELEASE.jar: 
~/.gradle/caches/artifacts-8/filestore/aopalliance/aopalliance/1.0/jar/ 235ba8b489512805ac13a8f9ea77a1ca5ebe3e8/aopalliance-1.0.jar: 
~/.gradle/caches/artifacts-8/filestore/org.springframework/spring-aop/ 3.1.1.RELEASE/jar/3c86058cdaea30df35e4b951a615e09eb07da589/spring- aop-3.1.1.RELEASE.jar: 
~/.gradle/caches/artifacts-8/filestore/org.springframework/spring-expression/ 3.1.1.RELEASE/jar/1486d7787ec4ff8da8cbf8752d30e4c808412b3f/spring- expression-3.1.1.RELEASE.jar
```

## FileCollection作为Module依赖

## 添加和截取FileCollection元素

## FileCollection作为SourceSet

