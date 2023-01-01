# 第十八章  Java8新特性



## 18.1 Lambda表达式与函数式接口



Lambda写的好可以极大的减少代码冗余，同时可读性也好过冗长的匿名内部类。



### 18.1.1 函数式接口



lambda表达式其实就是实现SAM接口的语法糖，所谓SAM接口就是Single Abstract Method，即该接口中只有一个抽象方法需要实现，当然该接口可以包含其他非抽象方法。



其实只要满足“SAM”特征的接口都可以称为函数式接口，但是如果要更明确一点，最好在声明接口时，加上@FunctionalInterface。



之前学过的接口中，是函数式接口的有：Runnable，Comparator，FileFilter



#### **1、消费型接口**



这类接口的抽象方法特点：有形参，但是返回值类型是void

| 接口名            | 抽象方法                       | 描述                       |
| ----------------- | ------------------------------ | -------------------------- |
| Consumer          | void accept(T t)               | 接收一个对象用于完成功能   |
| BiConsumer<T,U>   | void accept(T t, U u)          | 接收两个对象用于完成功能   |
| DoubleConsumer    | void accept(double value)      | 接收一个double值           |
| IntConsumer       | void accept(int value)         | 接收一个int值              |
| LongConsumer      | void accept(long value)        | 接收一个long值             |
| ObjDoubleConsumer | void accept(T t, double value) | 接收一个对象和一个double值 |
| ObjIntConsumer    | void accept(T t, int value)    | 接收一个对象和一个int值    |
| ObjLongConsumer   | void accept(T t, long value)   | 接收一个对象和一个long值   |



#### **2、供给型接口**



这类接口的抽象方法特点：无参，但是无返回值

| 接口名          | 抽象方法               | 描述              |
| --------------- | ---------------------- | ----------------- |
| Supplier        | T get()                | 返回一个对象      |
| BooleanSupplier | boolean getAsBoolean() | 返回一个boolean值 |
| DoubleSupplier  | double getAsDouble()   | 返回一个double值  |
| IntSupplier     | int getAsInt()         | 返回一个int值     |
| LongSupplier    | long getAsLong()       | 返回一个long值    |



#### **3、判断型接口**



这里接口的抽象方法特点：有参，但是返回值类型是boolean结果。

| 接口名           | 抽象方法                   | 描述             |
| ---------------- | -------------------------- | ---------------- |
| Predicate        | boolean test(T t)          | 接收一个对象     |
| BiPredicate<T,U> | boolean test(T t, U u)     | 接收两个对象     |
| DoublePredicate  | boolean test(double value) | 接收一个double值 |
| IntPredicate     | boolean test(int value)    | 接收一个int值    |
| LongPredicate    | boolean test(long value)   | 接收一个long值   |



#### **4、功能型接口**



这类接口的抽象方法特点：既有参数又有返回值

| 接口名                  | 抽象方法                                        | 描述                                                |
| ----------------------- | ----------------------------------------------- | --------------------------------------------------- |
| Function<T,R>           | R apply(T t)                                    | 接收一个T类型对象，返回一个R类型对象结果            |
| UnaryOperator           | T apply(T t)                                    | 接收一个T类型对象，返回一个T类型对象结果            |
| DoubleFunction          | R apply(double value)                           | 接收一个double值，返回一个R类型对象                 |
| IntFunction             | R apply(int value)                              | 接收一个int值，返回一个R类型对象                    |
| LongFunction            | R apply(long value)                             | 接收一个long值，返回一个R类型对象                   |
| ToDoubleFunction        | double applyAsDouble(T value)                   | 接收一个T类型对象，返回一个double                   |
| ToIntFunction           | int applyAsInt(T value)                         | 接收一个T类型对象，返回一个int                      |
| ToLongFunction          | long applyAsLong(T value)                       | 接收一个T类型对象，返回一个long                     |
| DoubleToIntFunction     | int applyAsInt(double value)                    | 接收一个double值，返回一个int结果                   |
| DoubleToLongFunction    | long applyAsLong(double value)                  | 接收一个double值，返回一个long结果                  |
| IntToDoubleFunction     | double applyAsDouble(int value)                 | 接收一个int值，返回一个double结果                   |
| IntToLongFunction       | long applyAsLong(int value)                     | 接收一个int值，返回一个long结果                     |
| LongToDoubleFunction    | double applyAsDouble(long value)                | 接收一个long值，返回一个double结果                  |
| LongToIntFunction       | int applyAsInt(long value)                      | 接收一个long值，返回一个int结果                     |
| DoubleUnaryOperator     | double applyAsDouble(double operand)            | 接收一个double值，返回一个double                    |
| IntUnaryOperator        | int applyAsInt(int operand)                     | 接收一个int值，返回一个int结果                      |
| LongUnaryOperator       | long applyAsLong(long operand)                  | 接收一个long值，返回一个long结果                    |
|                         |                                                 |                                                     |
| BiFunction<T,U,R>       | R apply(T t, U u)                               | 接收一个T类型和一个U类型对象，返回一个R类型对象结果 |
| BinaryOperator          | T apply(T t, T u)                               | 接收两个T类型对象，返回一个T类型对象结果            |
| ToDoubleBiFunction<T,U> | double applyAsDouble(T t, U u)                  | 接收一个T类型和一个U类型对象，返回一个double        |
| ToIntBiFunction<T,U>    | int applyAsInt(T t, U u)                        | 接收一个T类型和一个U类型对象，返回一个int           |
| ToLongBiFunction<T,U>   | long applyAsLong(T t, U u)                      | 接收一个T类型和一个U类型对象，返回一个long          |
| DoubleBinaryOperator    | double applyAsDouble(double left, double right) | 接收两个double值，返回一个double结果                |
| IntBinaryOperator       | int applyAsInt(int left, int right)             | 接收两个int值，返回一个int结果                      |
| LongBinaryOperator      | long applyAsLong(long left, long right)         | 接收两个long值，返回一个long结果                    |



### 18.1.2 Lambda表达式语法



Lambda表达式是用来给【函数式接口】的变量或形参赋值用的。



其实本质上，Lambda表达式是用于实现【函数式接口】的“抽象方法”



Lambda表达式语法格式



```java
(形参列表) -> {Lambda体}
```



说明：



- (形参列表)它就是你要赋值的函数式接口的抽象方法的(形参列表)，照抄

- {Lambda体}就是实现这个抽象方法的方法体

- ->称为Lambda操作符（减号和大于号中间不能有空格，而且必须是英文状态下半角输入方式）



优化：Lambda表达式可以精简



- 当{Lambda体}中只有一句语句时，可以省略{}和{;}

- 当{Lambda体}中只有一句语句时，并且这个语句还是一个return语句，那么return也可以省略，但是如果{;}没有省略的话，return是不能省略的

- 当(形参列表)的类型是已知的或可以推断的，那么形参的类型可以省略

- 当(形参列表)的类型是已知的或可以推断的，，并且形参个数只有一个，那么可以把数据类型和()一起省略，但是形参名不能省略

- 当(形参列表)是空参时，()不能省略



### 18.1.3 方法引用与构造器引用



Lambda表达式是可以简化函数式接口的变量与形参赋值的语法。而方法引用和构造器引用是为了简化Lambda表达式的。



1、当Lambda表达式满足一些特殊的情况时，还可以再简化：



（1）Lambda体只有一句语句，并且是通过调用一个对象的/类现有的方法来完成的



例如：System.out对象，调用println()方法来完成Lambda体



​      Math类，调用random()静态方法来完成Lambda体



（2）并且Lambda表达式的形参正好是给该方法的实参，或者形参是调用方法的对象，即没有多余的数据出现



例如：t->System.out.println(t)



​     () -> Math.random() 都是无参



2、方法引用的语法格式：



（1）实例对象名::实例方法



（2）类名::静态方法



（3）类名::实例方法：当我们调用这个方法的对象是其中一个形参



说明：



- ::称为方法引用操作符（两个:中间不能有空格，而且必须英文状态下半角输入）

- Lambda表达式的形参列表，全部在Lambda体中使用上了，要么是作为调用方法的对象，要么是作为方法的实参。

- 在整个Lambda体中没有额外的数据。



3、构造器引用



（1）当Lambda表达式是创建一个对象，并且满足Lambda表达式形参，正好是给创建这个对象的构造器的实参列表。



（2）  当Lambda表达式是创建一个数组对象，并且满足Lambda表达式形参，正好是给创建这个数组对象的长度



4、构造器引用的语法格式：



- 类名::new

- 数组元素类型[]::new



## 18.2 StreamAPI



Java8中有两大最为重要的改变。第一个是 Lambda 表达式；另外一个则是 Stream API。



Stream API ( java.util.stream) 把真正的函数式编程风格引入到Java中。这是目前为止对Java类库最好的补充，因为Stream API可以极大提高Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。



Stream 是 Java8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。 使用Stream API 对集合数据进行操作，就类似于使用 SQL 执行的数据库查询。也可以使用 Stream API 来并行执行操作。简言之，Stream API 提供了一种高效且易于使用的处理数据的方式。



Stream是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列。“集合讲的是数据，负责存储数据，Stream流讲的是计算，负责处理数据！”



注意：



①Stream 自己不会存储元素。



②Stream 不会改变源对象。每次处理都会返回一个持有结果的新Stream。即所有的中间操作需要解释新的Stream，除非连写



③Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行。



Stream 的操作三个步骤：



1- 创建 Stream：通过一个数据源（如：集合、数组），获取一个流



2- 中间操作：中间操作是个操作链，对数据源的数据进行n次处理，但是在终结操作前，并不会真正执行。



3- 终止操作：一旦执行终止操作，就执行中间操作链，最终产生结果并结束Stream。






### 18.2.1 创建Stream



**1、创建 Stream方式一：通过集合**



Java8 中的 Collection 接口被扩展，提供了两个获取流的方法：



-  public default Stream stream() : 返回一个顺序流 

-  public default Stream parallelStream() : 返回一个并行流 



**2、创建 Stream方式二：通过数组**



Java8 中的 Arrays 的静态方法 stream() 可以获取数组流：



- public static  Stream stream(T[] array): 返回一个流



重载形式，能够处理对应基本类型的数组：



- public static IntStream stream(int[] array)：返回一个整型数据流

- public static LongStream stream(long[] array)：返回一个长整型数据流

- public static DoubleStream stream(double[] array)：返回一个浮点型数据流



**3、创建 Stream方式三：通过Stream的of()**



可以调用Stream类静态方法 of(), 通过显示值创建一个流。它可以接收任意数量的参数。



- public static Stream of(T... values) : 返回一个顺序流



**4、创建 Stream方式四：创建无限流**



可以使用静态方法 Stream.iterate() 和 Stream.generate(), 创建无限流。



- public static Stream iterate(final T seed, final UnaryOperator f):返回一个无限流

- public static Stream generate(Supplier s) ：返回一个无限流



### 18.2.2  中间操作



多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否则中间操作不会执行任何的处理！而在终止操作时一次性全部处理，称为“惰性求值”。



多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否则中间操作不会执行任何的处理！而在终止操作时一次性全部处理，称为“惰性求值”。

| 方法返回值类型 | **方  法**                          | **描  述**                                                   |
| -------------- | ----------------------------------- | ------------------------------------------------------------ |
| Stream         | **filter(Predicate p)**             | 接收 Lambda ， 从流中排除某些元素                            |
| Stream         | **distinct()**                      | 筛选，通过流所生成元素的equals() 去除重复元素                |
| Stream         | **limit(long maxSize)**             | 截断流，使其元素不超过给定数量                               |
| Stream         | **skip(long n)**                    | 跳过元素，返回一个扔掉了前 n 个元素的流。若流中元素不足 n 个，则返回一个空流。与 limit(n) 互补 |
| Stream         | **peek(Consumer****action)**        | 接收Lambda，对流中的每个数据执行Lambda体操作                 |
| Stream         | **sorted()**                        | 产生一个新流，其中按自然顺序排序                             |
| Stream         | **sorted(Comparator com)**          | 产生一个新流，其中按比较器顺序排序                           |
| Stream         | **map(Function f)**                 | 接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。 |
| Stream         | **mapToDouble(ToDoubleFunction f)** | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 DoubleStream。 |
| Stream         | **mapToInt(ToIntFunction f)**       | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 IntStream。 |
| Stream         | **mapToLong(ToLongFunction f)**     | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 LongStream。 |
| Stream         | **flatMap(Function f)**             | 接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流 |



### 18.2.3  终结操作



终端操作会从流的流水线生成结果。其结果可以是任何不是流的值，例如：List、Integer，甚至是 void。流进行了终止操作后，不能再次使用。

| 方法的返回值类型 | **方法**                             | **描述**                                                     |
| ---------------- | ------------------------------------ | ------------------------------------------------------------ |
| boolean          | **allMatch(Predicate p)**            | 检查是否匹配所有元素                                         |
| boolean          | **anyMatch**(**Predicate p**)        | 检查是否至少匹配一个元素                                     |
| boolean          | **noneMatch(Predicate  p)**          | 检查是否没有匹配所有元素                                     |
| Optional         | **findFirst()**                      | 返回第一个元素                                               |
| Optional         | **findAny()**                        | 返回当前流中的任意元素                                       |
| long             | **count()**                          | 返回流中元素总数                                             |
| Optional         | **max(Comparator c)**                | 返回流中最大值                                               |
| Optional         | **min(Comparator c)**                | 返回流中最小值                                               |
| void             | **forEach(Consumer c)**              | 迭代                                                         |
| T                | **reduce(T iden, BinaryOperator b)** | 可以将流中元素反复结合起来，得到一个值。返回 T               |
| U                | **reduce(BinaryOperator b)**         | 可以将流中元素反复结合起来，得到一个值。返回 Optional        |
| R                | **collect(Collector c)**             | 将流转换为其他形式。接收一个 Collector接口的实现，用于给Stream中元素做汇总的方法 |



Collector 接口中方法的实现决定了如何对流执行收集的操作(如收集到 List、Set、Map)。另外， Collectors 实用类提供了很多静态方法，可以方便地创建常见收集器实例。



## 18.3 Optional类



到目前为止，臭名昭著的空指针异常是导致Java应用程序失败的最常见原因。以前，为了解决空指针异常，Google公司著名的Guava项目引入了Optional类，Guava通过使用检查空值的方式来防止代码污染，它鼓励程序员写更干净的代码。受到Google Guava的启发，Optional类已经成为Java 8类库的一部分。



Optional实际上是个容器：它可以保存类型T的值，或者仅仅保存null。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。



它的API：



1、如何创建Optional对象？或者说如何用Optional来装值对象或null值



（1）static  Optional empty() ：用来创建一个空的Optional



（2）static  Optional of(T value) ：用来创建一个非空的Optional



（3）static  Optional ofNullable(T value) ：用来创建一个可能是空，也可能非空的Optional



2、如何从Optional容器中取出所包装的对象呢？



（1）T get()  ：要求Optional容器必须非空



T get()与of(T value)使用是安全的



（2）T orElse(T other)  ：



orElse(T other) 与ofNullable(T value)配合使用，



如果Optional容器中非空，就返回所包装值，如果为空，就用orElse(T other)other指定的默认值（备胎）代替



（3）T orElseGet(Supplier<? extends T> other)  ：



如果Optional容器中非空，就返回所包装值，如果为空，就用Supplier接口的Lambda表达式提供的值代替



（4） T orElseThrow(Supplier<? extends X> exceptionSupplier)



如果Optional容器中非空，就返回所包装值，如果为空，就抛出你指定的异常类型代替原来的NoSuchElementException



3、其他方法



（1）boolean isPresent()  ：判断Optional容器中的值是否存在



（2）void ifPresent(Consumer<? super T> consumer) ：



判断Optional容器中的值是否存在，如果存在，就对它进行Consumer指定的操作，如果不存在就不做



（3） Optional map(Function<? super T,? extends U> mapper)



判断Optional容器中的值是否存在，如果存在，就对它进行Function接口指定的操作，如果不存在就不做