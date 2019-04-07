🖕欢迎关注我的公众号“彤哥读源码”，查看更多源码系列文章, 与彤哥一起畅游源码的海洋。 

---

## 简介

这篇文章主要讲述jdk本身的源码该如何阅读，关于各种框架的源码阅读我们后面再一起探讨。

笔者认为阅读源码主要包括下面几个步骤。

## 设定目标

凡事皆有目的，阅读源码也是一样。

从大的方面来说，我们阅读源码的目的是为了提升自己的技术能力，运用到工作中，遇到问题快速定位，升职加薪等等。

从小的方面来说，阅读某一段源码的目的就是要搞清楚它的原理，就是死磕，就是那种探索真相的固执。

目的是抽象的，目标是具体的，我们阅读源码之前一定要给自己设定一个目标。

比如，下一章我们将要一起学习的ConcurrentHashMap，我们可以设定以下目标：

（1）熟悉ConcurrentHashMap的存储结构；

（2）熟悉ConcurrentHashMap中主要方法的实现过程；

（3）探索ConcurrentHashMap中出现的新技术；

## 提出问题

有了目标之后，我们要试着提出一些问题。

还是以ConcurrentHashMap为例，笔者提出了以下这些问题：

（1）ConcurrentHashMap与HashMap的数据结构是否一样？

（2）HashMap在多线程环境下何时会出现并发安全问题？

（3）ConcurrentHashMap是怎么解决并发安全问题的？

（4）ConcurrentHashMap使用了哪些锁？

（5）ConcurrentHashMap的扩容是怎么进行的？

（6）ConcurrentHashMap是否是强一致性的？

（7）ConcurrentHashMap不能解决哪些问题？

（8）ConcurrentHashMap除了并发安全，还有哪些与HashMap不同的地方，为什么要那么实现？

（8）ConcurrentHashMap中有哪些不常见的技术值得学习？

## 如何提出问题

很多人会说，我也知道要提出问题，但是该怎么提出问题呢？

这确实是很困难的一件事，笔者认为主要是三点：

（1）问自己

把自己当成面试官问自己，往死里问的那种。

如果问自己问不出几个问题，也不要紧，请看下面。

（2）问互联网

很多问题可能自己也想不到，那就需要上网大概查一下相关的博客，看人家有没有提出什么问题。

或者，查询相关面试题。

比如，笔者学习ConcurrentHashMap这个类时，上网一查很多都是基于jdk7的，那这时候就可以提出一个问题，jdk8与jdk7中ConcurrentHashMap这个类的实现方式有何不同？jdk8对jdk7作了哪些优化？

（3）不断发现问题

在源码阅读的过程中，可能看着看着就遇到个问题，这是非常常见的，这种问题也应该保留下来研究研究。

比如，ConcurrentHashMap中size()方法是怎么实现的？`@sun.misc.Contended`这玩意是什么鬼东西？然后上网一查，与是为了避免伪共享，我X，`伪共享`又是啥？然后你再查一下`伪共享`，又出来了CPU多级缓存？学完CPU多级缓存，是不是觉得跟jvm的内存模型很像？问完这一连串问题，是不是感觉世界都清晰了？^_^

看吧，问题是源源不断地被发现的。

所以，一开始提不出几个问题也不要紧，关键是要看，看了才能发现更多的问题。

## 带着问题阅读源码，忽略不必要的细节，死磕重要的细节

首先，一定要带着问题阅读源码。

其次，一定要忽略不必要的细节。

再次，一定要死磕重要的细节。

乍一看，后面两步似乎有所矛盾，其实不然，忽略不必要的细节是为了不迷失在源码的世界中，死磕重要的细节是为了弄清楚源码的真相。

这里的细节是忽略还是死磕，主要是看跟问题的相关性。

jdk源码还是比较好阅读的，如果后面看spring的源码，做不到忽略不必要的细节，真的是会迷失的，先埋个伏笔哈~~

举个例子，之前阅读过ArrayList的序列化相关的代码中的readObject()方法。

`s.readInt();`这行是干嘛的？省略行不行？这时候就要去了解序列化相关的知识，然后看看writeObject()里面的实现，这就是要死磕的代码。

`SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);`这行又是干嘛的？乍一看，好像是跟权限相关的代码，跟我们的问题“序列化”无关，忽略之，如果实在想知道，先打个标记，等把序列化的问题解决了再来研究这个东西。

```java
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
    // 声明为空数组
    elementData = EMPTY_ELEMENTDATA;

    // 读入非transient非static属性（会读取size属性）
    s.defaultReadObject();

    // 读入元素个数，没什么用，只是因为写出的时候写了size属性，读的时候也要按顺序来读
    s.readInt();

    if (size > 0) {
        // 计算容量
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        // 检查是否需要扩容
        ensureCapacityInternal(size);
        
        Object[] a = elementData;
        // 依次读取元素到数组中
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

## 多做比较

在阅读jdk源码的时候，还有很重要的一点，就是要多做比较，比较也可以分为横向比较和纵向比较。

（1）横向比较

就是与相似的类做比较。比如，集合模块中，基本都是各种插入、查询、删除元素，那这时候可以从数据结构、时间复杂度等维度进行比较，这就是横向比较。

（2）纵向比较

可以从集合发展的历史进行比较。比如，HashMap的发展史，从（单个数组）实现（没错，可以直接用一个数组实现HashMap），到（多数组+链表）实现，再到jdk8中的（多数组+链表+红黑树）实现，这就是纵向比较。

## 多做实验

最后一步，最最最最重要的就是要多做实验。

比如，ConcurrentHashMap是不是强一致性的？

可以启动多个线程去不断调用get()、put()、size()方法，看看是不是强一致性的。

### 彩蛋

哎呀，一不小心透露了下一章ConcurrentHashMap的内容。

大家可以用本篇所说的方法试着阅读一下ConcurrentHashMap的源码，下一章我们再一起学习哈哈~~

---

如果你觉得本篇还算有用

赏个鸡腿呗😉

