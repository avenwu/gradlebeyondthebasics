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
~/file-collection-lab/src/main/java/org/gradle/example/DefaultContentFactory.java, 
~/file-collection-lab/src/main/java/org/gradle/example/DonneContent.java, 
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

把FileCollection转换为字符路径的最简单案例就是module依赖。有必要指出的是依赖配置本事就是文件集合。当我们把依赖的module写在build文件的dependencies{}模块中时，他们总是被赋值给一个被命名过的configuration。configuration定义在configurations{}模块中。

作为一名教师和经常参加分享会介绍Gradle的演讲者，有时候我会想让学生们在没有网络连接的情况下编译Java代码（许多美国的旅店和会议中心，他们的网络带宽都很有限，不足以支撑大量网络终端连接）。尽管我个人倾向于用构建工具管理所有的依赖，比如把所有的依赖都下载到本地。但是对于有些Java framework和APIs,手动下载这些Jars是一个不小的负担。通过使用文件集合形式的模块依赖，我们可以把这些工作自动化。

如果我们把下面这个copy任务添加到Example 1-19中，会发现执行task后lib目录下生成了我们需要的这些JARs文件。新增依赖后后，只需要再次执行这个task，接着新的JARs就会按预期出现在目录中。注意看这里的from，他接受一个目录名，当然也可以接受FileCollection。

Example 1-22. 把module依赖作为FileCollection输出JAR文件

```groovy
task copyDependencies(type: Copy) { 
    from configurations.compile into 'lib'
}
```

## 添加和截取FileCollection元素

FileCollection支持添加和删除操作，我们可以用以来配置来得到一个测试用的文件集合。

这一节中我们的案例是一个使用了Spring框架的命令行程序。Spring会引入一堆的JARs,因此命令行执行程序会相当痛苦。JavaExec任务提供了一些方便的手段来解决这个问题，比如运行的类和启动JVM启动的classpath。

我们需要的classpath包括两部分：所有编译时的依赖和源码目录下编译出来的的class。前者可以通过configurations.compile结合，后者通过sourceSets.main.output.稍后介绍sourceSet。

Example 1-23. 添加运行时classpath

```groovy
task run(type: JavaExec) { 
    main = 'org.gradle.example.PoetryEmitter' classpath = configurations.compile + sourceSets.main.output
}
```

从一个文件结合取子集会创建一个新的文件集合。比如创建一个集合包含除了Shelley的诗歌之外的所有文本资源。  
Example 1-24.创建两个FileCollection的交集

```groovy
def poems = fileTree(dir: 'src/main/resources', include: '*.txt')def romantics = fileTree(dir: 'src/main/resources', include: 'shelley*')
def goodPoems = poems - romantics
```

输出goodPoems包含的文件，可以发现只包含src/main/resources目录下以.txt结尾的，不以shelley开头的文件。在特殊的构建当中，可能由exclude来完成相同效果，也可以处理一些更细节的操作，比如在构建Java Web应用中去除WAR任务的一些JARs依赖。

## FileCollection作为SourceSet

在更早的示例中，我们用了fileTree()方法创建一个包含所有源码的文件集合。实际上Gradle提供了一个更简单的实现方式。SourceSets是Gradle用来承载所有源码的对象，他们正好一FileCollection的像是对外提供输入和输出。

SourceSet的allSource属性返回一个包含所有输入的文件集合，比如Java 项目中，也包括了所有的resurce文件。在我们的例子中，我们检测这个苏醒然后把结果输出到终端Example 1-25。

Example 1-25.输出java SOurceSet下的文件

```shell
println sourceSets.main.allSource.files

[~/file-collection-lab/src/main/resources/application-context.xml, 
~/file-collection-lab/src/main/resources/chesterton.txt,
~/file-collection-lab/src/main/resources/henley.txt, 
~/file-collection-lab/src/main/resources/shakespeare.txt, 
~/file-collection-lab/src/main/resources/shelley.txt, 
~/file-collection-lab/src/main/java/org/gradle/example/ConsoleContentSink.java, 
~/file-collection-lab/src/main/java/org/gradle/example/Content.java, 
~/file-collection-lab/src/main/java/org/gradle/example/ContentFactory.java, 
~/file-collection-lab/src/main/java/org/gradle/example/ContentRegistry.java, 
~/file-collection-lab/src/main/java/org/gradle/example/ContentSink.java, 
~/file-collection-lab/src/main/java/org/gradle/example/DefaultContentFactory.java, 
~/file-collection-lab/src/main/java/org/gradle/example/DonneContent.java, 
~/file-collection-lab/src/main/java/org/gradle/example/PoetryEmitter.java, 
~/file-collection-lab/src/main/java/org/gradle/example/ShakespeareContent.java]
```

类似的构建中的输出可以通过outputs属性得到。outputs不会输出详细的所有文件，他的输出是一个目录列表：

```shell
println sourceSets.main.output.files 
[~/file-collection-lab/build/classes/main,
~/file-collection-lab/build/resources/main]
```