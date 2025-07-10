# 传统IPC通信方式详解

## 1. Socket通信

### 1.1 基本概念
```
Socket通信模型
.
├── 客户端          # 创建Socket连接
├── 服务端          # 监听端口
└── 传输层协议       # TCP/UDP
```

### 1.2 核心特点
- 跨网络通信能力
- 全双工通信
- 协议开销较大
- 需要序列化/反序列化

### 1.3 Android中的应用
- 网络通信
- 跨设备通信
- 复杂协议场景

### 1.4 实现原理
1. 系统层实现：
```
Linux内核Socket实现
.
├── socket结构体     # 包含协议、状态、缓冲区等信息
├── 文件系统抽象      # 通过文件描述符访问
├── 协议栈处理       # TCP/IP协议处理
└── 设备驱动层       # 网卡驱动交互
```

2. 内核关键流程：
- 创建：alloc_socket()分配资源
- 绑定：inet_bind()绑定端口
- 连接：tcp_v4_connect()建立TCP连接
- 数据传输：通过sk_buff结构体传递数据包

3. 建立连接流程：
```
Socket建立连接过程
.
├── 服务端
│   ├── 1. 创建ServerSocket
│   ├── 2. 绑定端口(bind)
│   ├── 3. 监听连接(listen)
│   └── 4. 接受连接(accept)
└── 客户端
    ├── 1. 创建Socket
    └── 2. 连接服务端(connect)
```

2. 数据传输：
- 基于TCP协议保证可靠传输
- 使用输入/输出流进行数据读写
- 需要处理粘包/拆包问题

3. Android实现示例：
```java
// 服务端
ServerSocket server = new ServerSocket(8080);
Socket client = server.accept();
BufferedReader in = new BufferedReader(
    new InputStreamReader(client.getInputStream()));
PrintWriter out = new PrintWriter(client.getOutputStream());

// 客户端
Socket socket = new Socket("127.0.0.1", 8080);
BufferedWriter out = new BufferedWriter(
    new OutputStreamWriter(socket.getOutputStream()));
BufferedReader in = new BufferedReader(
    new InputStreamReader(socket.getInputStream()));
```

4. 注意事项：
- 网络操作需在子线程执行
- 需要处理连接超时和异常
- 注意资源释放

## 2. 共享内存

### 2.1 基本概念
```
共享内存原理  
.
├── 进程A           # 映射共享内存区域
├── 进程B           # 映射同一内存区域
└── 内核            # 管理共享内存
```

### 2.2 内核实现
1. 数据结构：
```
共享内存内核结构
.
├── shmid_kernel    # 共享内存标识符结构
│   ├── shm_perm    # 权限信息
│   ├── shm_segsz   # 内存段大小
│   └── shm_pages   # 页表指针
└── vm_area_struct  # 进程虚拟内存映射
```

2. 关键流程：
- 创建：shmget()系统调用创建共享内存段
- 映射：shmat()将共享内存映射到进程地址空间
- 同步：使用信号量或互斥锁保护共享数据
- 释放：shmdt()解除映射，shmctl()删除

3. Android实现：
- Ashmem(Anonymous Shared Memory)
- 通过Binder传递文件描述符
- 图形系统广泛使用(SurfaceFlinger)

### 2.3 核心特点
- 零拷贝，性能最高
- 需要同步机制(如信号量)
- 安全性较低
- 实现复杂度高

### 2.3 Android中的应用
- 图形系统(SurfaceFlinger)
- 大数据量传输
- 高性能计算场景

## 3. 管道/消息队列

### 3.1 基本概念
```
管道通信模型
.
├── 写端进程        # 向管道写入数据
└── 读端进程        # 从管道读取数据
```

### 3.2 内核实现
1. 数据结构：
```
管道内核结构
.
├── pipe_inode_info # 管道核心结构
│   ├── buf         # 环形缓冲区
│   ├── head        # 写入位置
│   └── tail        # 读取位置
└── file            # 文件描述符
```

2. 关键流程：
- 创建：pipe()系统调用创建管道
- 写入：copy_from_user()将数据写入缓冲区
- 读取：copy_to_user()从缓冲区读取数据
- 同步：等待队列实现阻塞读写

3. Android实现：
- 通过pipe2()系统调用创建
- 常用于进程间stdout/stderr重定向
- Logcat日志系统底层使用管道

### 3.3 核心特点
- 半双工通信
- 简单易用
- 效率较低
- 容量有限

### 3.3 Android中的应用
- Shell命令管道
- 简单进程间通信
- 日志系统

## 4. 与Binder的对比

```
Android IPC机制对比
.
├── Binder          # 高效安全的IPC机制(默认使用)
│   ├── 优点：高性能、安全、易用
│   └── 缺点：传输大小受限
├── Socket          # 网络通信/复杂场景
│   ├── 优点：灵活、跨网络
│   └── 缺点：开销大
├── 共享内存         # 高性能但复杂
│   ├── 优点：零拷贝
│   └── 缺点：同步复杂
└── 管道/消息队列     # 简单但效率低
    ├── 优点：简单
    └── 缺点：效率低
```

## 5. 常见问题

### Q: 为什么Android主要使用Binder？
A: 主要因为：
1. 性能优于传统IPC
2. 安全性更好
3. 面向对象的调用方式更符合Android架构

### Q: 什么场景下会使用共享内存？
A: 典型场景包括：
1. 图形缓冲区共享
2. 大数据量传输
3. 对性能要求极高的场景

### Q: Socket在Android中的应用？
A: 主要用于：
1. 网络通信
2. 跨设备通信
3. 复杂协议场景
