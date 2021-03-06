# 文件懒加载

在使用文件集合编程的时候，你可能会认为这些文件集合是静态的列表。比如说，在调用了fileTree()之后会同时扫描整个文件系统，然后生成一个不可变的列表一共构建过程中使用。当然不可变的数据结构有他们的用处，但是在Gradle的构建生命周期中，这会使得文件接口不易使用。因此，FileCollection接口的实例都是懒加载的，只有在真正需要的时候才会执行。

举例来说，一个task使用了fileTree()来创建一个文件集合，这个集合包含build/main/classes目录下的所有符合**/*Test.class正则表达式的文件。如果这个文件集合是在文件配置阶段就创建了，那么他需要的文件很可能是不存在的，直到在执行期间build下的文件才会生成。因此，文件集合被设计为了集合语义的静态描述，而真正的集合内容是在执行过程中动态处理的。