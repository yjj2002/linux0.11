# 信号量的实现与应用

1. [实验内容](#实验内容)
2. [实验过程](#实验过程)
	- [实验结果](#实验结果)
	- [实验分析](#实验分析)

## 实验内容
1. 在Ubuntu下编写程序，用信号量解决生产者-消费者问题    
  在Ubuntu下编写应用程序`pc.c`，解决经典的生产者-消费者问题，完成下面的功能：    
    - 建立一个生产者进程，N个消费者进程(N > 1)
    - 用文件建立一个共享缓冲区
    - 生产者进程依次向缓冲区写入整数0,1,2,...,M, M >= 500
    - 消费者进程从缓冲区读数，每次读一个，并将读出的数字从缓冲区删除，然后将本进程ID+数字输出到标准输出
    - 缓冲区同时最多只能保存10个数    
  `pc.c`中将会用到`sem_open()`,`sem_unlink()`,`sem_wait()`,`sem_post()`等信号量相关的系统调用，请查阅相关文档。    
2. 在0.11中实现信号量，用生产者-消费者程序检验之    
  Linux在0.11版还没有实现信号量，Linus把这件富有挑战的工作留给了你。如果能够实现一套山寨版的完全符合POSIX规范的信号量，无疑是很有成就感的。但时间暂时不允许我们这么做，所以先弄一套缩水版的类POSIX信号量，它的原型和标准并不完全相同，而且只包含如下系统调用：    
    ```c
    sem_t *sem_open(const char *name, unsigned int value);
    int sem_wait(sem_t *sem);
    int sem_post(sem_t *sem);
    int sem_unlink(const char *name);
    ```

    - `sem_t`是信号量类型，根据实现的需要自定义
    - `sem_open`    
      功能是创建一个信号量，或打开一个已经存在的信号量。
	  `name`是信号量的名字。不同的进程可以通过提供同样的`name`而共享同一个信号量。如果该信号量不存在，就创建新的名为`name`的信号量；如果存在，就打开已经存在的名为`name`的信号量。    
	  `value`是信号量的初值，仅当新建信号量时，此参数才有效，其余情况下它被忽略。    
	  当成功时，返回值是该信号量的唯一标识(比如，在内核的地址，ID等)，由另外两个系统调用使用。如失败，返回值是NULL。    
    - `sem_wait`    
      信号量的P原子操作。如果继续运行的条件不满足，则令调用进程等待在信号量`sem`上。    
	  返回0表示成功，返回-1表示失败。    
    - `sem_post`    
      信号量的V原子操作。如果有等待`sem`的进程，它会唤醒其中的一个。返回0表示成功，返回-1表示失败。    
    - `sem_unlink`    
      功能是删除名为`name`的信号量。返回0表示成功，返回-1表示失败。    
  在*kernel*目录下新建`sem.c`文件实现如下功能。然后将`pc.c`从Ubuntu移植到0.11下，测试自己实现的信号量。    
3. 实验报告    
  1. 在`pc.c`中去掉所有与信号量有关的代码，再运行程序，执行效果有变化吗？为什么会这样？    
  2. 实验的设计者在第一次编写生产者-消费者程序的时候，是这么做的：
    ```c
    Producer()
	{
		P(Mutex);  //互斥信号量
		生产一个产品item;
		P(Empty);  //空闲缓存资源
		将item放到空闲缓存中;
		V(Full);  //产品资源
		V(Mutex);
	}

	Consumer()
	{
		P(Mutex);  
		P(Full);  
		从缓存区取出一个赋值给item;
		V(Empty);
		消费产品item;
		V(Mutex);
	}
	```
    这样可行吗？如果可行，那么它和标准解法在执行效果上会有什么不同？如果不可行，那么它有什么问题使它不可行？    

## 实验过程    

### 实验结果    
1. 在Ubuntu上运行pc.c的结果：    
  ![Ubuntu上pc.c的执行结果](https://github.com/Wangzhike/HIT-Linux-0.11/raw/master/5-semaphore/picture/result-on-ubuntu.png)

2. 在添加了实现信号量的0.11上运行移植好的pc.c的结果：    
  ![编译执行移植好的pc.c](https://github.com/Wangzhike/HIT-Linux-0.11/raw/master/5-semaphore/picture/experiment-result.png)
  ![执行结果](https://github.com/Wangzhike/HIT-Linux-0.11/raw/master/5-semaphore/picture/experiment-result-2.png)

