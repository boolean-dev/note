# 1. 前言

本文主要介绍我在面试中所遇到的问题，并且以我的角度，来描述我说理解的知识，尽量使用通俗话的方法

# 2. Java基础知识

### 2.1 反射

#### 反射是什么？

反射 (Reflection) 是 Java 程序开发语言的特征之一，它允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。通过 Class 获取 class 信息称之为反射（Reflection）。

通俗来讲，反射允许我们在**运行时**，获取一个类的属性和方法，并且能够修改属性值、调用方法。

#### 主要用处

反射在我们开发中，其实随处可见

- DEA和Eclipse的语法提示
- Spring框架中，使用的大量的xml配置文件，通过配置文件加载Bean
- MyBatis框架中，查询结果就是通过通过实体类反射返回

#### 获取Class对象

1. 调用运行时类本身的 `.class` 属性

```java
Class clazz1 = Person.class;
System.out.println(clazz1.getName());
```

2. 通过运行时类的对象获取 `getClass();`

```java
Person p = new Person();
Class clazz3 = p.getClass();
System.out.println(clazz3.getName());
```

3. `Class.forName()`，Mysql加载驱动

```java
public static Class<?> forName(String className)
// 在JDBC开发中常用此方法加载数据库驱动:
Class.forName(driver);
```

4. `ClassLoader()`通过类的加载器

```java
ClassLoader classLoader = this.getClass().getClassLoader();
Class clazz5 = classLoader.loadClass(className);
System.out.println(clazz5.getName());
```

### 2.2 注解

#### 什么是注解

注解是Java5引入的新特性，简单来说：注解其实就是**代码中的特殊标记**，这些标记可以**在编译、类加载、运行时被读取，并执行相对应的处理**。

#### 元注解

1. @Retention ，注解的生命周期
2. @Target，注解的作用位置
3. @Documented ，是否出现在Java文档中
4. @Inherited ，如果有该标记，则该注解作用于类的子类

#### 使用示例

1. `Spring`中常见的注解`Controller`，等注解，用于标记是`Controller`
2. Swagger文档中常用的注解

### 2.3 深拷贝和浅拷贝

#### 深拷贝

对基本类型数据进行值传递，对对象引用创建新对象

#### 浅拷贝

被复制对象的所有变量都含有与原来的对象相同的值，而所有的对其他对象的引用仍然指向原来的对象。换言之，浅拷贝仅仅复制所拷贝的对象，而不复制它所引用的对象。

# 3. 面向对象

## 3.1 Java的四个基本特性

- **抽象**：将一类事物所共有的特征和行为总结出来，构造一个类的过程。例如`Person`，有`头`,`脚`等特征，有`吃饭`、`睡觉`的动作
- **封装**：封装就是隐藏一切可隐藏的东西，只向外界提供最简单的编程接口
- **继承**：子类继承父类的某些特性
- **多态**：不同子类型的对象对同一消息作出不同的响应，例如鸡叫和狗叫完全不同

## 3.2 volatile

### 并发三大概念

- **原子性**：全部执行OR全部失败
- **可见性**：一个线程修改后，其余线程立即可见
- **有序性**：代码按照顺序执行，不会被指令重排

### volatile

Volatile 是一个**类型修饰符**（type specifier），它是被设计用来修饰被不同线程访问和修改的变量

保证了并发的两大特性

1. 保证了不同线程对这个变量进行操作时的可见性
2. 禁止进行指令重排序

# 4. 集合

## 4.1 集合架构图

![1535785576589.png](../image/1535785576589-6077b7ee.png)

## 4.2 Collection

- `ArrayList`：**线程不同步**。默认初始容量为 10，当数组大小不足时容量扩大为 1.5 倍。为追求效率，ArrayList 没有实现同步（synchronized），如果需要多个线程并发访问，用户可以手动同步，也可使用 Vector 替代。

- `LinkedList`：**线程不同步**。**双向链接实现**。LinkedList 同时实现了 List 接口和 Deque 接口，也就是说它既可以看作一个顺序容器，又可以看作一个队列（Queue），同时又可以看作一个栈（Stack）。这样看来，LinkedList 简直就是个全能冠军。当你需要使用栈或者队列时，可以考虑使用 LinkedList，一方面是因为 Java 官方已经声明不建议使用 Stack 类，更遗憾的是，Java 里根本没有一个叫做 Queue 的类（它是个接口名字）。关于栈或队列，现在的首选是 ArrayDeque，它有着比 LinkedList（当作栈或队列使用时）有着更好的性能。

- `Stack and Queue`：Java 里有一个叫做 Stack 的类，却没有叫做 Queue 的类（它是个接口名字）。当需要使用栈时，Java 已不推荐使用 Stack，而是推荐使用更高效的 ArrayDeque；既然 Queue 只是一个接口，当需要使用队列时也就首选 ArrayDeque 了（次选是 LinkedList ）。

- `Vector`：**线程同步**。默认初始容量为 10，当数组大小不足时容量扩大为 2 倍。它的同步是通过 `Iterator` 方法加 `synchronized` 实现的。

- `Stack`：**线程同步**。继承自 Vector，添加了几个方法来完成栈的功能。现在已经不推荐使用 Stack，在栈和队列中有限使用 ArrayDeque，其次是 LinkedList。

- `TreeSet`：**线程不同步**，内部使用 `NavigableMap` 操作。默认元素 “自然顺序” 排列，可以通过 `Comparator` 改变排序。TreeSet 里面有一个 TreeMap（适配器模式）

- `HashSet`：**线程不同步**，内部使用 HashMap 进行数据存储，提供的方法基本都是调用 HashMap 的方法，所以两者本质是一样的。集合元素可以为 NULL。

- `Set`：Set 是一种不包含重复元素的 Collection，Set 最多只有一个 null 元素。Set 集合通常可以通过 Map 集合通过适配器模式得到。

- ```
  PriorityQueue
  ```

  ：Java 中 PriorityQueue 实现了 Queue 接口，不允许放入 null 元素；其通过堆实现，具体说是通过完全二叉树（complete binary tree）实现的

  小顶堆

  （任意一个非叶子节点的权值，都不大于其左右子节点的权值），也就意味着可以通过数组来作为 PriorityQueue 的底层实现。

  - **优先队列的作用是能保证每次取出的元素都是队列中权值最小的**（Java 的优先队列每次取最小元素，C++ 的优先队列每次取最大元素）。这里牵涉到了大小关系，**元素大小的评判可以通过元素本身的自然顺序（natural ordering），也可以通过构造时传入的比较器**（*Comparator*，类似于 C++ 的仿函数）。

- `NavigableSet`：添加了搜索功能，可以对给定元素进行搜索：小于、小于等于、大于、大于等于，放回一个符合条件的最接近给定元素的 key。

- `EnumSet`：线程不同步。内部使用 Enum 数组实现，速度比 `HashSet` 快。**只能存储在构造函数传入的枚举类的枚举值**。

集合类中，共有两个顶级接口，一个是Collection，一个是Map

- Collection下面有三个接口，分别是List,Queue,Set
  - List分别有三个实现形式，ArrayList，Vector，LinkList
    - ArrayList是数组结构，线程不同步，初始化容量是10，扩容量为1.5
    - LinkedList是双向链表结构，线程不同步，同时，它也实现了Deque接口，也可实现队列以及栈的结构
    - Vactor也是基于数组实现，是线程安全的，初始化容量是10，扩容量是2。它的同步是通过 `Iterator` 方法加 `synchronized` 实现的
  - Queue是队列，数据具有先进先出
    - PriorityQueue是线程不安全，无界的，实现原理是通过数组，具有先进先出的特征
    - ArrayQueue是线程不安全，它是基于数组实现，拥有两个指针，分别指向队列和队尾
  - Set是一个元素不重复的集合，主要实现方式有两种，LinkHashSet,HashSet,TreeSet
    - LinkHashSet,LinkedHashSet集合同样是根据元素的hashCode值来决定元素的存储位置，但是它同时使用链表维护元素的次序。这样使得元素看起 来像是以插入顺序保存的，也就是说，当遍历该集合时候，LinkedHashSet将会以元素的添加顺序访问集合的元素
    - HashSet,**线程不同步**，内部使用 HashMap 进行数据存储，提供的方法基本都是调用 HashMap 的方法，所以两者本质是一样的。集合元素可以为 NULL.HashSet会调用该对象的hashCode()方法来得到该对象的hashCode值，然后根据 hashCode值来决定该对象在HashSet中存储位置
    - TreeSet:TreeSet是SortedSet接口的唯一实现类，TreeSet可以确保集合元素处于排序状态

