---
layout: post
title: "遍历HashSet和HashMap"
date: 2015-04-12
category: Java
tag: Java
---

### 遍历HashSet

```java
Set<String> set = new HashSet<String>();
set.add("123");
set.add("abc");

// 方法一：使用迭代器
for(Iterator<String> i = set.iterator(); i.hasNext(); ) {
    System.out.println(i.next());
}

// 方法二：使用 JDK 5 新增的 foreach 循环，只有数组和实现了 Iterable 接口的类才能这样写。
// 冒号左边是一个局部变量，表示这个 set 中被迭代出来的元素，右边是一个数组或者是实现了 Iterable 
// 接口的类。
for(String str : set) {
    System.out.println(str);
}
```

### 遍历HashMap

```java
Map<String, String> map = new HashMap<String, String>();
map.put("1", "a");
map.put("2", "b");

// 方法一：通过 map 的 keySet 集，获得 map 中所有 key 的 set 对象
for(Iterator<String> i = map.keySet().iterator(); i.hasNext(); ) {
    String key = i.next();
    String value = map.get(key);
    System.out.println(key + " --> " + value);
}

// 方法二：通过 map 的 entrySet，获得 map 中所有键值对的 set 对象
for(Iterator<Map.Entry<String, String>> i = map.entrySet().iterator(); i.hasNext(); ) {
    Map.Entry<String, String> entry = i.next();
    System.out.println(entry.getKey() + " --> " + entry.getValue());
}

// 方法三：使用 foreach 循环，并通过 map 的 entrySet 遍历
for(Map.Entry<String, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " --> " + entry.getValue());
}
```