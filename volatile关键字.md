

## 简介

volatile，由java1.5引入，被认为是一种“轻量级锁”，在多线程编程中时常用到。

## 实验

看下以下代码中对MY_INT有无volatile关键字修饰的影响，以下实验环境为：Mac下的jdk1.7.0_79 ：
```java
/**
 * Created by Qingchang Bai on 2018/9/10.
 */
public class VolatileTest {

    private static int MY_INT = 0;

    public static void main(String[] args) {
        new ChangeListener().start();
        new ChangeMaker().start();
    }

    static class ChangeListener extends Thread {
        @Override
        public void run() {
            int localValue = MY_INT;
            while (localValue < 5) {
                if (localValue != MY_INT) {
                    System.out.println("Got Change for localValue : " + localValue + "  MY_INT: " + MY_INT);
                    localValue = MY_INT;
                }
            }
        }
    }

    static class ChangeMaker extends Thread {
        @Override
        public void run() {
            int localValue = MY_INT;
            while (MY_INT < 5) {
                System.out.println("Incrementing MY_INT to : " + (++localValue));
                MY_INT = localValue;
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

稍微尝试一下就会发现输出结果不一样。

带volatile关键字输出：
```
Incrementing MY_INT to : 1
Got Change for localValue : 0  MY_INT: 1
Incrementing MY_INT to : 2
Got Change for localValue : 1  MY_INT: 2
Incrementing MY_INT to : 3
Got Change for localValue : 2  MY_INT: 3
Incrementing MY_INT to : 4
Got Change for localValue : 3  MY_INT: 4
Incrementing MY_INT to : 5
Got Change for localValue : 4  MY_INT: 5
```
不带volatile关键字输出：
```
Incrementing MY_INT to : 1
Incrementing MY_INT to : 2
Incrementing MY_INT to : 3
Incrementing MY_INT to : 4
Incrementing MY_INT to : 5
```

在Java7下，不带volatile关键字的没有捕捉到一次变化，带volatile关键字捕捉到了每一次变化。

抱着好奇的态度又在Mac下的jdk1.8.0_60尝试了下，输出又有变化，但带volatile和不带volatile关键字的结果仍然不一致。

带volatile关键字输出：

```
Incrementing MY_INT to : 1
Got Change for localValue : 0  MY_INT: 1
Incrementing MY_INT to : 2
Got Change for localValue : 1  MY_INT: 2
Incrementing MY_INT to : 3
Got Change for localValue : 2  MY_INT: 3
Incrementing MY_INT to : 4
Got Change for localValue : 3  MY_INT: 4
Incrementing MY_INT to : 5
Got Change for localValue : 4  MY_INT: 5
```

不带volatile关键字输出：

```
Incrementing MY_INT to : 1
Got Change for localValue : 0  MY_INT: 1
Incrementing MY_INT to : 2
Incrementing MY_INT to : 3
Incrementing MY_INT to : 4
Incrementing MY_INT to : 5
```

在Java8下，不带volatile关键字的能捕捉到第一次变化，但后续的变化没有补助到。

## 原理

之所以有没有volatile关键字对输出结果有这样的差别，主要跟java的内存模型（Java Momery Model）有关系。

TODO