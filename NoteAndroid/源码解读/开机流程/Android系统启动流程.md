# Android系统启动流程

> 参考资料：
>
> 1. [Android进阶宝典 -- 史上最详细Android系统启动全流程分析 - 掘金 (juejin.cn)](https://juejin.cn/post/7232947178690412602)
> 2. [Android Code Search](https://cs.android.com/)
>
> 学习目标：
>
> 1. 了解Android系统启动流程，对整个系统启动的流程有个基本的认识 ----- 第一次学习；



## 1 系统启动流程分析

> 当我们打开电源键的时候，硬件执行的第一段代码就是BootLoader，会做一些初始化的操作，例如初始化CPU速度、内存等。然后会启动第一个进程idle进程（pid = 0），这个进程是在内核空间初始化的。
>
> idle进程作为系统启动的第一个进程，它会创建两个进程，系统创建进程都是通过fork的形式完成，其中在Kernel空间会创建kthreadd进程，还有一个就是在用户空间创建init进程（pid = 1），这个进程想必我们都非常熟悉了。
>
> 像我们启动app，或者系统应用，都需要zygote进程来孵化进程，那么zygote进程也是通过init进程来创建完成的，像系统服务的创建和启动，是通过system_server进程来管理，而system_server进程则是由zygote进程fork创建。
>
> 所以通过下面这个图，我们就能大致了解，从电源按下的那一刻到应用启动的流程。
>
> ![image.png](./Android%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B.assets/4850218d9be44c1b98c738b9760a6286tplv-k3u1fbpfcp-zoom-in-crop-mark1512000-1688180652132-8.webp)
>
> 接下来我们分析每个进程启动流程。
>
> 作者：layz4android
> 链接：https://juejin.cn/post/7232947178690412602
> 来源：稀土掘金
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

> 这个第一部分很多概念不大懂，先硬着头皮看吧。
>
> 不懂的地方：
>
> 1. BootLoader的作用、硬件电路怎么驱动运行BootLoader、BootLoader的代码存在哪里？
>
>    > 参考：
>    >
>    > [Bootloader详解，理解Bootloader看这篇就够了_「已注销」的博客-CSDN博客](https://blog.csdn.net/iduuigdg/article/details/122144655)
>    >
>    > [【嵌入式实战】STM32 Bootloader 快速实现（超详细）_stm32bootloader_HinGwenWoong的博客-CSDN博客](https://blog.csdn.net/hxj0323/article/details/108414536)
>
> 2. idle进程的作用？运行的机制？
>
>    > 参考：
>    >
>    > [Linux中的特殊进程：idle进程、init进程、kthreadd进程_JoggingPig的博客-CSDN博客](https://blog.csdn.net/joggingpig/article/details/110239518)
>    >
>    > [[linux\]进程（三）——idle进程_知了112的博客-CSDN博客](https://blog.csdn.net/u013686805/article/details/19905941)
>
> 3. fork的原理
>
>    > 参考：
>    >
>    > [操作系统——fork()_操作系统fork()_NanoNino的博客-CSDN博客](https://blog.csdn.net/qq_44198589/article/details/109828245?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-8-109828245-blog-109169941.235^v38^pc_relevant_sort_base2&spm=1001.2101.3001.4242.5&utm_relevant_index=9)
>    >
>    > [fork()函数详解_fork函数_丿安桥的博客-CSDN博客](https://blog.csdn.net/weixin_57133901/article/details/124732913?spm=1001.2101.3001.6650.7&utm_medium=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~Rate-7-124732913-blog-109169941.235^v38^pc_relevant_sort_base2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~Rate-7-124732913-blog-109169941.235^v38^pc_relevant_sort_base2&utm_relevant_index=11)

## 2 C/C++ Framework Native层

### 2.1 init进程启动分析

