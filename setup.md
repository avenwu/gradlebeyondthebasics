## Setup

为了运行本章中的代码，你需要有一个数据库以供Liquibase链接。本书的示例代码中有一个build文件，用于连接H2数据库。现在我们用Git来获取代码，克隆http://github.com/gradleware/oreilly-gradle-book-examples 这个仓库,进入plugins/database-setup目录，执行下面的任务：

```
$ gradle -b database.gradle createDatabaseScript
$ gradle -b database.gradle buildSchema
```

第一个命令，会创建一个名为starth2的平台相关脚本，用于启动内嵌的H2数据库，并提供可以在插件开发过程中观察数据库协议的窗口。第二个命令，会创建一个示例数据库，就像我们的测试开发环境。

## 勾勒你的插件

我们的Liquibase插件要解决的事情包括：创建任务执行ChangeLog的逆向工程，ChangeLog同步，更新ChangeLog.深入分析Liquibase的API后，可以知道执行这些命令的最佳方式是调用liquibase.integration.commandling.Main.main()方法，这个方法接受一个数组参数，分别表示需要连接的数据库以及需要执行的Liquibase命令。每个需要执行Liquibase操作的任务，我们沟通过这种参数形式来调用这个API。

现在认真思考一下这些任务的确切含义。鉴于我们要支持三个Liquibase命令--generateChangeLog, changeLogSync, update--我们可以创建三个同名的task。在有些情形下，我们可能会倾向于为task加上前缀来更好的进行区分，但是这里我们尽可能保持简单明了，因此不加任何前缀标示。

还记得前面提到的，我们要支持DSL语法吧，现在我们暂时把这个放一边，在后续真正需要的地方在回过头来处理这个实现。

### 自定义Liquibase Tasks

我们的插件最后会引入实现完整的task，通过少量配置来调起Liquibase。在这之前，我们需要创建一个自定义的的task类型，这个task的作用在于将task的参数转换为Liquibase需要的参数列表。具体的实现详见Example 2-1.

**Example 2-1. Liquibase task原型**

```
import org.gradle.api.DefaultTask
import org.gradle.api.tasks.TaskAction
import liquibase.integration.commandline.Mainclass LiquibaseTask extends DefaultTask {  
	String command 
	String url, password, username 
	File changeLog  
	
	@TaskAction 
	def liquibaseAction() {  
		def args = [ 
			"--url=${url}",
			"--password=${password}", 
			"--username=${username}", 
			"--changeLogFile=${changeLog.absolutePath}", 
			command 
		]
		Main.main(args as String[])  
	}
}
```

记住，自定义的task类型只是一个简单的类，他实现了org.gradle.api.Task接口，或者继承自org.gradle.api.DefaultTask。LiquibaseTask提供了基础的接口用于构建任务和Liquibase命令驱动之间操作。LiquibaseTask的属性后续将会变为task的配置参数。

定义好task后，我们只需要创建爱你这个类型的task实例，并且配置参数。Example2-2中，我们利用Gradle的配置语法为task的属性进行了赋值。

**Example 2-2. 初始化自定义的Liquibase task**

```
task generateChangeLog(type: LiquibaseTask) { 
	url = 'jdbc:h2:db/gradle_plugins' 
	username = 'secret' 
	password = 'sa' 
	changeLog = file('changelog.xml') 
	command = 'generateChangeLog'
}
```

### 使用

现在我们已经有了自定义的task，为在Gradle中执行Liquibase创造了可能。现在我们回过头，开始创建插件。最简单的Gradle插件就是一个实现了org.gradle.api.Plugin<Project>接口的类。这个类定义了唯一的方法：void apply(Project project)。

**Example 2-3 The apply() method of the first version of the Liquibase plug-in**

```
class LiquibasePlugin implements Plugin<Project> {  
	void apply(Project project) { 
		project.task('generateChangeLog', type: LiquibaseTask) { 
			group = 'Liquibase' 
			command = 'generateChangeLog' 
		} 
		project.task('changeLogSync', type: LiquibaseTask) { 
			group = 'Liquibase' 
			command = 'changeLogSync' 
		} 
		project.task('update', type: LiquibaseTask) { 
			group = 'Liquibase' 
			command = 'update' 
		}  
	}
}
```
顺带提醒一下，一开始写插件的时候，可以直接在build.gradle中编写这个类和相关的LiquibaseTask。现在我们需要了解和学习插件API以及scope和插件设计的本身。开发和打包最后再讲，现在先忽跳过。

我们的示例中创建了三个新的task，分别为：generateChangeLog，changeLogSync和update。因为我们用的是Groovy语言，因此可以用类似于Gradel的语法声明这些新的task。这里展示的代码可以一字不差的用到build文件中，无论有没有定义我们的插件。构建者不一定非用Groovy来编写插件，但是鉴于Groovy的语法接近月Gradle的build文件语法，和他相对Java来说更高的效率，使用Groovy是个不错的选择。

## Extensions

现在我们的插件差不多可以工作起来了，但是它的一些配置有些繁复。我们需要为每个task配置数据库的用户名，密码，URL以及changelog文件。

**Example 2-4. The Liquibase-enabled build so far**

```
generateChangeLog {
  url = 'jdbc:h2:db/gradle_plugin'
  password = 'secret'
  username = 'sa'
  changeLog = file('changelog.xml')
}

changeLogSync {
  url = 'jdbc:h2:db/gradle_plugin'
  password = 'secret'
  username = 'sa'
  changeLog = file('changelog.xml')
}

update {
  url = 'jdbc:h2:db/gradle_plugin'
  password = 'secret'
  username = 'sa'
  changeLog = file('changelog.xml')
}

```

显然，我们不能仅仅满足于现状。Gradle插件真正强大的地方不单是能隐藏大量冗余的声明代码，这个优点早在Gradle 1.0发布的十多年前，通过自定义Ant的task和Maven插件就已经能实现了。Gradle插件还拥有扩展model的能力，也是我们介绍的Extension API的主要功用。

设计插件extension应该从简单的build文件语法开始，在设计build语法前，先思考我们的构建文件要处理的是什么样的东西。举例来说，在上面的示例中，我们需要知道数据库和changelog。

数据库就是一个通过JDBC连接的特定的数据库实例。通过Liquibase自动迁移会有几个独立的数据库沙箱对象，比如stage是服务器上的数据库实例用于ad_hoc测试，正式数据库等等。

Liquibase的changelog文件包含了一个有序列表，内容为数据库上所有的重构操作日志，日志以XML格式存储。更多的信息可以在官网的[Liquibase changelog](http://www.liquibase.org/documentation/databasechangelog)处获取。Example 2-5有一个changelog，真实环境下，一般会有多个日志文件。我们的模型必须能够支持任意数量的changelog文件。

**Example 2-5. The goal of our plug-in’s new DSL**

```
liquibase {
  changelogs {
    main {
      file = file('changelog.groovy')
    } 
  }
  databases {
    sandbox {
      url = 'jdbc:h2:db/liquibase_workshop;FILE_LOCK=NO'
      username = 'sa'
      password = ''
    }
    staging {
      url = 'jdbc:mysql://staging.server/app_db'
      username = 'dev_account'
      password = 'ab87d24bdc7452e557'
    } 
  }
  defaultDatabase = databases.sandbox
}
```
