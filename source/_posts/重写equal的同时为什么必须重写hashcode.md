---
title: 重写equal的同时为什么必须重写hashcode？
date: 2016-07-05 11:39:30
tags: Android
---

hashCode是编译器为不同对象产生的不同整数，根据equal方法的定义：如果两个对象是相等（equal）的，那么两个对象调用 hashCode必须产生相同的整数结果，即：equal为true，hashCode必须为true，equal为false，hashCode也必须 为false，所以必须重写hashCode来保证与equal同步。

```java
class Student {
 int num;
 String name;

 Student(int num, String name) {
  this.num = num;
  this.name = name;
 }

 public int hashCode() {
  return num * name.hashCode();
 }

 public boolean equals(Object o) {
  Student s = (Student) o;
  return num == s.num && name.equals(s.name);
 }

 public String toString() {
  return num + ":" + name;
 }
}
```

在Java的集合中，判断两个对象是否相等的规则是：

1. 判断两个对象的hashCode是否相等
如果不相等，认为两个对象也不相等，完毕
如果相等，转入2

2. 判断两个对象用equals运算是否相等
如果不相等，认为两个对象也不相等
如果相等，认为两个对象相等

```
1、为什么要重载equal方法？

答案：因为Object的equal方法默认是两个对象的引用的比较，意思就是指向同一内存,地址则相等，否则不相等；如果你现在需要利用对象里面的值来判断是否相等，则重载equal方法。

2、为什么重载hashCode方法？

答案：一般的地方不需要重载hashCode，只有当类需要放在HashTable、HashMap、HashSet等等hash结构的集合时才会 重载hashCode，那么为什么要重载hashCode呢？就HashMap来说，好比HashMap就是一个大内存块，里面有很多小内存块，小内存块 里面是一系列的对象，可以利用hashCode来查找小内存块hashCode%size(小内存块数量)，所以当equal相等时，hashCode必 须相等，而且如果是object对象，必须重载hashCode和equal方法。

3、 为什么equals()相等，hashCode就一定要相等，而hashCode相等，却不要求equals相等?
答案：
- 因为是按照hashCode来访问小内存块，所以hashCode必须相等。
- HashMap获取一个对象是比较key的hashCode相等和equal为true。

之所以hashCode相等，却可以equal不等，就比如ObjectA和ObjectB他们都有属性name，那么hashCode都以name计算，所以hashCode一样，但是两个对象属于不同类型，所以equal为false。

4、 为什么需要hashCode?

通过hashCode可以很快的查到小内存块。
通过hashCode比较比equal方法快，当get时先比较hashCode，如果hashCode不同，直接返回false。
```
以下是一个具体类的实例代码：
```java
public class Person
    {
        private String name;
        private int age;

        @Override
        public int hashCode()
        {
            final int prime = 31;
            int result = 1;
            result = prime * result + age;
            result = prime * result + ((name == null) ? 0 : name.hashCode());
            return result;
        }

        @Override
        public boolean equals(Object obj)
        {
            if (this == obj)
                return true;
            if (obj == null)
                return false;
            if (getClass() != obj.getClass())
                return false;
            Person other = (Person) obj;
            if (age != other.age)
                return false;
            if (name == null)
            {
                if (other.name != null)
                    return false;
            } else if (!name.equals(other.name))
                return false;
            return true;
        }
    }
```
首先equals()和hashcode()这两个方法都是从object类中继承过来的。
equals()方法在object类中定义如下：
```java
public boolean equals(Object obj) {
return (this == obj); //==永远是比较两个对象的地址值
}
```
很明显是对两个对象的地址值进行的比较（即比较引用是否相同）。但是我们必需清楚，当String 、Math、还有Integer、Double。。。。等这些封装类在使用equals()方法时，已经覆盖了object类的equals（）方法。比 如在String类中如下：
```java
public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = count;
            if (n == anotherString.count) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = offset;
                int j = anotherString.offset;
                while (n-- != 0) {
                    if (v1[i++] != v2[j++])
                        return false;
                }
                return true;
            }
        }
        return false;
    }
```
很明显，这是进行的内容比较，而已经不再是地址的比较。依次类推Double、Integer、Math。。。。等等这些类都是重写了equals()方法的，从而进行的是内容的比较。当然了基本类型是进行值的比较，这个没有什么好说的。

我们还应该注意，Java语言对equals()的要求如下，这些要求是必须遵循的：

- 对称性：如果x.equals(y)返回是“true”，那么y.equals(x)也应该返回是“true”。
- 反射性：x.equals(x)必须返回是“true”。
- 类推性：如果x.equals(y)返回是“true”，而且y.equals(z)返回是“true”，那么z.equals(x)也应该返回是“true”。
- 还有一致性：如果x.equals(y)返回是“true”，只要x和y内容一直不变，不管你重复x.equals(y)多少次，返回都是“true”。
- 任何情况下，x.equals(null)，永远返回是“false”；x.equals(和x不同类型的对象)永远返回是“false”。

以上这五点是重写equals()方法时，必须遵守的准则，如果违反会出现意想不到的结果，请大家一定要遵守。

其次是hashcode() 方法，在object类中定义如下：
```java
public native int hashCode();
```
说明是一个本地方法，它的实现是根据本地机器相关的。当然我们可以在自己写的类中覆盖hashcode()方法，比如String、Integer、 Double。。。。等等这些类都是覆盖了hashcode()方法的。例如在String类中定义的hashcode()方法如下：
```java
public int hashCode() {
        int h = hash;
        if (h == 0) {
            int off = offset;
            char val[] = value;
            int len = count;

            for (int i = 0; i < len; i++) {
                h = 31*h + val[off++];
            }
            hash = h;
        }
        return h;
    }
```
解释一下这个程序（String的API中写到）：
```java
s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
```

使用 int 算法，这里 s[i] 是字符串的第 i 个字符，n 是字符串的长度，^ 表示求幂。（空字符串的哈希码为 0。）

首先，想要明白hashCode的作用，你必须要先知道Java中的集合。 
　　
总的来说，Java中的集合（Collection）有两类，一类是List，再有一类是Set。
你知道它们的区别吗？前者集合内的元素是有序的，元素可以重复；后者元素无序，但元素不可重复。

那么这里就有一个比较严重的问题了：要想保证元素不重复，可两个元素是否重复应该依据什么来判断呢？

这就是Object.equals方法了。但是，如果每增加一个元素就检查一次，那么当元素很多时，后添加到集合中的元素比较的次数就非常多了。

也就是说，如果集合中现在已经有1000个元素，那么第1001个元素加入集合时，它就要调用1000次equals方法。这显然会大大降低效率。

于是，Java采用了哈希表的原理。哈希（Hash）实际上是个人名，由于他提出一哈希算法的概念，所以就以他的名字命名了。

哈希算法也称为散列算法，是将数据依特定算法直接指定到一个地址上。如果详细讲解哈希算法，那需要更多的文章篇幅，我在这里就不介绍了。

初学者可以这样理解，hashCode方法实际上返回的就是对象存储的物理地址（实际可能并不是）。

这样一来，当集合要添加新的元素时，先调用这个元素的hashCode方法，就一下子能定位到它应该放置的物理位置上。

如果这个位置上没有元素，它就可以直接存储在这个位置上，不用再进行任何比较了；如果这个位置上已经有元素了，
就调用它的equals方法与新元素进行比较，相同的话就不存了，不相同就散列其它的地址。

所以这里存在一个冲突解决的问题。这样一来实际调用equals方法的次数就大大降低了，几乎只需要一两次。
