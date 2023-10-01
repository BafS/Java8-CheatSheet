
# JAVA 8 - 备忘录

## Lambda 表达式

```java
(int a) -> a * 2; // 求a乘以2后的值
a -> a * 2; // 或者更直接的去掉类型也是可以的
```

```java
(a, b) -> a + b; // 相加
```

如果lambda里面的代码块超过1行，可以配合使用 `{ }` 加 `return`来处理

```java
(x, y) -> {
    int sum = x + y;
    int avg = sum / 2;
    return avg;
}
```

一个lamdba表达式必须依赖一个具体的功能接口而存在

```java
interface MyMath {
    int getDoubleOf(int a);
}

MyMath d = a -> a * 2; // 关联到具体的接口实现
d.getDoubleOf(4); // is 8
```

---

下面所有的测试都是用到这个`list` :

```java
List<String> list = [Bohr, Darwin, Galilei, Tesla, Einstein, Newton]
```


## Collections 集合

**sort** `sort(list, comparator)`

```java
list.sort((a, b) -> a.length() - b.length())
list.sort(Comparator.comparing(n -> n.length())); // 使用具体Comparator接口实现
list.sort(Comparator.comparing(String::length)); // 这样写和上面也是一样的
//> [Bohr, Tesla, Darwin, Newton, Galilei, Einstein]
```

**removeIf**

```java
list.removeIf(w -> w.length() < 6);
//> [Darwin, Galilei, Einstein, Newton]
```

**merge**
`merge(key, value, remappingFunction)`

```java
Map<String, String> names = new HashMap<>();
names.put("Albert", "Ein?");
names.put("Marie", "Curie");
names.put("Max", "Plank");

//  "Albert" 这个值是存在的 就命中了后面处理流程
// {Marie=Curie, Max=Plank, Albert=Einstein}
names.merge("Albert", "stein", (old, val) -> old.substring(0, 3) + val);

// "Newname" 这个值是不存在的 所以后面的流程就不处理
// {Marie=Curie, Newname=stein, Max=Plank, Albert=Einstein}
names.merge("Newname", "stein", (old, val) -> old.substring(0, 3) + val);
```


## 方法引用 `Class::staticMethod` 

允许引用类方法或者构造函数，引用时候是不执行的

```java
//通过lamdba
getPrimes(numbers, a -> StaticMethod.isPrime(a));

//通过应用方法:
getPrimes(numbers, StaticMethod::isPrime);
```

| Method Reference | Lambda Form |
| ---------------- | ----------- |
| `StaticMethod::isPrime` | `n -> StaticMethod.isPrime(n)` |
| `String::toUpperCase`   | `(String w) -> w.toUpperCase()` |
| `String::compareTo`     | `(String s, String t) -> s.compareTo(t)` |
| `System.out::println`   | `x -> System.out.println(x)` |
| `Double::new`           | `n -> new Double(n)` |
| `String[]::new`         | `(int n) -> new String[n]` |


## Streams 流式处理

和`collections`类似, 但有所不同

 - 不能储存数据
 - 数据来源外部例如 (collection, file, db, web, ...)
 - `immutable`不可变性，不影响外部数据 (因为产生了一个新的stream)
 - `lazy`懒式处理 (只有在计算的时候才用到，不处理不用 !)
 
```java
// 仅仅计算前3个"filter"
Stream<String> longNames = list
   .filter(n -> n.length() > 8)
   .limit(3);
```

**创建一个stream**

```java
Stream<Integer> stream = Stream.of(1, 2, 3, 5, 7, 11);
Stream<String> stream = Stream.of("Jazz", "Blues", "Rock");
Stream<String> stream = Stream.of(myArray); // 通过数组
Stream<String> stream = list.stream(); // 通过list

// Infinit stream [0; inf[
Stream<Integer> integers = Stream.iterate(0, n -> n + 1);
```

**集合结果集**

```java
//返回成一个数组 (::new 是构造函数的引用)
String[] myArray = stream.toArray(String[]::new);

// 返回成list或set
List<String> myList = stream.collect(Collectors.toList());
Set<String> mySet = stream.collect(Collectors.toSet());

// 返回成String
String str = list.collect(Collectors.joining(", "));

// 返回成一个LinkedHashMap
list.stream().collect(Collectors.toMap(k -> k, v -> v, (a, b) -> a, LinkedHashMap::new));
// 默认转换成HashMap
list.stream().collect(Collectors.toMap(k -> k, v -> v));
```

**map** `map(mapper)`<br>
对每个元素进行类型转换

```java
// 对每个元素使用 "toLowerCase" 处理
Stream<String> res = stream.map(w -> w.toLowerCase());
Stream<String> res = stream.map(String::toLowerCase);
//> bohr darwin galilei tesla einstein newton

Stream<Integer> res = Stream.of(1, 2, 3, 4, 5).map(x -> x + 1);
//> 2 3 4 5 6
```

**filter** `filter(predicate)`<br>
过滤处理，只保留匹配到的元素

```java
// 过掉保留 "E" 开头的元素
res = stream.filter(n -> n.substring(0, 1).equals("E"));
//> Einstein

res = Stream.of(1, 2, 3, 4, 5).filter(x -> x < 3);
//> 1 2
```

**reduce**<br>
汇聚处理成为单一返回结果

```java
String reduced = stream.reduce("", (acc, el) -> acc + "|" + el);
//> |Bohr|Darwin|Galilei|Tesla|Einstein|Newton
```

**limit** `limit(maxSize)`<br>
保留前`maxSize`个元素

```java
res = stream.limit(3);
//> Bohr Darwin Galilei
```

**skip**<br>
忽略掉前`n`个元素

```java
res = stream.skip(2); // 忽略 Bohr 和 Darwin
//> Galilei Tesla Einstein Newton
```

**distinct**<br>
去除重复的元素

```java
res = Stream.of(1, 0, 0, 1, 0, 1).distinct();
//> 1 0
```

**sorted**<br>
排序 (必须使用 *Comparable* 接口)

```java
res = stream.sorted();
//> Bohr Darwin Einstein Galilei Newton Tesla 
```

**allMatch** / **noneMatch**

```java
// 检查是否每个元素都是“e”开头
boolean res = words.allMatch(n -> n.contains("e"));
```

`anyMatch`: 只要其中一个元素包含"e"即可 <br>
`noneMatch`: 元素里面是否没有"e" 

**parallel**<br>
返回一个并行的stream

**findAny**<br>
在并行流上findFirst执行更快

### 原始类型的 Streams

原子类型的stream自动封装是低效的 (例如 Stream<Integer>) ，因为它需要对每个元素进行大量拆箱和装箱. 所以最好使用 `IntStream`, `DoubleStream`, 等等.

**初始化**

```java
IntStream stream = IntStream.of(1, 2, 3, 5, 7);
stream = IntStream.of(myArray); // 通过数组
stream = IntStream.range(5, 80); //  5 到 80范围

Random gen = new Random();
IntStream rand = gen(1, 9); // stream of randoms
```

使用 *mapToX* (mapToObj, mapToDouble, mapToLong) 如果需要把字段转换成 Object, double, long的话. values.

### Grouping 数据集

**Collectors.groupingBy**

```java
// 通过长度分组
Map<Integer, List<String>> groups = stream.collect(Collectors.groupingBy(w -> w.length()));
//> 4=[Bohr], 5=[Tesla], 6=[Darwin, Newton], ...
```

**Collectors.toSet**

```java
// 和之前一样但是使用的是Set
Map<String, Set<String>> groups2 = stream.collect(Collectors.groupingBy(w -> w.substring(0, 1), Collectors.toSet()));
//> {B=[Bohr], T=[Tesla], D=[Darwin], E=[Einstein], G=[Galilei], N=[Newton]}
```

**Collectors.counting**<br>
获取元素总算

**Collectors.summing__**<br>
`summingInt`, `summingLong`, `summingDouble` 计算所有元素值相加后结果

**Collectors.averaging__**<br>
`averagingInt`, `averagingLong`, ... 计算所有元素值的算术平均值

```java
// 计算平均数
Collectors.averagingInt(String::length)
```

*PS*: 另外不要忘记 Optional (例如 `Map<T, Optional<T>>`) 有同样的处理方法 (例如 `Collectors.maxBy`).


### 并行 Streams 

**创建一个并行处理的stream**

```java
Stream<String> parStream = list.parallelStream();
Stream<String> parStream = Stream.of(myArray).parallel();
```

**unordered** 
能提高计算 `limit`，`distinct`的速度

```java
stream.parallelStream().unordered().distinct();
```

*PS*: 使用streams类库, 例如使用 `filter(x -> x.length() < 9)` 代替 `forEach` 和 `if`


## Optional

在Java, 通常使用`null`表示没有结果，但是如果不检查的话很容易出现`NullPointerException`.

```java
// Optional<String> 包含一个string和空
Optional<String> res = stream
   .filter(w -> w.length() > 10)
   .findFirst();

// 返回元素长度或者返回 "" 如果没有的话
int length = res.orElse("").length();

// 使用lambda作为一个返回值
res.ifPresent(v -> results.add(v));
```

返回一个 Optional

```java
Optional<Double> squareRoot(double x) {
   if (x >= 0) { return Optional.of(Math.sqrt(x)); }
   else { return Optional.empty(); }
}
```

---

**注意引用推测限制**

```java
interface Pair<A, B> {
    A first();
    B second();
}
```

一个 steam 类型 `Stream<Pair<String, Long>>` :
- `stream.sorted(Comparator.comparing(Pair::first)) // 有效`
- `stream.sorted(Comparator.comparing(Pair::first).thenComparing(Pair::second)) // 无效`

Java不能通过 `.comparing(Pair::first)`回调过来的数据来判断类型, 故 `Pair::first` 就不能这样用了

如果需要使用泛型接口的话需要显示写清楚，否则无效

```java
stream.sorted(
    Comparator.<Pair<String, Long>, String>comparing(Pair::first)
    .thenComparing(Pair::second)
) // 有效
```

---

This cheat sheet was based on the lecture of Cay Horstmann
http://horstmann.com/heig-vd/spring2015/poo/
