---
title: Android IPC传输大数据
categories:
  - Android
tags:
  - Android
  - Binder
description: 讨论APP如何能解决IPC传输大数据问题以及原理。
comments: true
abbrlink: 6141f281
date: 2021-10-12 22:17:46
---
<!--more-->
<meta name="referrer" content="no-referrer"/>

# 前言


## IPC 方式


### Activity
```java
Intent callIntent = new Intent(Intent.ACTION_CALL, Uri.parse("tel:021-1234567" ); 
mContext.startActivity(callIntent);
```


### Content Provider
Android应用程序可以使用文件或SqlLite数据库来存储数据。
Content Provider提供了一种在多个应用程序之间数据共享的方式（跨进程共享数据），应用程序可以利用Content Provider完成下面的工作：

- 查询数据
- 修改数据
- 添加数据
- 删除数据



### Broadcast
```java
Intent intent = new Intent("com.android.ACTION_TEST");
intent.putExtra("value","content");
mContext.sendBroadcast(intent);
```


### LocalSocket
#### 客户端
```java
客户端LocalSocket代码

//创建对象
LocalSocket localSocket = new LocalSocket();
//连接socketServerSocket
localSocket.connect(new LocalSocketAddress(String addrStr));
获取localSocket的输入输出流：
outputStream = localSocket.getOutputStream();
inputStream = localSocket.getInputStream();
写入数据：
outputStream.write("数据".getBytes());
循环接收数据：
try {    
	int readed = inputStream.read();
	int size = 0;
	byte[] bytes = new byte[0];
	while (readed != -1) {
	  byte[] copy = new byte[500000]; 
	  System.arraycopy(bytes, 0, copy, 0, bytes.length);
	  bytes = copy;
	  bytes[size++] = (byte) readed; 
	  //以换行符标识成结束
	  if ('\n' == (byte) readed) {
	      String resultStr = new String(bytes, 0, size); 
	      break;                                                         				     
	} 
	readed = inputStream.read();
	}
} catch (IOException e) {
	return false;
}
```


#### 服务端
```java
服务端LocalServerSocket代码

//初始化
try {
    //socketAddress需跟localSocket地址一致，否则无法连接上
    serverSocket = new LocalServerSocket(socketAddress);
} catch (IOException e) {
    LoggerHelper.e("Server to establish connection exception:" + e.toString());
    e.printStackTrace();
    return false;
}
try {
    //获取接收的LocalSocket
    localSocket = serverSocket.accept();
    //设置缓冲大小
    localSocket.setReceiveBufferSize(ConstantConfig.BUFFER_SIZE);
    localSocket.setSendBufferSize(ConstantConfig.BUFFER_SIZE);
} catch (IOException e) {
    e.printStackTrace();
    LoggerHelper.d("Waiting to be linked to a bug,error:" + e.toString());
    return false;
}
获取输入输出流一致：

if (localSocket != null) {
    try {
        inputStream = localSocket.getInputStream();
        outputStream = localSocket.getOutputStream();
        /** 允许一直接收数据，一直到连接被断开，则认为应用端退出，自己也退出 */
        while (isLock && receiveData()) ;
    } catch (IOException e) {
        LoggerHelper.e("Get stream exception:" + e.toString());        e.printStackTrace();
        return false;
    }
}
```


### AIDL Service
aidl的基本用法请自行百度
​

## 方案

- 文件操作

如果用文件写入磁盘的方案肯定是没问题的，但是IO操作效率低下。

- IPC（上述几种方式）

首先Binder性能是可以，用起来也方便，但是Binder对传输数据大小有限制， 传输超过1M的文件就会报android.os.TransactionTooLargeException异常。
​

不妨先思考一个问题，在Android系统中，APP端View视图的数据是如何传递SurfaceFlinger服务的呢？View绘制的数据最终是按照一帧一帧显示到屏幕的，而每一帧都会占用一定的存储空间，在APP端执行draw的时候，数据很明显是要绘制到APP的进程空间，但是视图窗口要经过SurfaceFlinger图层混排才会生成最终的帧，而SurfaceFlinger又运行在另一个独立的服务进程，那么View视图的数据是如何在两个进程间传递的呢，普通的Binder通信肯定不行，因为Binder不太适合这种数据量较大的通信，那么View数据的通信采用的是什么IPC手段呢？研究过SurfaceFlinger的同学都知道，答案就是共享内存，更精确的说是匿名共享内存。
​

# Ashmem
Android中的匿名共享内存(Ashmem)是基于Linux共享内存的，借助Binder+文件描述符(FileDescriptor)实现了共享内存的传递。它可以让多个进程操作同一块内存区域，并且除了物理内存限制，没有其他大小限制。相对于Linux的共享内存，Ashmem对内存的管理更加精细化，并且添加了互斥锁。Java层在使用时需要用到MemoryFile，它封装了native代码。Android平台上共享内存通常的做法如下：

- 进程A通过MemoryFile创建共享内存，得到fd(FileDescriptor)
- 进程A通过fd将数据写入共享内存
- 进程A将fd封装成实现Parcelable接口的ParcelFileDescriptor对象，通过Binder将ParcelFileDescriptor对象发送给进程B
- 进程B获从ParcelFileDescriptor对象中获取fd，从fd中读取数据

​

## 使用代码
```java
/**
 * IMemoryAidlInterface.aidl
 */
package com.journeyOS.ipcbigdata;
import android.os.ParcelFileDescriptor;

interface IMemoryAidlInterface {
    ParcelFileDescriptor getParcelFileDescriptor();
}


/**
 * MemoryFetchService.java
 */
public class MemoryFetchService extends Service {
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return new MemoryFetchStub();
    }
    static class MemoryFetchStub extends IMemoryAidlInterface.Stub {
        @Override
        public ParcelFileDescriptor getParcelFileDescriptor() throws RemoteException {
            MemoryFile memoryFile = null;
            try {
                memoryFile = new MemoryFile("test_memory", 1024);
                memoryFile.getOutputStream().write(new byte[]{1, 2, 3, 4, 5});
                Method method = MemoryFile.class.getDeclaredMethod("getFileDescriptor");
                FileDescriptor des = (FileDescriptor) method.invoke(memoryFile);
                return ParcelFileDescriptor.dup(des);
            } catch (Exception e) {}
            return null;
     }}}

     }

/**
 * TestActivity.java
 */
Intent intent = new Intent(MainActivity.this, MemoryFetchService.class);
bindService(intent, new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {

        byte[] content = new byte[10];
        IMemoryAidlInterface iMemoryAidlInterface
                = IMemoryAidlInterface.Stub.asInterface(service);
        try {
            ParcelFileDescriptor parcelFileDescriptor = iMemoryAidlInterface.getParcelFileDescriptor();
            FileDescriptor descriptor = parcelFileDescriptor.getFileDescriptor();
            FileInputStream fileInputStream = new FileInputStream(descriptor);
            fileInputStream.read(content);
        } catch (Exception e) {
        }}

    @Override
    public void onServiceDisconnected(ComponentName name) {

    }
}, Service.BIND_AUTO_CREATE);   
```
封装好的或者更详细的代码可以参考：[https://github.com/CCLCM/AndroidSharedMemoryDemo](https://github.com/CCLCM/AndroidSharedMemoryDemo)
本文的重心在于分析原理，而不是使用方法。
以上是应用层使用匿名共享内存的方法，关键点就是文件描述符（FileDescriptor）的传递，文件描述符是Linux系统中访问与更新文件的主要方式。从MemoryFile字面上看出，共享内存被抽象成了文件，不过本质也是如此，就是在tmpfs临时文件系统中创建一个临时文件，**（只是创建了节点，而没有看到实际的文件）** 该文件与Ashmem驱动程序创建的匿名共享内存对应，可以直接去proc/pid下查看：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1759879/1631692887527-a18a6a37-ca9c-4f9b-9e43-f5a1a6125010.png#clientId=uf347de8d-52cd-4&from=paste&id=u2aae91d9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=404&originWidth=1166&originalType=url&ratio=1&size=685106&status=done&style=none&taskId=u149735ca-c82d-48a7-91e9-526b50d3a59)


## MemoryFile
MemoryFile是android在最开始就引入的一套框架，其内部实际上是封装了android特有的内存共享机制Ashmem匿名共享内存，简单来说，Ashmem在Android内核中是被注册成一个特殊的字符设备，Ashmem驱动通过在内核的一个自定义slab缓冲区中初始化一段内存区域，然后通过mmap把申请的内存映射到用户的进程空间中（通过tmpfs），就可以在用户进程中使用这里申请的内存了，另外Ashmem的一个特性就是可以在系统内存不足的时候，回收掉被标记为"unpin"的内存(可自行查看网上资料)，MemoryFile也可以通过Binder跨进程调用来让两个进程共享一段内存区域。由于整个申请内存的过程并不再Java层上，可以很明显的看出使用MemoryFile申请的内存实际上是并不会占用Java堆内存的。
​

### native_mmap
先看下MemoryFile的构造函数：
```java
public MemoryFile(String name, int length) throws IOException {
    mLength = length;
    mFD = native_open(name, length);
    if (length > 0) {
        mAddress = native_mmap(mFD, length, PROT_READ | PROT_WRITE);
    } else {
        mAddress = 0;
    }
}
```
可以看到 Java层只是简单的封装，具体实现在native层 。

- 通过native_open调用ashmem_create_region创建共享内存。
- 通过native_mmap调用mmap将共享内存映射到当前进程空间，之后Java层就能利用FileDescriptor，像访问文件一样访问共享内存。
```c
static jint android_os_MemoryFile_mmap(JNIEnv* env, jobject clazz, jobject fileDescriptor,
        jint length, jint prot)
{
    int fd = jniGetFDFromFileDescriptor(env, fileDescriptor);
    //系统调用mmap，分配内存
    jint result = (jint)mmap(NULL, length, prot, MAP_SHARED, fd, 0);
    if (!result)
        jniThrowException(env, "java/io/IOException", "mmap failed");
    return result;
}          
```


### ashmem_create_region
```c
static jobject android_os_MemoryFile_open(JNIEnv* env, jobject clazz, jstring name, jint length)
{
    const char* namestr = (name ? env->GetStringUTFChars(name, NULL) : NULL);

    int result = ashmem_create_region(namestr, length);

    if (name)
        env->ReleaseStringUTFChars(name, namestr);

    if (result < 0) {
        jniThrowException(env, "java/io/IOException", "ashmem_create_region failed");
        return NULL;
    }

    return jniCreateFileDescriptor(env, result);
}
```
ashmem_create_region这个函数是如何向Linux申请一块共享内存的呢？
```c
int ashmem_create_region(const char *name, size_t size)
{
    int fd, ret;
    fd = open(ASHMEM_DEVICE, O_RDWR);
    if (fd < 0)
        return fd;
        if (name) {
        char buf[ASHMEM_NAME_LEN];
        strlcpy(buf, name, sizeof(buf));
        ret = ioctl(fd, ASHMEM_SET_NAME, buf);
        if (ret < 0)
            goto error;
    }

    ret = ioctl(fd, ASHMEM_SET_SIZE, size);
    if (ret < 0)
        goto error;

    return fd;

error:
    close(fd);
    return ret;
}
```


### misc(字符设备)
ASHMEM_DEVICE其实就是抽象的共享内存设备，它是一个杂项设备（字符设备的一种），在驱动加载之后，就会在/dev下创建ashem文件，之后用户就能够访问该设备文件，同一般的设备文件不同，它仅仅是通过内存抽象的，同普通的磁盘设备文件、串行端口字段设备文件不一样：
```c
#define ASHMEM_DEVICE   "/dev/ashmem"   
static struct miscdevice ashmem_misc = {
    .minor = MISC_DYNAMIC_MINOR,
    .name = "ashmem",
    .fops = &ashmem_fops,
};
```
接着进入驱动看一下，如何申请共享内存，open函数很普通，主要是创建一个ashmem_area对象。
​

#### open
```c
static int ashmem_open(struct inode *inode, struct file *file)
{
    struct ashmem_area *asma;
    int ret;

    ret = nonseekable_open(inode, file);
    if (unlikely(ret))
        return ret;

    asma = kmem_cache_zalloc(ashmem_area_cachep, GFP_KERNEL);
    if (unlikely(!asma))
        return -ENOMEM;

    INIT_LIST_HEAD(&asma->unpinned_list);
    memcpy(asma->name, ASHMEM_NAME_PREFIX, ASHMEM_NAME_PREFIX_LEN);
    asma->prot_mask = PROT_MASK;
    file->private_data = asma;
    return 0;
}
```


#### ioctl
利用ashmem_ioctl设置共享内存的大小，
```c
static long ashmem_ioctl(struct file *file, unsigned int cmd, unsigned long arg)
{
    struct ashmem_area *asma = file->private_data;
    long ret = -ENOTTY;
    switch (cmd) {
    ...
    case ASHMEM_SET_SIZE:
        ret = -EINVAL;
        if (!asma->file) {
            ret = 0;
            asma->size = (size_t) arg;
        }
        break;
    ...
    }
   return ret;
}   
```


#### mmap
可以看到，其实并未真正的分配内存，这也符合Linux的风格，只有等到真正的使用的时候，才会通过缺页中断分配内存，接着mmap函数，它会分配内存吗？
```c
static int ashmem_mmap(struct file *file, struct vm_area_struct *vma)
{
    struct ashmem_area *asma = file->private_data;
    int ret = 0;
    mutex_lock(&ashmem_mutex);
    ...
    if (!asma->file) {
        char *name = ASHMEM_NAME_DEF;
        struct file *vmfile;

        if (asma->name[ASHMEM_NAME_PREFIX_LEN] != '\0')
            name = asma->name;
        // 这里创建的临时文件其实是备份用的临时文件。
        // 有文章说只对内核态可见，用户态不可见，我们也没有办法通过命令查询到，可以看做是个隐藏文件，用户空间看不到！！
        // 校准真正操作的文件
        vmfile = shmem_file_setup(name, asma->size, vma->vm_flags);
        asma->file = vmfile;
    }
    get_file(asma->file);
    if (vma->vm_flags & VM_SHARED)
        shmem_set_file(vma, asma->file);
    else {
        if (vma->vm_file)
            fput(vma->vm_file);
        vma->vm_file = asma->file;
    }
    vma->vm_flags |= VM_CAN_NONLINEAR;
out:
    mutex_unlock(&ashmem_mutex);
    return ret;
}
```
其实这里就复用了Linux的共享内存机制，虽然说是匿名共享内存，但底层其实还是给共享内存设置了名称（前缀ASHMEM_NAME_PREFIX+名字），如果名字未设置，那就默认使用ASHMEM_NAME_PREFIX作为名称。不过，在这里没直接看到内存分配的函数。但是有两个函数shmem_file_setup与shmem_set_file很重要，也是共享内存比较不好理解的地方，shmem_file_setup是原生linux的共享内存机制，不过Android也修改Linux共享内存的驱动代码，匿名共享内存其实就是在Linux共享内存的基础上做了改进。
```c
struct file *shmem_file_setup(char *name, loff_t size, unsigned long flags)
{
    int error;
    struct file *file;
    struct inode *inode;
    struct dentry *dentry, *root;
    struct qstr this;   
    error = -ENOMEM;
    this.name = name;
    this.len = strlen(name);
    this.hash = 0; /* will go */
    root = shm_mnt->mnt_root;
    dentry = d_alloc(root, &this);//分配dentry cat/proc/pid/maps可以查到
    error = -ENFILE;
    file = get_empty_filp();      //分配file
    error = -ENOSPC;
    inode = shmem_get_inode(root->d_sb, S_IFREG | S_IRWXUGO, 0, flags);//分配inode，分配成功就好比建立了文件，也许并未存在真实文件映射
    d_instantiate(dentry, inode);//绑定
    inode->i_size = size;
    inode->i_nlink = 0; /* It is unlinked */
        // 文件操作符，这里似乎真的是不在内存里面创建什么东西？？？
    init_file(file, shm_mnt, dentry, FMODE_WRITE | FMODE_READ,
          &shmem_file_operations);//绑定，并指定该文件操作指针为shmem_file_operations
    ...
}
```
通过shmem_file_setup在tmpfs临时文件系统中创建一个临时文件（也许只是内核中的一个inode节点），该文件与Ashmem驱动程序创建的匿名共享内存对应，不过用户态并不能看到该临时文件，之后就能够使用该临时文件了，注意**共享内存机制真正使用map的对象其实是这个临时文件，而不是ashmem设备文件，这里之所以是一次mmap，主要是通过vma->vm_file = asma->file完成map对象的替换，当映射的内存引起缺页中断的时候，就会调用shmem_file_setup创建的对象的函数，而不是ashmem的**，看下临时文件的对应的hook函数。
```c
void shmem_set_file(struct vm_area_struct *vma, struct file *file)
{
    if (vma->vm_file)
        fput(vma->vm_file);
    vma->vm_file = file;
    vma->vm_ops = &shmem_vm_ops;
}
```


### writeBytes
到这里回到之前的MemoryFile，看一下写操作：
```java
public void writeBytes(byte[] buffer, int srcOffset, int destOffset, int count)
        throws IOException {
    if (isDeactivated()) {
        throw new IOException("Can't write to deactivated memory file.");
    }
    if (srcOffset < 0 || srcOffset > buffer.length || count < 0
            || count > buffer.length - srcOffset
            || destOffset < 0 || destOffset > mLength
            || count > mLength - destOffset) {
        throw new IndexOutOfBoundsException();
    }
    native_write(mFD, mAddress, buffer, srcOffset, destOffset, count, mAllowPurging);
}
```
进入native代码
```c
static jint android_os_MemoryFile_write(JNIEnv* env, jobject clazz,
        jobject fileDescriptor, jint address, jbyteArray buffer, jint srcOffset, jint destOffset,
        jint count, jboolean unpinned)
{
    int fd = jniGetFDFromFileDescriptor(env, fileDescriptor);
    if (unpinned && ashmem_pin_region(fd, 0, 0) == ASHMEM_WAS_PURGED) {
        ashmem_unpin_region(fd, 0, 0);
        return -1;
    }
    env->GetByteArrayRegion(buffer, srcOffset, count, (jbyte *)address + destOffset);
    if (unpinned) {
        ashmem_unpin_region(fd, 0, 0);
    }
    return count;
}
```
在内核中，一块内存对应的数据结构是ashmem_area：
```c
struct ashmem_area {
    char name[ASHMEM_FULL_NAME_LEN];/* optional name for /proc/pid/maps */
    struct list_head unpinned_list; /* list of all ashmem areas */
    struct file *file;      /* the shmem-based backing file */
    size_t size;            /* size of the mapping, in bytes */
    unsigned long prot_mask;    /* allowed prot bits, as vm_flags */
};
```
当使用Ashmem分配了一块内存，部分不被使用时，就可以将这块内存unpin掉，内核可以将unpin对应的物理页面回收，回收后的内存还可以再次被获得(通过缺页handler)，因为unpin操作并不会改变已经mmap的地址空间，不过MemoryFile只会操作整个共享内存，而不会分块访问，所以pin与unpin对于它没多大意义，可以看做整个区域都是pin或者unpin的，首次通过env->GetByteArrayRegion访问会引发缺页中断，进而调用tmpfs 文件的相应操作，分配物理页，在Android现在的内核中，缺页中断对应的vm_operations_struct中的函数是fault，在共享内存实现中，对应的是shmem_fault如下。
```c
static struct vm_operations_struct shmem_vm_ops = {
    .fault      = shmem_fault,
    
#ifdef CONFIG_NUMA
    .set_policy     = shmem_set_policy,
    .get_policy     = shmem_get_policy,
#endif
};
```
当mmap的tmpfs文件引发缺页中断时， 就会调用shmem_fault函数。
```c
static int shmem_fault(struct vm_area_struct *vma, struct vm_fault *vmf)
{
    struct inode *inode = vma->vm_file->f_path.dentry->d_inode;
    int error;
    int ret;

    if (((loff_t)vmf->pgoff << PAGE_CACHE_SHIFT) >= i_size_read(inode))
        return VM_FAULT_SIGBUS;

    error = shmem_getpage(inode, vmf->pgoff, &vmf->page, SGP_CACHE, &ret);
    if (error)
        return ((error == -ENOMEM) ? VM_FAULT_OOM : VM_FAULT_SIGBUS);

    return ret | VM_FAULT_LOCKED;
}
```
到这里，就可以看到会调用shmem_getpage函数分配真实的物理页，具体的分配策略比较复杂，不在分析。
​

### FileDescriptor
原生Linux共享内存是通过传递已知的key来处理的（附录会讲解），但Android是通过Binder传递文件描述符来处理，Binder驱动通过当前进程的fd找到对应的文件，然后为目标进程新建fd，并传递给目标进程，核心就是把进程A中的fd转化成进程B中的fd。**即client进程在收到此fd之后是可以作为自己进程已经打开的fd来是使用的。**看一下Android中binder的实现：
```c
void binder_transaction(){
   ...
        case BINDER_TYPE_FD: {
        int target_fd;
        struct file *file;
        //1. 可以根据fd在当前进程获取到file，多个进程打开同一文件，在内核中对应的file是一样
        file = fget(fp->handle);
        //2. 为目标进程获取空闲fd
        target_fd = task_get_unused_fd_flags(target_proc, O_CLOEXEC);
        //3. 将目标进程的空闲fd与file绑定
        task_fd_install(target_proc, target_fd, file);
        fp->handle = target_fd;
    } break;    
    ...
 }

// 从当前进程打开的files中找到file在内核中的实例
struct file *fget(unsigned int fd)
{
    struct file *file;
    struct files_struct *files = current->files;
    rcu_read_lock();
    file = fcheck_files(files, fd);
    rcu_read_unlock();
    return file;
}


static void task_fd_install(
    struct binder_proc *proc, unsigned int fd, struct file *file)
{
    struct files_struct *files = proc->files;
    struct fdtable *fdt;
    if (files == NULL)
        return;
    spin_lock(&files->file_lock);
    fdt = files_fdtable(files);
    rcu_assign_pointer(fdt->fd[fd], file);
    spin_unlock(&files->file_lock);
}
```
如果fp->type是**BINDER_TYPE_FD**, 那么说明这次要传递的是一个跨进程的FD：

- 调用file(类型是struct file * 即Linux内核规定的file数据结构) = fget(fp->handle);
- 对于struct file的解释: 文件结构体代表一个打开的文件，系统中的每个打开的文件在内核空间都有一个关联的struct file。它由内核在打开文件时创建，并传递给在文件上进行操作的任何函数。在文件的所有实例都关闭后，内核释放这个数据结构。在内核创建和驱动源码中，struct file的指针通常被命名为file或filp。
- file = fget(fp->handle), fget(…)函数基于输入的**fd来得到此fd”指向”的file数据结构体： current(当前进程的进程结构体)->files->fd[fd]**， 此时file里保存的就是fd对应的file结构体的指针。
- 调用security_binder_transfer_file(proc->tsk, target_proc->tsk, file)基于**检查目标进程(target_proc)是否具有访问文件file的权限。如果有的话，那么就允许源进程将它传递给目标进程访问。
- target_fd = task_get_unused_fd_flags(target_proc, O_CLOEXEC); **在target进程中找一个该进程还没有被使用的Fd编号。**
- task_fd_install(target_proc, target_fd, file), **将得到targetFd和file指针指向的file结构体在target进程中挂接起来。**
- fp->handle = target_fd.
- task_fd_install(struct binder_proc *proc, unsigned int fd, struct file *file)**函数将file指向的file结构体和fd关联起来，会在proc指定的进程中生效。**
   - **binder_proc是binder驱动自己定义的结构体，是一个进程在binder驱动内的信息保存者, 每个进程只会open binder一次，binder进而将这个特殊的结构体和该进程打开binder时的fd对应的file结构体对应起来。**
   - struct files_struct *files = proc->files:
      - 对一个进程来说，其binder_proc对象被建立的时机是**binder_open(…)的时候**
      - struct binder_proc *proc = kzalloc(sizeof(*proc), GFP_KERNEL); 为binder_proc申请一块内存
      - proc->tsk = current; 将当前进程的task struct指针保存在binder_proc的task中
      - proc->pid = current->group_leader->pid; 保存进程Id
      - filp->private_data = proc; 将申请出来的binder_proc对象指针保存在打开/dev/binder对应的file结构的private_data中这是binder驱动能够识别不同进程的关键, binder驱动在进程打开他时给其一个特殊的binder_proc对象，进程将此对象保存在打开binder文件的file结构体中, 后面只要还使用这个file结构体和binder驱动交互, binder启动就可以根据private_data中的binder_proc对象来识别该进程(private_data的说明: 系统在调用驱动程序的open方法前将这个指针置为NULL。驱动程序可以将这个字段用于任意目的，也可以忽略这个字段。驱动程序可以用这个字段指向已分配的数据，但是一定要在内核释放file结构前的release方法中清除它。即这个字段是由驱动来根据自己的需要进行扩展的)
      - 在binder_mmap时, 会将binder_proc对象的files指针赋值: proc->files = **get_files_struct(proc->tsk(当前进程对应的task struct)) 得到此进程的files结构体数组的指针**
   - fdt = files_fdtable(files); files_fdtable: 去读取files所在进程(目的进程)的文件描述表
   - rcu_assign_pointer(fdt->fd[fd], file); 将file指针和fd关联起来. 这样在target process中就可以使用这个fd操作另外一个proc中打开的file结构体了

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1759879/1631696407348-62335e83-c944-4cad-bf38-57ffe184eadf.png#clientId=uf347de8d-52cd-4&from=paste&id=u48f977ad&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1657&originWidth=2213&originalType=url&ratio=1&size=710076&status=done&style=none&taskId=u025ecedf-ee77-477e-974d-bf330e73b6d)
​

### 优缺点

- 优点：内存不会占用Dalvik Heap与Native Heap，不会导致OOM
- 缺点：如果肆意使用，会导致系统资源不足，性能下降



[dumpsys meminfo使用命令](https://blog.csdn.net/c_z_w/article/details/85336283)
![memory.png](https://cdn.nlark.com/yuque/0/2021/png/1759879/1631696638336-daeba8ac-7714-4ce1-8f56-1b86edbe7453.png#clientId=uf347de8d-52cd-4&from=ui&id=VBIlo&margin=%5Bobject%20Object%5D&name=memory.png&originHeight=844&originWidth=747&originalType=binary&ratio=1&size=467982&status=done&style=none&taskId=u39fd7d5e-ae0e-4e62-9009-e5b7227637a)


## putBinder
现在我们继续分析网上传得很神奇的putBinder可以传输大图片到底是为何。
​

### 使用代码
```java
//发送端
Intent intent = new Intent(this, SecondActivity.class);
Bundle bundle = new Bundle();
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR2) {
    bundle.putBinder("bitmap", new ImageBinder(mBitmap));
}
intent.putExtras(bundle);
startActivity(intent);

//接收端
Bundle bundle = getIntent().getExtras();
if (bundle != null) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR2) {
        ImageBinder imageBinder = (ImageBinder) bundle.getBinder("bitmap");
        Bitmap bitmap = imageBinder.getBitmap();
        mTv.setText(String.format(("bitmap大小为%dkB"), bitmap.getByteCount() / 1024));
        mIv.setImageBitmap(bitmap);
    }
}
```


### 源码
#### writeToParcel
```java
int startActivity(..., Intent intent, ...) {
    Parcel data = Parcel.obtain();
    ......  
    intent.writeToParcel(data, 0);    
    ......
    mRemote.transact(START_ACTIVITY_TRANSACTION, data, reply, 0);  
    ......
}

public void writeToParcel(Parcel out, int flags) {
    ......    
    out.writeBundle(mExtras);
}

public void writeToParcel(Parcel out, int flags) {    
    ......    
    out.writeBundle(mExtras);
}
```
writeToParcel函数，其实就是给Intent里的Bundle写到Parcel了。
#### writeBundle
继续往下走，看Bundle怎么写到Parcel的，原来是调到了Bundle的writeToParcel函数。
```java
public final void writeBundle(Bundle val) {    
    val.writeToParcel(this, 0);
}
```
#### writeToParcelInner
继续往下走，又调到了writeToParcelInner。
```java
public void writeToParcel(Parcel parcel, int flags) {    
    final boolean oldAllowFds = parcel.pushAllowFds(mAllowFds); 
    super.writeToParcelInner(parcel, flags);    
    parcel.restoreAllowFds(oldAllowFds);
}
```
#### pushAllowFds
这不就是上文我们提到的fd吗，继续往下看。
```java
bool Parcel::pushAllowFds(bool allowFds) {    
    const bool origValue = mAllowFds;    
    if (!allowFds) {        
        mAllowFds = false;    
    }    
    return origValue;
}
```
> 返回true，不再深入分析

#### writeToParcelInner
```java
void writeToParcelInner(Parcel parcel, int flags) {    
    ......    
    parcel.writeArrayMapInternal(mMap);
}

void writeArrayMapInternal(ArrayMap<String, Object> val) {      
    final int N = val.size();   
    writeInt(N);    
    for (int i = 0; i < N; i++) {        
        writeString(val.keyAt(i));        
        writeValue(val.valueAt(i));    
    }
}

public final void writeValue(Object v) {    
    ......     
    else if (v instanceof Parcelable) {        
        writeInt(VAL_PARCELABLE);        
        writeParcelable((Parcelable) v, 0);    
    }     
    ......
}

void writeParcelable(Parcelable p, int parcelableFlags) {    
    writeParcelableCreator(p);    
    p.writeToParcel(this, parcelableFlags);
}
```
就是在一个for循环里给map的key和value依次写到parcel。因为Bitmap是Parcelable的，所以我们只关注这个分支，这又调到了Bitmap的writeToParcel函数。
```java
void writeParcelable(Parcelable p, int parcelableFlags) {    
    writeParcelableCreator(p);    
    p.writeToParcel(this, parcelableFlags);
}
```
进入了native层
```java
jboolean Bitmap_writeToParcel(JNIEnv* env, jobject, ...) {    
    android::Bitmap* androidBitmap = reinterpret_cast<Bitmap*>(bitmapHandle);    
    androidBitmap->getSkBitmap(&bitmap);  
     
    // 往parcel里写Bitmap的各种配置参数    
    int fd = androidBitmap->getAshmemFd();    
    if (fd >= 0 && !isMutable && p->allowFds()) {        
        status = p->writeDupImmutableBlobFileDescriptor(fd);        
        return JNI_TRUE;    
    }    
     
    android::Parcel::WritableBlob blob;    
    status = p->writeBlob(size, mutableCopy, &blob);    
    const void* pSrc =  bitmap.getPixels();    
    memcpy(blob.data(), pSrc, size);
}
```
这里首先拿到native层的Bitmap对象，叫androidBitmap，然后拿到对应的SkBitmap。先看bitmap里带不带ashmemFd，如果带，并且这个Bitmap不能改，并且Parcel是允许带fd的话，就给fd写到parcel里，然后返回。否则的话继续往下，先有个WriteBlob对象，通过writeBlob函数给这个blob在parcel里分配了一块空间，然后给bitmap拷贝到这块空间里。我们看这个writeBlob函数。
```java
status_t Parcel::writeBlob(size_t len, bool mutableCopy, WritableBlob* outBlob) {    
    if (!mAllowFds || len <= BLOB_INPLACE_LIMIT) {        
        status = writeInt32(BLOB_INPLACE);        
        void* ptr = writeInplace(len);        
        outBlob->init(-1, ptr, len, false);        
        return NO_ERROR;    
    }    
    int fd = ashmem_create_region("Parcel Blob", len);    
    void* ptr = mmap(NULL, len, ..., MAP_SHARED, fd, 0);    
    ......    
    status = writeFileDescriptor(fd, true);    
    outBlob->init(fd, ptr, len, mutableCopy);    
    return NO_ERROR;
}
```
这个writeBlob函数，首先看如果不允许带fd，或者这个数据小于16K，就直接在parcel的缓冲区里分配一块空间来保存这个数据。不然的话呢，就另外开辟一个ashmem，映射出一块内存，数据就保存在ashmem的内存里，parcel里只写个fd就好了，这样就算数据量很大，parcel自己的缓冲区也不用很大。
​

看到这里就明白了，其实还有用了Ashmen机制。
​

# 总结
Android匿名共享内存是基于Linux共享内存的，都是在tmpfs文件系统上新建文件，并将其映射到不同的进程空间，从而达到共享内存的目的，只是Android在Linux的基础上进行了改造，并借助Binder+fd文件描述符实现了共享内存的传递。
​

# 附录
## Linux共享内存
首先看一下两个关键函数，

- int shmget(key_t key, size_t size, int shmflg); 该函数用来创建共享内存
- void *shmat(int shm_id, const void *shm_addr, int shmflg); 要想访问共享内存，必须将其映射到当前进程的地址空间
> 其中key_t是共享内存的唯一标识，可以说，Linux的共享内存其实是**有名**共享内存，而名字就是key，具体用法如下

### 读取进程
```java
int main()  
{  
    void *shm = NULL;//分配的共享内存的原始首地址  
    struct shared_use_st *shared;//指向shm  
    int shmid;//共享内存标识符  
    //创建共享内存  
    shmid = shmget((key_t)12345, sizeof(struct shared_use_st), 0666|IPC_CREAT);   
    //将共享内存映射到当前进程的地址空间  
    shm = shmat(shmid, 0, 0);
    //设置共享内存  
    shared = (struct shared_use_st*)shm;  
    shared->written = 0;  
    //访问共享内存
    while(1) {
        if(shared->written != 0) { 
            printf("You wrote: %s", shared->text);
             if(strncmp(shared->text, "end", 3) == 0)  
                break;
            }}
    //把共享内存从当前进程中分离  
    if(shmdt(shm) == -1) { }  
    //删除共享内存  
    if(shmctl(shmid, IPC_RMID, 0) == -1) {  }  
    exit(EXIT_SUCCESS);  
}  
```
### 写进程
```java
int main()  
{  
    void *shm = NULL;  
    struct shared_use_st *shared = NULL;  
    char buffer[BUFSIZ + 1];//用于保存输入的文本  
    int shmid;  
    //创建共享内存  
    shmid = shmget((key_t) 12345, sizeof(struct shared_use_st), 0666|IPC_CREAT);  
    //将共享内存连接到当前进程的地址空间  
    shm = shmat(shmid, (void*)0, 0);  
    printf("Memory attached at %X\n", (int)shm);  
    //设置共享内存  
    shared = (struct shared_use_st*)shm;  
    while(1)//向共享内存中写数据  
    {  
        //数据还没有被读取，则等待数据被读取,不能向共享内存中写入文本  
        while(shared->written == 1)  
        {  
            sleep(1);  
        }  
        //向共享内存中写入数据  
        fgets(buffer, BUFSIZ, stdin);  
        strncpy(shared->text, buffer, TEXT_SZ);  
        shared->written = 1;  
        if(strncmp(buffer, "end", 3) == 0)  
            running = 0;  
    }  
    //把共享内存从当前进程中分离  
    if(shmdt(shm) == -1) {    }  
    sleep(2);  
    exit(EXIT_SUCCESS);  
} 
```
可以看到，Linux共享内存通信效率非常高，进程间不需要传递数据，便可以直接访问，缺点也很明显，Linux共享内存没有提供同步的机制，在使用时，要借助其他的手段来处理进程间同步。
​

# 参考
[Android 共享内存实现跨进程大文件传输](https://blog.csdn.net/ChaoLi_Chen/article/details/106105894?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.no_search_link)
[如何使用 AIDL 实现跨进程传输一个2M大小的文件](https://blog.csdn.net/A_pyf/article/details/119255871?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.no_search_link)
[Android 跨进程传输大图片](https://blog.csdn.net/qq_27885521/article/details/100572835?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link)
