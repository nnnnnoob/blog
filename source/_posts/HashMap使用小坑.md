---
title: HashMap使用小坑
date: 2017-11-26 16:25:32
categories: java
tags: [java,hashmap]
comments: true

---
>HashMap是java中最常用的集合工具类之一，同时HashMap在使用过程中也有很多小坑，稍不注意可能会引起重大问题，下面稍作总结<!-- more -->

### 隐患的泛型

#### case回放
```java
public class MapGeneric {

	static class Apple {
	}

	static class IPhone {
	}

	public static void main(String[] args) {

		Map<String, List> map = new HashMap<String, List>();

		List<Apple> appleList = new ArrayList<Apple>();

		appleList.add(new Apple());

		map.put("appleList", appleList);

		map.get("appleList").add(new IPhone());

		for (Apple apple : appleList) {

		}
	}

}
```
执行这段代码，会抛出java.lang.ClassCastException异常。

#### case分析
代码逻辑比较简单，重点在第n行map.get()方法返回的List未指定具体类型，因此后面add()方法可以添加一个IPhone，程序可以正常执行。但是此时的appleList里面包含了Apple和IPhone两种对象，再次使用appleList便出现了问题。

#### case总结
使用泛型时，请务必明确指定各类型；复杂数据结构请使用Object，别再List和Map嵌套了好吗。

### 飙升的CPU

#### case回放
某次上线后，开始收到CPU过高的报警，通过top和stack分析查看，发现有两个线程一直在执行HashMap.put()方法。用以下代码复现
```java
public class MapResize {

	public static void main(String[] args) throws Exception {

		final Map<Integer, String> map = new HashMap<Integer, String>(16);
		final String value = "";

		Thread thread1 = new Thread(new Runnable() {
			@Override
			public void run() {
				for (int i = 1; i < 100000; i++) {
					map.put(i, value);
				}
			}
		});

		Thread thread2 = new Thread(new Runnable() {
			@Override
			public void run() {
				for (int i = 100000; i < 200000; i++) {
					map.put(i, value);
				}
			}
		});

		thread1.start();
		thread2.start();

		Thread.sleep(1000 * 5);
	}
}
```
stack信息如下
```shell
"Thread-1" prio=5 tid=0x00007fb66308c000 nid=0x4c03 runnable [0x0000000119989000]
   java.lang.Thread.State: RUNNABLE
        at java.util.HashMap.put(HashMap.java:494)
        at MapResize$2.run(MapResize.java:24)
        at java.lang.Thread.run(Thread.java:745)

"Thread-0" prio=5 tid=0x00007fb66308b000 nid=0x4a03 runnable [0x0000000119886000]
   java.lang.Thread.State: RUNNABLE
        at java.util.HashMap.put(HashMap.java:494)
        at MapResize$1.run(MapResize.java:15)
        at java.lang.Thread.run(Thread.java:745)

```

#### case分析
罪魁祸首是resize()方法导致，HashMap(1.7)中resize源码如下
```java
void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
```
其中引用的transfer()方法源码如下
```java
void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
}
```
在for循环中，转移节点时出现了并发问题，从而导致了死循环的出现。具体细节网上已有多篇文章详述。

#### case总结
使用HashMap时，尽量设置合理的初始size，减少resize次数；并发多的话使用ConcurrentHashMap吧。

### 灾难的OutOfMemoryError

#### case回放
如果没有合理的监控，resize导致的CPU飙升问题很难发现。但是如果不及时解决，很可能会导致OutOfMemoryError的发生。复现代码如下
```shell
public class MapResize {

	public static void main(String[] args) throws Exception {

		final Map<Integer, String> map = new HashMap<Integer, String>(16);
		final String value = "";

		Thread thread1 = new Thread(new Runnable() {
			@Override
			public void run() {
				for (int i = 1; i < 100000; i++) {
					map.put(i, value);
				}
			}
		});

		Thread thread2 = new Thread(new Runnable() {
			@Override
			public void run() {
				for (int i = 100000; i < 200000; i++) {
					map.put(i, value);
				}
			}
		});

		thread1.start();
		thread2.start();

		Thread.sleep(1000 * 5);

		System.out.println(map);
	}
}
```
代码只比上面的多了一行print。

#### case分析
上面说到resize会导致死循环，那map的容量应该是有限的呀，怎么会OutOfMemoryError呢？看一下HashMap的源码，并没有toString()方法，默认继承了AbstractMap的toString()方法，AbstractMap的toString()源码如下
```java
public String toString() {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        if (! i.hasNext())
            return "{}";

        StringBuilder sb = new StringBuilder();
        sb.append('{');
        for (;;) {
            Entry<K,V> e = i.next();
            K key = e.getKey();
            V value = e.getValue();
            sb.append(key   == this ? "(this Map)" : key);
            sb.append('=');
            sb.append(value == this ? "(this Map)" : value);
            if (! i.hasNext())
                return sb.append('}').toString();
            sb.append(',').append(' ');
        }
}
```
看到for循环和i.next应该能确定了，根本原因还是上述死循环导致，在这里不停地拼接字符串，最后导致了OutOfMemoryError。

#### case总结
使用HashMap时，设置合理的初始size！！2的n次方！！