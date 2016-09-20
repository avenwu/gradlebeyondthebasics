## 插件案例

在这一节中，我们会创建一个Gradle插件，用于自动调用名为Liquibase的开源数据库重构工具。Liquibase是用Java编写的命令行工具，用于管理关系型数据库协议。他可以将一个已经存在的数据库协议逆向工程为一个XML的的变更日志，并且记录数据库变化的版本号，已决定是否需要对新的数据库执行重构。对应习惯Groovy语法的开发者们也可以选择使用Groovy Liquibase DSL

更多的的Liquibase信息可以在[Liquibase Quick Start](http://www.liquibase.org/quickstart)查阅。

Liquibase可以很好地实现他的功能，但是在使用上面，如果没有一个封装好的脚本，会比较笨重。另外，高度自动化的构建和发布通常是一个非显式的目标，我们倾向于把Liquibase的操作整合到构建的生命周期中。

在这一节汇总，我们的目标是完成下面这些事项：

* 创建一个Gradle的task，与generateChangeLog,changeLogSync和update这几个命令对应；
* 使用Groovy DSL配置默认的XML日志格式；
* 把这个Gradle的task重构为自定义的task类型；
* 引入Gradle DSL extensions来描述Changelogs和数据库配置；
* 将插件打包为独立的JAR文件；

我们的Liquibase插件奖惩标准的Gradle build文件开始编写。这种方式相对来说更容易编写原型代码，这也是开发新的自动构建任务时典型的工作流程。当插件开始形成我们想要的东西是，在慢慢将其重构为独立的插件项目。这种方式开发插件非常合适，并且降低了学习API的成本。
