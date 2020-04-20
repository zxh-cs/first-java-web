#                               day11.多线程

## 一.创建线程的第一种方式(extends Thread)

```java
1.创建一个类
2.继承Thread类
3.重写Thread中的run方法-->run方法的作用就是该线程要做的事儿
4.创建主线程,创建自己的线程类对象
5.调用start方法
   调用run:仅仅代表我调用了此对象中的run方法,但是不代表,此线程开启了
   调用start:1.开启线程 2.jvm调用run方法
6.注意:
   一个线程对象,不能连续调用多次start()方法
```

```java
public class MyMainTest02 {
    public static void main(String[] args) {
        //创建线程类对象
        MyThread myThread = new MyThread();
        //启动线程
        myThread.start();

        for (int i = 0; i < 10; i++) {
            System.out.println("MyMain..."+i);
        }
    }
}

public class MyThread extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println("MyThread..."+i);
        }
    }
}
```

## 二.Thread类中的方法

```java
1.String getName()-->获取线程名字
2.static Thread currentThread() -->获取当前正在运行的线程对象
                                   在哪个线程中调用,获取的就是哪个线程对象
3. void setName(String name)  -->为线程设置名字
4.static void sleep(long time) -->让线程睡眠,传递毫秒值
```

```java
public class MyThread extends Thread{
    public MyThread() {
    }

    public MyThread(String name) {
        super(name);
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
             try {
                Thread.sleep(1000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"..."+i);
        }
    }
}

public class MyMainTest02 {
    public static void main(String[] args) {
        //创建线程类对象
        MyThread myThread = new MyThread("涛哥");
        //myThread.setName("钢蛋");
        //启动线程
        myThread.start();
        for (int i = 0; i < 10; i++) {
             try {
                Thread.sleep(1000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"..."+i);
        }
    }
}
```

## 三.创建线程的第二种方式(实现Runnable接口)

```java
 创建多线程程序两种方式的区别
    1.使用实现Runnable接口的方式实现多线程程序,可以避免单继承的局限性
        a.类继承了Thread类,就不能继承其他的类
        b.类实现了Runnable接口,还可以继承其他的类
    2.使用实现Runnable接口的方式实现多线程程序,把设置线程任务和开启线程进行了解耦(解除了耦合性增强了扩展性)
        a.类继承了Thread类,在run方法中设置的什么线程任务,创建对象就执行什么线程任务
        b.类实现了Runnable接口的目的:重写run方法设置线程任务;创建Thread类对象的目的:传递线程任务,执行线程任务
```

```java
public class MyThread implements Runnable{
    @Override
    public void run() {
        //设置线程任务
        for (int i = 0; i < 10; i++) {
            System.out.println(Thread.currentThread().getName()+"..."+i);
        }
    }
}

public class MyMain01 {
    public static void main(String[] args) {
        //创建MyThread对象
        MyThread myThread = new MyThread();

        //创建Thread对象
        Thread thread = new Thread(myThread);
        thread.start();

        for (int i = 0; i < 10; i++) {
            System.out.println(Thread.currentThread().getName()+"..."+i);
        }
    }
}
```

## 四.使用匿名内部类方式创建多线程

```java

public class MyMain02 {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    System.out.println(Thread.currentThread().getName()+"..."+i);
                }
            }
        }).start();

        for (int i = 0; i < 10; i++) {
            System.out.println(Thread.currentThread().getName()+"..."+i);
        }
    }
}
```

## 五.高并发以及线程安全的介绍

```properties
高并发：是指在某个时间点上，有大量的用户(线程)同时访问同一资源。例如：天猫的双11购物节、12306的在线购票在某个时间点上，都会面临大量用户同时抢购同一件商品/车票的情况。
    
线程安全：在某个时间点上，当大量用户(线程)访问同一资源时，由于多线程运行机制的原因，可能会导致被访问的资源出现"数据污染"的问题。
```

## 六.高并发问题一_可见性

```java
先启动一个线程，在线程中将一个变量的值更改，而主线程却一直无法获得此变量的新值。
```

```java
//创建一个类继承Thread类
public class MyThread extends Thread {
    //定义一个共享的静态成员变量,供多个线程一起使用
    public static int a = 0;

    //重写Thread类中的run方法,设置线程任务
    @Override
    public void run() {
        System.out.println("Thread-0线程开始执行了,先睡眠2秒钟");
        try {
            Thread.sleep(2000);//睡眠的期间,好让主线程去执行
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Thread-0线程睡醒了,改变变量a的值为1");
        a=1;
        System.out.println("Thread-0线程的线程任务执行完毕");
    }
}
```

```java
public class Demo01Visible {
    public static void main(String[] args) {
        //创建Thread类的子类对象
        MyThread mt = new MyThread();
        //调用start方法,开启新的线程执行run方法
        mt.start();

        System.out.println("main线程,开启完新的线程,会继续执行main方法中的代码!");
        while (true){
            /*由于死循环速度太快了,所以一直读到的是自己内存空间中的0,所以就一直执行不下去,可以使用sleep
             降低主线程中死循环的循环效率,这样就可以访问到主内存中被修改的变量a了
            */
            /*try {
                Thread.sleep(1000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }*/
            if(MyThread.a==1){
                System.out.println("main线程,判断变量a的值为1,结束死循环");
                break;
            }
        }
    }
}
```

###      6.1.Java内存模型JMM

概述：JMM(Java Memory Model)Java内存模型,是java虚拟机规范中所定义的一种内存模型。

Java内存模型(Java Memory Model)描述了Java程序中各种变量(线程共享变量)的访问规则，以及在JVM中将变量存储到内存和从内存中读取变量这样的底层细节。

所有的共享变量都存储于主内存。这里所说的变量指的是实例变量(成员变量)和类变量(静态成员变量)。不包含局部变量，因为局部变量是线程私有的(属于方法本身)，因此不存在竞争问题。每一个线程还存在自己的工作内存，线程的工作内存，保留了被线程使用的变量的工作副本。[线程对变量的所有的操作(读，取)都必须在工作内存中完成，而不能直接读写主内存中的变量，不同线程之间也不能直接访问]

对工作内存中的变量，线程间变量的值的传递需要通过主内存完成。

![](image/1561516472597.png)



### 6.2可见性的原理:

![1581316624977](image/1581316624977.png)

## 七.高并发线程安全问题二-有序性

​	1).有序性：多行代码的编写顺序和编译顺序。
有些时候，编译器在编译代码时，为了提高效率，会对代码“重排”：

```java
.java文件
int a = 10;		//第一行
int b = 20;		//第二行
int c = a * b;	//第三行
```

在执行第三行之前，由于第一行和第二行的先后顺序无所谓，所以编译器可能会对“第一行”和“第二行”进行代码重排：

```java
.class
int b = 20;
int a = 10;
int c = a * b;
```

2).但在多线程环境下，这种重排可能是我们不希望发生的，因为：重排，可能会影响另一个线程的结果,所以我们不需要代码进行重排

![1581317330167](image/1581317330167.png)

## 八.高并发问题三_原子性(不可分割)

```java
package demo04atom;

public class MyThread extends Thread{
    public static int money = 0;

    @Override
    public void run() {
        System.out.println("Thread-0线程开始改变money的值!");
        for (int i = 0; i < 10000; i++) {
            money++;
        }
        System.out.println("Thread-0线程执行完毕!");
    }
}
```

```java
package demo04atom;

public class Demo01Thread {
    public static void main(String[] args) throws InterruptedException {
        //开启新的线程
        MyThread mt  = new MyThread();
        mt.start();

        System.out.println("main继续往下执行");
        System.out.println("main线程开始改变money的值!");
        for (int i = 0; i < 10000; i++) {
            MyThread.money++;
        }
        System.out.println("main线程暂停1秒,让Thread-0执行完毕!");
        Thread.sleep(1000);
        System.out.println("最终money为:"+MyThread.money);
    }
}

```

```java
main线程会继续执行下边的代码!
main线程开始改变变量money的值!
main线程暂停1秒钟,等待Thread-0线程执行完毕,在继续执行!
Thread-0线程开始改变money的值!
Thread-0线程线程任务执行完毕!
最终money的值为:20000
    
main线程会继续执行下边的代码!
main线程开始改变变量money的值!
Thread-0线程开始改变money的值!
Thread-0线程线程任务执行完毕!
main线程暂停1秒钟,等待Thread-0线程执行完毕,在继续执行!
最终money的值为:12484
    
main线程会继续执行下边的代码!
main线程开始改变变量money的值!
Thread-0线程开始改变money的值!
main线程暂停1秒钟,等待Thread-0线程执行完毕,在继续执行!
Thread-0线程线程任务执行完毕!
最终money的值为:17965    
```

![1581318378866](image/1581318378866.png)

每个线程访问money变量，都需要三步：
	1).取money的值；
	2).将money++
	3).将money写回
这三步就不具有“原子性”——执行某一步时，很可能会被暂停，执行另外一个线程，就会导致变量的最终结果错误！！！！

## 九 volatile关键字

### 1. volatile解决可见性

```java
package com.itheima.demo03volatileVisible;

//创建一个类继承Thread类
public class MyThread extends Thread{
    //定义一个共享的静态成员变量,供多个线程一起使用
    //给共享的成员变量,添加一个volatile关键字修饰
    public static volatile int a = 0;

    //重写Thread类中的run方法,设置线程任务
    @Override
    public void run() {
        System.out.println("Thread-0线程开始执行了,先睡眠2秒钟");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Thread-0线程睡醒了,改变a的值为1");
        /*
        a变量被volatile修饰,当改变了变量a的值,volatile关键字会让所有a的变量副本失效,线程想要使用a变的         值,需要在静态区中重新获取
        */
        a=1;
        System.out.println("Thread-0线程的线程任务执行完毕!");
    }
}
```

```java
package com.itheima.demo03volatileVisible;

public class Demo01Visible {
    public static void main(String[] args) {
        //创建Thread类的子类对象
        MyThread mt = new MyThread();
        //调用继续自Thread类中的start方法,开启新的线程执行run方法
        mt.start();

        System.out.println("main线程,会继续执行main方法中的代码");
        while (true){
            if(MyThread.a==1){
                System.out.println("main线程,判断变量a的值为1,结束死循环!");
                break;
            }
        }
    }
}


```

![1581319944662](image/1581319944662.png)

### 2.volatile解决有序性

**注意:变量添加了volatile关键字,就不会在进行重排了**

![1576375878625](image/1576375878625.png)

### 3. volatile不能解决原子性

```java
package demo06volatile;

public class MyThread extends Thread{
    public static volatile int money = 0;

    @Override
    public void run() {
        System.out.println("Thread-0线程开始改变money的值!");
        for (int i = 0; i < 10000; i++) {
            money++;
        }
        System.out.println("Thread-0线程执行完毕!");
    }
}
```

```java
package demo06volatile;

public class Demo01Thread {
    public static void main(String[] args) throws InterruptedException {
        //开启新的线程
        MyThread mt  = new MyThread();
        mt.start();

        System.out.println("main继续往下执行");
        System.out.println("main线程开始改变money的值!");
        for (int i = 0; i < 10000; i++) {
            MyThread.money++;
        }
        System.out.println("main线程暂停1秒,让Thread-0执行完毕!");
        Thread.sleep(1000);
        System.out.println("最终money为:"+ MyThread.money);
        //最终money为:15668 仍然不准确,volatile解决不了原子性
    }
}
```

## 十.原子性

概述：所谓的原子性是指在一次操作或者多次操作中，要么所有的操作全部都得到了执行并且不会受到任何因素的干扰而中断，要么所有的操作都不执行，

多个操作是一个不可以分割的整体(原子)。

比如：从张三的账户给李四的账户转1000元，这个动作将包含两个基本的操作：从张三的账户扣除1000元，给李四的账户增加1000元。这两个操作必须符合原子性的要求，

要么都成功要么都失败。

## 1.原子类概述

概述：java从JDK1.5开始提供了java.util.concurrent.atomic包(简称Atomic包)，这个包中的原子操作类提供了一种用法简单，性能高效，线程安全地更新一个变量的方式。

1). java.util.concurrent.atomic.AtomicInteger：对int变量进行原子操作的类。

2). java.util.concurrent.atomic.AtomicLong：对long变量进行原子操作的类。

3). java.util.concurrent.atomic.AtomicBoolean：对boolean变量进行原子操作的类。

这些类可以保证对“某种类型的变量”原子操作，多线程、高并发的环境下，就可以保证对变量访问的有序性，从而保证最终的结果是正确的。

AtomicInteger原子型Integer，可以实现原子更新操作

```java
构造方法:
public AtomicInteger()：	   				初始化一个默认值为0的原子型Integer
public AtomicInteger(int initialValue)： 初始化一个指定值的原子型Integer
成员方法:
int get():   			 				 获取值
int getAndIncrement():      			 以原子方式将当前值加1，注意，这里返回的是自增前的值。
int incrementAndGet():     				 以原子方式将当前值加1，注意，这里返回的是自增后的值。
int addAndGet(int data):				 以原子方式将输入的数值与实例中的值（AtomicInteger里的value）相加，并返回结果。
int getAndSet(int value):   			 以原子方式设置为newValue的值，并返回旧值。
```

## 2.AtomicInteger类的基本使用

```java
package com.itheima.demo01AtomicInteger;

import java.util.concurrent.atomic.AtomicInteger;

/*
     java.util.concurrent.atomic.AtomicInteger：对int变量进行原子操作的类
        构造方法:
            public AtomicInteger()：	   				初始化一个默认值为0的原子型Integer
            public AtomicInteger(int initialValue)： 初始化一个指定值的原子型Integer
        成员方法:
            int get():   			 				 获取值
            int getAndIncrement():      			 以原子方式将当前值加1，注意，这里返回的是自增前的值。
            int incrementAndGet():     				 以原子方式将当前值加1，注意，这里返回的是自增后的值。
            int addAndGet(int data):				 以原子方式将输入的数值与实例中的值（AtomicInteger里的value）相加，并返回结果。
            int getAndSet(int value):   			 以原子方式设置为newValue的值，并返回旧值。
 */
public class Demo01AtomicInteger {
    public static void main(String[] args) {
        //创建AtomicInteger对象,初始值为10
        AtomicInteger ai = new AtomicInteger(10);

        //int get():  获取值
        int i = ai.get();
        System.out.println(i);//10

        //int getAndIncrement():  以原子方式将当前值加1，注意，这里返回的是自增前的值。
        int i2 = ai.getAndIncrement();  //i++;
        System.out.println("i2:"+i2);//i2:10
        System.out.println(ai);//11

        //int incrementAndGet(): 以原子方式将当前值加1，注意，这里返回的是自增后的值。
        int i3 = ai.incrementAndGet();// ++i
        System.out.println("i3:"+i3);//i3:12
        System.out.println(ai);//12

        //int addAndGet(int data):以原子方式将输入的数值与实例中的值（AtomicInteger里的value）相加，并返回结果。
        int i4 = ai.addAndGet(100);// 12+100
        System.out.println("i4:"+i4);//i4:112
        System.out.println(ai);//112

        //int getAndSet(int value): 以原子方式设置为newValue的值，并返回旧值。
        int i5 = ai.getAndSet(88);
        System.out.println("i5:"+i5);//i5:112 被替换的值
        System.out.println(ai);//88
    }
}
```



## 3.AtomicInteger解决变量的可见性、有序性、原子性(重点)

```java
package demo07AtomicInteger;

import java.util.concurrent.atomic.AtomicInteger;

public class MyThread extends Thread{
    //不要直接使用int类型的变量,使用AtomicInteger
    public static AtomicInteger money = new AtomicInteger(0);

    @Override
    public void run() {
        System.out.println("Thread-0线程开始改变money的值!");
        for (int i = 0; i < 10000; i++) {
            //money.getAndIncrement();//先获取后自增  money++
            money.incrementAndGet();//先自增后获取  ++money
        }
        System.out.println("Thread-0线程执行完毕!");
    }
}

```

```java
package demo07AtomicInteger;

public class Demo01Thread {
    public static void main(String[] args) throws InterruptedException {
        //开启新的线程
        MyThread mt  = new MyThread();
        mt.start();

        System.out.println("main继续往下执行");
        System.out.println("main线程开始改变money的值!");
        for (int i = 0; i < 10000; i++) {
            MyThread.money.incrementAndGet();
        }
        System.out.println("main线程暂停1秒,让Thread-0执行完毕!");
        Thread.sleep(1000);
        System.out.println("最终money为:"+ MyThread.money);
    }
}
```

## 3.AtomicInteger的原理_CAS机制（乐观锁）

![1581386343690](image/1581386343690.png)

## 3.AtomicIntegerArray(了解)

可以用原子方式更新其元素的 `int` 数组:可以保证数组的原子性

构造方法:

```java
AtomicIntegerArray(int length) 创建指定长度的给定长度的新 AtomicIntegerArray。
AtomicIntegerArray(int[] array) 创建与给定数组具有相同长度的新 AtomicIntegerArray，并从给定数组复制其所有元素。
```

成员方法:

```java
int addAndGet(int i, int delta) 以原子方式将给定值与索引 i 的元素相加。 
i:获取指定索引处的元素
delta:给元素增加的值

int get(int i) 获取指定索引处元素的值

```

### 没有使用AtomicIntegerArray数组存在原子性的问题

```java
package com.itheima.demo06AtomicIntegerArray;

public class MyThread extends Thread{
    //定义一个静态所有线程共享的数组
    public static int[] arr = new int[1000];

    @Override
    public void run() {
        //遍历数组,改变数组中变量的值0-->1
        for (int i = 0; i < arr.length; i++) {
            arr[i]++;//arr[i]=arr[i]+1;
        }
    }
}
```

```java
package com.itheima.demo06AtomicIntegerArray;

import java.util.Arrays;

public class Demo01AtomicIntegerArray {
    public static void main(String[] args) {
        //创建1000个线程,每个线程都把数组中的元素的值加1==>0-->1000 最终的结果[1000,1000,1000,...]
        for (int i = 0; i < 1000; i++) {
            new MyThread().start();
        }

        //让主线程睡眠5秒钟,等待1000个线程执行完毕
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("main线程睡眠结束,遍历数组,获取数组中的元素");
        System.out.println(Arrays.toString(MyThread.arr));
    }
}
```

执行结果:

```java
[1000, 1000, 999, 999, 999, 999, 999, 999, 999, 999, 999, 999, 999, 999, 999, 999, 999, 1000, 1000, 1000, 1000, 1000, 1000, 1000,]
```

### 使用AtomicIntegerArray数组解决数组的原子性问题

```java
package com.itheima.demo07AtomicIntegerArray;

import java.util.concurrent.atomic.AtomicIntegerArray;

public class MyThread extends Thread{
    //定义一个静态所有线程共享的数组,不要使用基本数据类型的数组了,使用AtomicIntegerArray数组
    public static int[] arr = new int[1000];
    public static AtomicIntegerArray integerArray = new AtomicIntegerArray(arr);

    @Override
    public void run() {
        //遍历数组,改变数组中变量的值0-->1
        for (int i = 0; i < 1000; i++) {
            //把指定索引处的元素,加1
            integerArray.addAndGet(i,1);
        }
    }
}
```

```java
package com.itheima.demo07AtomicIntegerArray;

import java.util.Arrays;

public class Demo01AtomicIntegerArray {
    public static void main(String[] args) {
        //创建1000个线程,每个线程都把数组中的元素的值加1==>0-->1000 最终的结果[1000,1000,1000,...]
        for (int i = 0; i < 1000; i++) {
            new MyThread().start();
        }

        //让主线程睡眠5秒钟,等待1000个线程执行完毕
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("main线程睡眠结束,遍历数组,获取数组中的元素");
        for (int i = 0; i < MyThread.integerArray.length(); i++) {
            //获取指定索引处的元素
            System.out.println(MyThread.integerArray.get(i));
        }
    }
}

```

# 

