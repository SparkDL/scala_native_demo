# scala_native_demo
Demo for scala-c++ cross compile project using sbt and cmake (which could directly work with idea and clion).

## Structure
|目录|详情|  
|---|---|  
|demo_scala| 使用sbt构建的scala项目，可以直接使用idea打开并开发|
|demo_cpp| 使用cmake构建的c++项目，可以直接使用clion打开并开发|
|lib| demo_cpp生成的动态链接库|

## Workflow
### 1. 打开项目
使用idea打开Sample项目，clion打开SampleSubc项目。

### 2. 创建含有native方法的scala类
在idea内，/src/main/scala/文件夹下创建一个含有native方法的scala类：

例如
```scala
class OwnMath {
  @native def add(a:Double,b:Double):Double
}
```

### 3. 编译scala类
选择compile配置，如下图，注意是compile不是run。

![](https://coding.net/u/zeta159/p/Sample/git/raw/master/doc/sbt-task-icon.jpg)
 
然后点击旁边的三角符号运行compile任务（它是一个sbt task）进行编译。

编译时构建脚本会调用`javah`命令生产相应的.h文件到“SampleSubc/OwnMath.h”内，并且自动生成相应的.cpp文件，在CMakeLists.txt里加入一个新的动态库target。

### 4. 实现native方法并编译c++动态库
然后转到clion，编写native方法对应的c++方法，例如OwnMath.cpp里的内容：
```cpp

#include "OwnMath.h"

/*
 * Class:     OwnMath
 * Method:    add
 * Signature: (DD)D
 */
JNIEXPORT jdouble JNICALL Java_OwnMath_add
  (JNIEnv *env, jobject obj, jdouble a, jdouble b) {
    return a+b;
}


```

完成之后编译即可，方法是依次点击菜单栏的Run->Build选项进行cpp工程的编译，编译完成后，动态库会生成到`lib`目录内。

### 5.在scala内使用动态库
动态库编译完成就可以在scala项目内任意使用了。

这是因为在jvm的启动参数里设置了`-Djava.library.path=/Users/pcz/IdeaProjects/Sample/lib`选项。

设置方式是点击“Edit Configurations...”

![](https://coding.net/u/zeta159/p/Sample/git/raw/master/doc/sbt-task-icon.jpg)

然后进入如下的页面：

![](https://coding.net/u/zeta159/p/Sample/git/raw/master/doc/vm-option.jpg)

在自己使用的时候需要将其中的路径改成自己的lib路径。

在scala代码内使用native类的方法是调用`System.loadLibrary()`加载动态库，然后直接使用类即可：

```
object Main {
  def main(args: Array[String]): Unit = {
    System.loadLibrary("OwnMath")

    val c = new OwnMath
    println(c.add(123.0, 123.0))
  }
```
