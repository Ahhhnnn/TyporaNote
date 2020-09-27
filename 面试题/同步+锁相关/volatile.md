# volatile

参考 文章：

https://juejin.im/post/6861885337568804871?utm_source=gold_browser_extension



## Volatile是Java虚拟机提供的`轻量级`的同步机制（三大特性）

- 保证可见性
- 不保证原子性
- 禁止指令重排

## JMM

屏蔽各种硬件和操作系统的内存访问差异

**说明用volatile修饰的变量，当某线程更新变量后，其他线程也能感知到**

