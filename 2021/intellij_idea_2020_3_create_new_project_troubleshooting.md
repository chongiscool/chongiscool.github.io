```txt
IntelliJ IDEA 2020.3 (Community Edition)
Build #IC-203.5981.155, built on December 1, 2020
Runtime version: 11.0.9+11-b1145.21 x86_64
VM: OpenJDK 64-Bit Server VM by JetBrains s.r.o.
macOS 10.15.7
GC: ParNew, ConcurrentMarkSweep
Memory: 1998M
Cores: 6
Non-Bundled Plugins: org.jetbrains.kotlin
```

记录一下，创建Kotlin JVM新项目，在构建系统(Build System)选择`Gradle Kotlin` 和 `Gradle Groovy`时，都直接进入创建成功的页面，但gradle sync failed；我感觉应该是Gradle下载不下来 —— 导致有`main()`方法却不能运行，项目为构建成功时，那个运行的图案/标都没有；

那我是怎么盘活的呢？

1. 由于本机安装了Gradle，于是在命令行(terminal)各种一通操作；

```bash
gradle init
gradle wrapper

gradle build
gradle clean
```

再重启IDEA，然后就能运行`main()`函数了；

猜测1：可能是我设置socks代理，IDEA哪里有缓存，导致只要用到要下载Gradle时，直接失败；
猜测2：可能是我的代理抽风；

总之简单记录下

refs：

[1. Tutorial: Create your first Kotlin application](https://www.jetbrains.com/help/idea/create-your-first-kotlin-app.html)

[2. Creating a New Project in IntelliJ IDEA](https://blog.jetbrains.com/idea/2021/01/creating-a-new-project-in-intellij-idea/)
