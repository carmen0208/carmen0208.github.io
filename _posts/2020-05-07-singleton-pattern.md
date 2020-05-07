---
layout: post
title:  "设计模式1 单例模式  Singleton Design Pattern"
date:   2020-05-07 21:26:03 +1200
tags: designpattern programming
---
---
#### 什么是单例模式
它是一种创建型模式。一个类只允许创建一个对象。

#### What? Singleton
*Singleton* is a creational design pattern that lets you ensure that a class have only one instance while providing a global access point to this instance.

---

#### 为什么我们需要单例模式。

1. 更好的对共享资源进行管理

    在应用程序中，当我们需要访问各种的外部资源，例如文件，数据库等等的时候，会容易出现资源占用的问题甚至冲突。
   	* 对于很昂贵的外部资源访问，比如数据库连接，ssh连接，我们不想每次都新建一个连接，而是沿用之前建立好的
   	* 如果我们有多处地方正在同时对这个资源进行修改，那么这个共享资源会产生内容冲突。

    举例：我们程序中提供了写日志的功能，将程序运行的日志写到一个叫xxx.log的文件中去。代码如下：
  
    ```java
	  public class Logger {
		  private FileWriter writer
      public Logger() {
        File file = new File("/you/path/xxx.log");
        writer = new FileWriter(file, true) // true代表追加写入
      }

    public void log(String message) {
      writer.write(message)
    }
    } 
    ```

    	程序运行的时候，我们的TicketService和Payment Service 在同时被调用并且他们同时都要写日志。假设他们同时拿到这个文件，因为这个文件目前有的是在这两个service写入之前的内容。OrderService先写完保存，Paymentservice之后写完同时保存。当PaymentService写完后保存时，它并不知道OrderService已经修改了这个文件，就直接将文件覆盖了。

      为了解决这个冲突单例模式是解决这个问题的方案之一。 [注1](http://carmen0208.github.io/2020/singleton-pattern/#%E6%B3%A8%E9%87%8A1)

2. 全局唯一标识，比如自增长ID. 这种全局标识如果暴露在外非常不安全，因为每一个地方都有可能对他进行修改，从而导致系统崩溃，如果我们能把它放到一个对象中，就能保整这个全局变量被任意修改。

#### Why we need it ?
1. Better manage our  external Resources

    It is normal that we need to access to external resources like file, database from our application and there is where  we want to have more control:
   	
     1. For some expensive external resource, we want to have one connection, like database connection and ssh connection. We don’t want to open and close all the time.
	Instead of receiving a fresh object, we would like to get the one that we already created. This behaviour is impossible to implement with a regular constructor as a constructor call must always return new object.
   	
     2. Also, If we have a share file that need to be updated often from multiply places(like logger function). It more likely to have a conflict.

    For example, We have logging function in our application which writing logs to xxx.log file.  Here’s the code:
  
    ```java
	  public class Logger {
		  private FileWriter writer
      public Logger() {
        File file = new File("/you/path/xxx.log");
        writer = new FileWriter(file, true) // true代表追加写入
      }

    public void log(String message) {
      writer.write(message)
    }
    } 
    ```

    Assume both TicketingService and OrderService are using the logger to write the log to file. The get the file at the same time and write concurrently. TicketingSerivce finish write later that OrderService. When TicketingService finished write the log. The log will replace the one that OrderService just saved.

    Singleton is one of the solution for tackle this problem.[注1](http://carmen0208.github.io/2020/singleton-pattern/#%E6%B3%A8%E9%87%8A1)

2. Global variables is very unsafe since any code can potentially overwrite the contents of those variables and crash the app. Just like a global variable, the Singleton pattern lets you access some object from anywhere in the program. However, it also protect that instance from being overwritten by other code.

---

#### 单例模式应该怎么写
* 最重要的两点要素
  1. 保证默认的构造器私有，这样阻止了其他程序用new 的方式来创建新的对象的可能
  2. 提供一个静态方法去获取这个类的对象。内部实现更应该是调用私有构造器创建他并存储在某处

##### 他的实现方式通常有两种：

* 懒汉式： -> Lazy Singleton

  ```java
  package com.carmen.design.patterns.singleton;

  import java.util.concurrent.atomic.AtomicLong;

  public class IdGeneratorLazy {
      private AtomicLong id = new AtomicLong(0);

      private static IdGeneratorLazy idGenerator;
      private IdGeneratorLazy(){}
      // because it is not thread save, ot can be create multiply times, so we have to a synchronized the method
      public static synchronized IdGeneratorLazy getInstance() {
          if (idGenerator == null) {
              return new IdGeneratorLazy();
          } else {
              return idGenerator;
          }
      }

      public long getId() {
          return id.incrementAndGet();
      }
  }
  ```
* 饿汉单例模式 -> Eager Singleton*(cannot wait to make the things)

  ```java
  package com.carmen.design.patterns.singleton;

  import java.util.concurrent.atomic.AtomicLong;

  public class IdGenerator {
      private AtomicLong id = new AtomicLong(0);
      //Eager Singleton
      private static final IdGenerator instance = new IdGenerator();

      private IdGenerator(){}

      public static IdGenerator getInstance() {
          return instance;
      }
      // this method can be get after instance been get
      public long getId() {
          return id.incrementAndGet();
      }
  }
  ```
##### 单例模式还应该考虑的点：
* 线程安全
* 是否支持延迟加载 ，例如懒汉模式
* getInstance()性能

  ```java
  package com.carmen.design.patterns.singleton;

  import java.util.concurrent.atomic.AtomicLong;

  public class IdGeneratorOptmestic {

      private AtomicLong id = new AtomicLong(0);

      private static IdGeneratorOptmestic idGenerator;

      private IdGeneratorOptmestic() {
      }

      // this is better synchronized than synchronized in method level. as when if not null, it can be return right away.
      public static IdGeneratorOptmestic getInstance() {
          if (idGenerator == null) {
              synchronized (IdGeneratorOptmestic.class) {
                  if (idGenerator == null) {
                      return new IdGeneratorOptmestic();
                  }
              }

          }
          return idGenerator;
      }

      public long getId() {
          return id.incrementAndGet();
      }
  }

  ```
---

#### How to write Singleton
* The 2 most important point:
  1. Make the default constructor private, to prevent other objects from using the new operator with the singleton class.
  2. Create a static creation method that acts as a constructor. Under the hood, this method calls the private constructor to create an object and saves it in a static field. All following calls to this method return the cached object.

##### It contains 2 implementation:
* Lazy Singleton -> Initial Object when call static getInstance method

  ```java
  package com.carmen.design.patterns.singleton;

  import java.util.concurrent.atomic.AtomicLong;

  public class IdGeneratorLazy {
      private AtomicLong id = new AtomicLong(0);

      private static IdGeneratorLazy idGenerator;
      private IdGeneratorLazy(){}
      // because it is not thread save, ot can be create multiply times, so we have to a synchronized the method
      public static synchronized IdGeneratorLazy getInstance() {
          if (idGenerator == null) {
              return new IdGeneratorLazy();
          } else {
              return idGenerator;
          }
      }

      public long getId() {
          return id.incrementAndGet();
      }
  }
  ```

* Eager Singleton -> Initial Object when load Class

  ```java
  package com.carmen.design.patterns.singleton;

  import java.util.concurrent.atomic.AtomicLong;

  public class IdGenerator {
      private AtomicLong id = new AtomicLong(0);
      //Eager Singleton
      private static final IdGenerator instance = new IdGenerator();

      private IdGenerator(){}

      public static IdGenerator getInstance() {
          return instance;
      }
      // this method can be get after instance been get
      public long getId() {
          return id.incrementAndGet();
      }
  }
  ```

##### Other things that need to take into consideration:
* Support Lazy Loading? Ex: Lazy Singleton
* Thread Safe, with Lazy Singleton, to avoid double initialise the instance, we have to synchronise the initialisation part.
* performance of getInstance, as we explained for lazy Singleton, if we do synchronise at the method level, it may block all the things, including the already initialled one.  So we need to consider to synchronise it after we check the instance is not null.

  ```java
  package com.carmen.design.patterns.singleton;

  import java.util.concurrent.atomic.AtomicLong;

  public class IdGeneratorOptmestic {

      private AtomicLong id = new AtomicLong(0);

      private static IdGeneratorOptmestic idGenerator;

      private IdGeneratorOptmestic() {
      }

      // this is better synchronized than synchronized in method level. as when if not null, it can be return right away.
      public static IdGeneratorOptmestic getInstance() {
          if (idGenerator == null) {
              synchronized (IdGeneratorOptmestic.class) {
                  if (idGenerator == null) {
                      return new IdGeneratorOptmestic();
                  }
              }

          }
          return idGenerator;
      }

      public long getId() {
          return id.incrementAndGet();
      }
  }

  ```


##### 实用范围
* When you want to use the same connection to a database to make every query
* When you open SSH connection to a server to do a few tasks
* Use the Singleton pattern when a class in your program should have just a single instance available to all clients; for example, a single database object shared by different parts of the program.
* Use the singleton pattern when you need stricter control over global variables.
Unlike global variables, the Singleton pattern guarantees that there’s just one instance of a class. Nothing, except for the Singleton class itself, can replace the cached instance.

#### 单例模式的缺点

* **单例会隐藏类之间的依赖关系**
* **单例对代码的可测试性不友好**


* Violates the *Single Responsibility Principle*. The pattern solves two problems at the time.
* Hide the dependency relationships
* Not Test friendly.


#### 替代方案：
实 际上，类对象的全局唯一性可以通过多种不同的方式来保证。我们既可以通过单例模式来强 制保证，也可以通过工厂模式、IOC 容器(比如 Spring IOC 容器)来保证，还可以通过程 序员自己来保证(自己在编写代码的时候自己保证不要创建两个类对象)。

DI instead of Singleton:

This seems like the correct answer to me.
Singleton works through global access, and in pure FP, you can not access any state which is not passed to you as an argument, so every attempt to access global state is breaking the pure FP paradigm.
So Singleton is really just, at the boundary, initiate a DB connection once, and pass it into your pure core logic. Which is a form of dependency injection.


---
###### 注释1
```
* 另外还有的方案有：
	* 加类级别的锁。
		* synchronized(Logger.class)
	* 分布式锁
	* 并发队列：
		* 多个线程同时往并发队列里写日志，一个单独的线程负责将并发队列中的数据，写入到日志 文件
```
