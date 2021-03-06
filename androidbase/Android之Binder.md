## Linux支持上进程间通信方式
1. 管道
2. 消息队列／共享内存／信号量
消息队列和管道采用存储-转发方式，即：数据先从发送方缓存区拷贝到内存开辟的缓存区中，然后再从内核缓存区拷贝到接收方缓存区。(至少有两次拷贝过程)
共享内存虽然无需拷贝，但控制复杂，难以使用

## Binder与Socket
Binder比传统的Socket方式更加高效
socket作为一款通用接口，其传输效率低、开销大，主要用在跨网络的进程间通信和本机上进程间的低速通信

首先传统IPC的接收方无法获得对方进程可靠的UID/PID（用户ID/进程ID），从而无法鉴别对方身份。Android为每个安装好的应用程序分配了自己的UID，故进程的UID是鉴别进程身份的重要标志。使用传统IPC只能由用户在数据包里填入UID/PID，但这样不可靠，容易被恶意程序利用。可靠的身份标记只有由IPC机制本身在内核中添加。其次传统IPC访问接入点是开放的，无法建立私有通道。比如命名管道的名称，system V的键值，socket的ip地址或文件名都是开放的，只要知道这些接入点的程序都可以和对端建立连接，不管怎样都无法阻止恶意程序通过猜测接收方地址获得连接。
基于以上原因，Android需要建立一套新的IPC机制来满足系统对通信方式，传输性能和安全性的要求，这就是Binder。Binder基于Client-Server通信模式，传输过程只需一次拷贝，为发送发添加UID/PID身份，既支持实名Binder也支持匿名Binder，安全性高。


## Binder通信模型
Binder框架定义了四个角色：Server，Client，ServiceManager（以后简称SMgr）以及Binder驱动。其中Server，Client，SMgr运行于用户空间，驱动运行于内核空间。

### Binder驱动
职责：进程之间Binder通信的建立，Binder在进程之间的传递，Binder引用计数管理，数据包在进程之间的传递和交互等一系列底层支持
文件操作：open() mmap() poll() ioctl()
驱动和应用程序之间定义了一套接口协议，主要功能由ioctl()接口实现，不提供read()，write()接口，因为ioctl()灵活方便，且能够一次调用实现先写后读以满足同步交互，而不必分别调用write()和read()。Binder驱动的代码位于linux目录的drivers/misc/binder.c中。


## 为什么需要进程间通讯
由于Android会为不同的进程分配各自独立的虚拟机，各自独立的虚拟机在内存分配上有不同的地址空间，所以无法通过传统的内存共享方式来实现进程间通信，

## Android中使用Binder的原因
1. 采用C/S的通讯模式。而在Linux的通信机制中，目前只有Socket支持C/S通信模式
2. 有更好的传输性能。对比于Linux通信机制
  a. socket：是一个通用接口，导致其传输效率低，开销大；
  b. 管道和消息队列：因为采用存储转发方式，所以至少需要拷贝2次数据，效率低；
  c. 共享内存：虽然在传输时没有拷贝数据，但其控制机制复杂（比如跨进程通信时，需获取对方进程的pid，得多种机制协同操作）。
3. 安全性更高。Linux的IPC机制在本身的实现中，并没有安全措施，得依赖上层协议来进行安全控制。而Binder机制的 UID/PID是由Binder机制本身在内核空间添加身份标识，安全性高；并且Binder可以建立私有通道，这是linux的通信机制所无法实现的 （Linux访问的接入点是开放的）

![binder_advantage](http://p5xecv7m0.bkt.clouddn.com/5bf63212fbc68bdf50a14ce847fea99f.png)

## 概念等
Linux的动态内核可加载模块（Loadable Kernel Module, LKM）机制：模块是具有独立功能的程序，它可以被单独编译，但不能独立运行。它在运行时被链接到内核作为内核的一部分运行。这样Android系统就可以通过动态添加一个内核模块运行在内核空间，用户进程之间通过这个内核模块作为桥梁实现通信（在 Android 系统中，这个运行在内核空间，负责各个用户进程通过 Binder 实现通信的内核模块就叫 Binder 驱动（Binder Dirver））

Linux 下的另一个概念：内存映射：Binder IPC 机制中涉及到的内存映射通过 mmap() 来实现，mmap() 是操作系统中一种内存映射的方法。内存映射简单的讲就是将用户空间的一块内存区域映射到内核空间。映射关系建立后，用户对这块内存区域的修改可以直接反应到内核空间；反之内核空间对这段区域的修改也能直接反应到用户空间。

## 通信过程
一次完整的 Binder IPC 通信过程通常是这样：
1. 首先 Binder 驱动在内核空间创建一个数据接收缓存区；
2. 接着在内核空间开辟一块内核缓存区，建立内核缓存区和内核中数据接收缓存区之间的映射关系，以及内核中数据接收缓存区和接收进程用户空间地址的映射关系；
3. 发送方进程通过系统调用 copy_from_user() 将数据 copy 到内核中的内核缓存区，由于内核缓存区和接收进程的用户空间存在内存映射，因此也就相当于把数据发送到了接收进程的用户空间，这样便完成了一次进程间的通信。

Binder 通信过程：
1. 首先，一个进程使用 BINDER_SET_CONTEXT_MGR 命令通过 Binder 驱动将自己注册成为 ServiceManager；
2. Server 通过驱动向 ServiceManager 中注册 Binder（Server 中的 Binder 实体），表明可以对外提供服务。驱动为这个 Binder 创建位于内核中的实体节点以及 ServiceManager 对实体的引用，将名字以及新建的引用打包传给 ServiceManager，ServiceManger 将其填入查找表。
3. Client 通过名字，在 Binder 驱动的帮助下从 ServiceManager 中获取到对 Binder 实体的引用，通过这个引用就能实现和 Server 进程的通信

## Binder工作机制


## 参考
https://mp.weixin.qq.com/s/gZdQ_KTF5h8Xd_s5KI5Iew
