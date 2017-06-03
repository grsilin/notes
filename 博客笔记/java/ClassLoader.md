
** Class 文件
  使用`javac`命令将 `.java`格式的源码文件编译成`.class`文件，这个文件里面是存放的是 Java 字节码。用 C 或 Python 编写的程序在正确转换成 `.lass`文件后，同样可以由 java 虚拟机加载运行。

** Java 类加载器
  * BootstrapClassLoader
  最顶层的加载类，主要加载核心类库， %JRE_HOME% \\lib 下的 `tr.jar`,`resources.jar`, `charset.jar` 和 class 等等。
  在启动 JVM 时，使用 `java -Xbootclasspath ` 来改变 Bootstrap ClassLoader 的加载目录。
  * ExtensionClassLoader
  扩展的类加载器，加载目录 %JRE_HOME%\\lib\\ext下的 jar 包和 class 文件。还可以加载 `-D java.ext.dirs` 选项指定的目录。
  * Appclass ClassLoader
  也称为 SystemAppClass ,加载当前应用的 classpath 的所有类。

类加载器的执行顺序为：
BootstrapClassLoader --> ExtensionClassLoader --> AppclassLoader

```
public class Launcher {
  private static Launcher launcher = new Launcher();
  private static String bootClassPath =
      System.getProperty("sun.boot.class.path");

  public static Launcher getLauncher() {
      return launcher;
  }

  private ClassLoader loader;

  public Launcher() {
      // Create the extension class loader
      ClassLoader extcl;
      try {
          extcl = ExtClassLoader.getExtClassLoader();
      } catch (IOException e) {
          throw new InternalError(
              "Could not create extension class loader", e);
      }

      // Now create the class loader to use to launch the application
      try {
          loader = AppClassLoader.getAppClassLoader(extcl);
      } catch (IOException e) {
          throw new InternalError(
              "Could not create application class loader", e);
      }

      //设置AppClassLoader为线程上下文类加载器，这个文章后面部分讲解
      Thread.currentThread().setContextClassLoader(loader);
  }

  /*
   * Returns the class loader used to launch the main application.
   */
  <!-- public ClassLoader getClassLoader() { -->
      return loader;
  }
  /*
   * The class loader used for loading installed extensions.
   */
  static class ExtClassLoader extends URLClassLoader {}

/**
   * The class loader used for loading from java.class.path.
   * runs in a restricted security context.
   */
  static class AppClassLoader extends URLClassLoader {}
```
BootstrapClassLoader 加载`   System.getProperty("sun.boot.class.path")` 得到的目录下的文件，他们是 jre 目录下的 jar 包 和 class 文件。

ExtensionClassLoader 加载`   System.getProperty("java.ext.dirs")` 得到的
`C:\Program Files\Java\jre1.8.0_91\lib\ext;C:\Windows\Sun\Java\lib\ext`下的文件。
AppclassLoader
加载 `java.class.path` 标识的目录下的文件，即当前工程的 bin 文件夹下的 class 文件。

** 类加载器的父加载器
自定义类的加载器为 `AppclassLoader`,其父加载器为`ExtClassLoader`,parent的赋值在 ClassLoader 对象的构造方法中，这有两种情况：
1、由外部类创建 ClassLoader 时直接指定一个 ClassLoader 为 parent 。
2、由 Launcher 的`getSystemClassLoader()`方法生成，它是一个 AppClassLoader 。这个 AppClassLoader 由一个 ExtClassLoader 构造，因此 AppclassLoaderr 的父加载器为 ExtClassLoader , 在构造作为参数用的 ExtClassLoader 时，其父加载器为 null ,因此 ExtClassLoader 的父加载器为空。

** BootstrapClassLoader
BootstrapClassLoader 是由C/C++编写的，它本身是虚拟机的一部分，无法在 Java 代码中获取它的引用， JVM 启动时通过 Bootstrap 类加载器加载 rt.jar 等核心 jar 包中的 class 文件。BootstrapClassLoader是ExtClassLoader 的父加载器，但是无法在 Java 代码中获取到。

** 双亲委托
一个类加载器在查找 class 和 resource 时，通过`委托模式` 进行：当需要加载一个类时，首先判断这个类是不是已经成功加载，没有找到则委托它的父加载器进行加载，父加载器同样进行判断，直到找到或传递到 BootstrapClassLoader ,BootstrapClassLoader没有找到，则一级一级返回，最后到达自身来查找需要的对象。

```java
if (parent != null) {
    //父加载器不为空则调用父加载器的loadClass
    c = parent.loadClass(name, false);
} else {
    //父加载器为空则调用Bootstrap Classloader
    c = findBootstrapClassOrNull(name);
}
```
在进行委托时，当父容器不存在时，指定 `BootstrapClassLoader` 进行加载。

** 自定义 ClassLoader
  步骤：
  1. 编写一个类继承 ClassLoader 抽象类。
  2. 重写 `findClass()`方法。
  3. 在`findClass()`方法中调用`defineClass()`。
  *** defineClass()
  将`.class`文件中的二进制内容转换成 Class 对象。
