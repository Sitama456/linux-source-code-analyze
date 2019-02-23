# Socket族接口层
Socket的英文原本意思是 `孔` 或 `插座`。但在计算机科学中通常被称作为 `套接字`，主要用于相同机器的不同进程间或者不同机器间的通信。Socket的使用很多网络编程的书籍都有介绍，所以本文不打算介绍Socket的使用，只讨论Socket的具体实现，所以如果对Socket不太了解的同学可以先查阅Socket相关的资料或者书籍。

## Socket族系统调用
`Soket族系统调用` 提供了一系列的接口函数供用户使用，如下：
* socket()
* bind()
* listen()
* accept()
* connect()
* recv()
* send()
* recvfrom()
* sendto()

例如 `socket()` 接口用于创建一个socket句柄，而 `bind()` 函数将一个socket绑定到指定的IP和端口上。当然，系统调用最终都会调用到内核态的某个内核函数来进行处理，在系统调用一章我们介绍过相关的原理，所以这里只会介绍一下这些系统调用最终会调用哪些内核函数。

所有的 `Socket族系统调用` 最终都会调用 `sys_socketcall()` 函数来处理用户的请求，我们来看看 `sys_socketcall()` 函数的实现：
```cpp
asmlinkage long sys_socketcall(int call, unsigned long *args)
{
    unsigned long a[6];
    unsigned long a0,a1;
    int err;

    if(call<1||call>SYS_RECVMSG)
        return -EINVAL;

    /* copy_from_user should be SMP safe. */
    if (copy_from_user(a, args, nargs[call]))
        return -EFAULT;
        
    a0=a[0];
    a1=a[1];
    
    switch(call) 
    {
        case SYS_SOCKET:
            err = sys_socket(a0,a1,a[2]);
            break;
        case SYS_BIND:
            err = sys_bind(a0,(struct sockaddr *)a1, a[2]);
            break;
        case SYS_CONNECT:
            err = sys_connect(a0, (struct sockaddr *)a1, a[2]);
            break;
        ...
    }
    return err;
}
```
从 `sys_socketcall()` 函数可以看出，根据参数 `call` 不同的值会调用不同的内核函数，譬如 `call` 的值为 `SYS_SOCKET` 时会调用 `sys_socket()` 函数，而 `call` 的值为 `SYS_BIND` 时会调用 `sys_bind()` 函数。而参数 `args` 就是在用户态给 `Socket族系统调用` 传入的参数列表，Linux内核会先把这些参数复制到内核空间。

在用户空间调用 `socket()` 系统调用时会把参数 `call` 的值设置为 `SYS_SOCKET`，所以此时真正调用的是 `sys_socket()` 内核函数 。