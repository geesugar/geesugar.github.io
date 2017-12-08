---
title: linux共享内存
date: 2016-06-11 20:43:32
tags: 
---
## 概述
共享内存是可用IPC形式中最快的。使用共享内存需要掌握多个进程之间对给定内存区的*同步访问*。通常，*信号量*被用来实现对共享内存访问的同步。

## 内存布局

## 权限结构
内核给每个IPC对象维护一个信息结构ipc_perm,该结构规定了权限和所有者。在`sys/ipc.h`中定义如下：
```c
struct ipc_perm
  {
    __key_t __key;          /* Key.  */
    __uid_t uid;            /* Owner's user ID.  */
    __gid_t gid;            /* Owner's group ID.  */
    __uid_t cuid;           /* Creator's user ID.  */
    __gid_t cgid;           /* Creator's group ID.  */
    unsigned short int mode;        /* Read/write permission.  */
    unsigned short int __pad1;
    unsigned short int __seq;       /* Sequence number.  */
    unsigned short int __pad2;
    unsigned long int __unused1;
    unsigned long int __unused2;
  };  
```