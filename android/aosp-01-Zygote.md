# aosp-Zygote

## 谈谈你对 Zygote 的理解

### Zygote 的作用

1、 启动 SystemServer

2、 孵化应用进程

### Zygote 的启动流程

#### 启动三段式

Zygote 进程，系统服务进程， 和应用进程都是类似的启动流程

##### 1、进程启动

- 进程启动两种方式

![process_launc](https://raw.githubusercontent.com/afirez/sonice/master/android/assets/process_launch.png)

- init 进程启动 Zygote 进程

![init.zygote32.rc](https://raw.githubusercontent.com/afirez/sonice/master/android/assets/init.zygote32.rc.png)

- 子进程挂了怎么办？

parent 进程 fork 出 child 进程， child 进程如果挂了， parent 进程会都到 SIGCHLD 信号处理。 例如 Zygote 进程如果挂了， init 进程就会收到 SIGCHLD 信号， 就会重启Zygote 进程

##### 2、准备工作

进程启动之后都干了什么？

- Zygote 的 Native 世界

目的是为进入Java世界做准备。

![native_enter_java](https://raw.githubusercontent.com/afirez/sonice/master/android/assets/native_enter_java.png)

- Zygote 的 Java 世界

![in_java](https://raw.githubusercontent.com/afirez/sonice/master/android/assets/in_java.png)

进入 Java 世界后， Zygote 会预加载资源（常用类， JNI 函数， 主题资源， 共享库等）， 然后启动 SystermServer 进程。

1、 启动虚拟机

2、 注册 JNI 函数

3、 进入 Java 世界

- Zygote 中的细节

1、 Zygote fork 启动要单线程。 Zygote 中是有很多和虚拟机相关的守护线程， 但是 fork 子进程的时候要保持只有主线程在运行， 其他线程会暂时停掉。 在多线程情况下会出现奇怪的问题（死锁等）。

2、 Zygote 的 IPC 没有采用 Binder， 而是采用的本地的 Socket。 所有应用程序的 binder 机制并不是从 Zygote 继承过来的， 而是应用进程创建之后自己启动的 Binder 机制。

##### 3、Loop

Zygote 启动并做好准备工作后 便进入 Loop 循环。

### Zygote 的工作原理

1、孵化应用这种事为什么不教给 SystemServer 来做，而专门设计一个 Zygote （Zygote 是怎么孵化进程的）？

2、Zygote 等通信机制为什么不采用 Binder， 如果采用 Binder 会有什么问题 （Zygote 是怎么和其他进程通信的）？
