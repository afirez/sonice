# 说说系统的启动

## Android 有哪些主要的系统进程

init.rc 中配置了很多系统服务。

![services_in_init_rc](https://raw.githubusercontent.com/afirez/sonice/master/android/assets/services_in_init_rc.png)

另外 SystemServer 进程是在 Zygote 启动之后由 Zygote 启动的， 不是由 init 启动的。

## 这些系统进程是怎么启动的

### Zygote 的启动流程

- 1、 init 进程 fork 出 Zygote 进程
- 2、 启动虚拟机， 注册 JNI 函数 （未进入 Java 世界作准备）
- 3、 预加载系统资源
- 4、 启动 SystemServer
- 5、 进入 Socket Loop 循环

### SystemServer 的启动流程

![init_system_server](https://raw.githubusercontent.com/afirez/sonice/master/android/assets/init_system_server.png)

SytemServer 启动后会去启动其他系统服务， 之后进入循环。

### QA

1、 怎么解决系统服务启动的相互依赖？

2、 系统服务是怎么启动？

3、 系统服务跑在工作线程？（主线程？工作线程？Binder 线程？）

4、 为什么系统服务不都跑着 Binder 线程？

5、 为什么系统服务不都跑在工作线程？

6、 到底该跑在 Binder 线程还是工作线程？

## 进程启动之后主要做了哪些事情
