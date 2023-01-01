# 第十章 基础API



API的学习方法：



（1）第一步：先记主题和包.类名（最好当天学当天就记住）



（2）第二步：每一个类记几个常用，或者有代表性的方法（方法就是功能）



根据这几个方法，能够通过查询API（查询笔记、查询百度等）文档了解更多的方法的功能、使用的说明。



（3）第三步：如果基础稍微好一点的，或者大家学到后面，对Java的语法、逻辑已经有点基础了，想要有更高的目标和要求时，可以尽量多看看一些类或方法的源码实现。



## 10.1 数学相关类



```java
java.lang.Math类：各种数学计算的方法
java.math包的BigInteger,BigDecimal：任意精度的整数、任意精度的小数
java.util.Random类：产生各种类型的随机值
    
面试题：基本数据类型、包装类、BigInteger、BigDecimal有什么区别？
    基本数据类型是非对象，表示数值数据比较简单，支持的运算符操作非常丰富。每一种数据类型在内存中的宽度是固定的。
    包装类：当在程序中用基本数据类型表示了数据，之后又遇到与面向对象（或者引用数据类型）相关的API操作时，可以把基本数据类型的数据转换为包装类的对象。它是和基本数据类型的数据一一对应。包装类和基本数据类型之间支持自动装箱和自动拆箱。
    BigInteger、BigDecimal也是引用数据类型，只支持整数和小数。在我们需要表示更大的、或者精度要求更高的情况下使用它们。BigInteger、BigDecimal对象的创建必须new，不支持和基本数据类型之间自动转换。
    
后面大家在高级还会接触到AtomicInteger, AtomicLong等。
```



```java
1、java.lang.Math类：
    Math 类包含用于执行基本数学运算的方法，如初等指数、对数、平方根和三角函数等。
    public static final double PI：返回圆周率
    public static final double E：返回自然对数的底数
    
    public static double abs(double a) ：返回 double 值的绝对值（absolute）。

    public static double ceil(double a) ：返回大于等于参数的最小的整数。
    public static double floor(double a) ：返回小于等于参数最大的整数。
    public static long round(double a) ：返回最接近参数的 long。(相当于四舍五入方法)

    public static double pow(double a,double b)：返回a的b幂次方法
    public static double sqrt(double a)：返回a的平方根
    public static double random()：返回[0,1)的随机值
    
    public static double max(double x, double y)：返回x,y中的最大值
    public static double min(double x, double y)：返回x,y中的最小值
    ...

2、java.math包下的BigInteger和BigDecimal类
它们是当你要处理特别大的整数、小数时使用。或者对小数都是精度要求比较高时使用。

（1）BigInteger不可变的任意精度的整数。
- BigInteger(String val)
- BigInteger add(BigInteger val)：加
- BigInteger subtract(BigInteger val)：减
- BigInteger multiply(BigInteger val)：乘
- BigInteger divide(BigInteger val)：除
- BigInteger remainder(BigInteger val)：余数
- ....
                                       
（2）RoundingMode枚举类
CEILING ：向正无限大方向舍入的舍入模式。
DOWN ：向零方向舍入的舍入模式。
FLOOR：向负无限大方向舍入的舍入模式。
HALF_DOWN ：向最接近数字方向舍入的舍入模式，如果与两个相邻数字的距离相等，则向下舍入。
HALF_EVEN：向最接近数字方向舍入的舍入模式，如果与两个相邻数字的距离相等，则向相邻的偶数舍入。
HALF_UP：向最接近数字方向舍入的舍入模式，如果与两个相邻数字的距离相等，则向上舍入。
UNNECESSARY：用于断言请求的操作具有精确结果的舍入模式，因此不需要舍入。
UP：远离零方向舍入的舍入模式。
                                       
（3）BigDecimal
不可变的、任意精度的有符号十进制数。
- BigDecimal(String val)
- BigDecimal add(BigDecimal val)
- BigDecimal subtract(BigDecimal val)
- BigDecimal multiply(BigDecimal val)
- BigDecimal divide(BigDecimal val)
- BigDecimal divide(BigDecimal divisor, int roundingMode)
- BigDecimal divide(BigDecimal divisor, int scale, RoundingMode roundingMode)
- BigDecimal remainder(BigDecimal val)
- ....


3、java.util.Random类
用于产生随机数

- boolean nextBoolean():返回下一个伪随机数，它是取自此随机数生成器序列的均匀分布的 boolean 值。
- void nextBytes(byte[] bytes):生成随机字节并将其置于用户提供的 byte 数组中。
- double nextDouble():返回下一个伪随机数，它是取自此随机数生成器序列的、在 0.0 和 1.0 之间均匀分布的 double 值。
- float nextFloat():返回下一个伪随机数，它是取自此随机数生成器序列的、在 0.0 和 1.0 之间均匀分布的 float 值。
- double nextGaussian():返回下一个伪随机数，它是取自此随机数生成器序列的、呈高斯（“正态”）分布的 double 值，其平均值是 0.0，标准差是 1.0。
- int nextInt():返回下一个伪随机数，它是此随机数生成器的序列中均匀分布的 int 值。
- int nextInt(int n):返回一个伪随机数，它是取自此随机数生成器序列的、在 0（包括）和指定值（不包括）之间均匀分布的 int 值。
- long nextLong():返回下一个伪随机数，它是取自此随机数生成器序列的均匀分布的 long 值。
```



## 10.2 日期时间相关类



```java
第一代：
    java.util.Date：日期时间表示类
    java.text.SimpleDateFormat：把日期时间转为想要的格式的字符串，或者也可以把某种格式的字符串转为日期时间对象
    
第二代：
    java.util.Calendar：日历
    java.util.TimeZone：表示时区
    java.util.Locale：表示国家和地区
    
第三代：JDK1.8引入
    java.time.LocalDate、LocalTime、LocalDateTime：本地的日期、时间
    java.time.ZoneDateTime：指定时区的日期、时间
    java.time.ZoneId：表示时区的ID值
    java.time.Period日期间隔、java.time.Duration时间间隔
    java.time.format.DateTimeFormatter ：日期时间格式化
```



```java
1、java.util.Date
new  Date()：当前系统时间
long  getTime()：返回该日期时间对象距离1970-1-1 0.0.0 0毫秒之间的毫秒值
new Date(long 毫秒)：把该毫秒值换算成日期时间对象
其他方法基本上已过时
```



```java
2、java.text.SimpleDateFormat
SimpleDateFormat用于日期时间的格式化。
- public final String format(Date date)将一个 Date 格式化为日期/时间字符串。 
- public Date parse(String source)throws ParseException从给定字符串的开始解析文本，以生成一个日期。
```





```java
3、java.util.TimeZone
通常，使用 getDefault 获取 TimeZone，getDefault  基于程序运行所在的时区创建 TimeZone。
也可以用 getTimeZone 及时区 ID 获取 TimeZone 。例如美国太平洋时区的时区 ID 是  "America/Los_Angeles"。
常见时区ID：
    Asia/Shanghai
    UTC
    America/New_York
```



```java
4、java.util.Locale
Locale 对象表示了特定的地理、政治和文化地区。需要 Locale  来执行其任务的操作称为语言环境敏感的 操作，它使用 Locale 为用户量身定制信息。
语言参数是一个有效的 ISO 语言代码。这些代码是由 ISO-639 定义的小写两字母代码。
国家/地区参数是一个有效的 ISO 国家/地区代码。这些代码是由 ISO-3166 定义的大写两字母代码。
Locale 类提供了一些方便的常量，可用这些常量为常用的语言环境创建 Locale 对象。
常见的有：CN,US
```



```java
5、java.util.Calendar
Calendar 类是一个抽象类，它为特定瞬间与一组诸如  YEAR、MONTH、DAY_OF_MONTH、HOUR  等 日历字段之间的转换提供了一些方法，并为操作日历字段（例如获得下星期的日期）提供了一些方法。瞬间可用毫秒值来表示，它是距历元（即格林威治标准时间 1970 年 1 月 1 日的 00:00:00.000，格里高利历）的偏移量。与其他语言环境敏感类一样，
    
Calendar 提供了一个类方法  getInstance，以获得此类型的一个通用的对象。
public static Calendar getInstance()使用默认时区和语言环境获得一个日历。
public static Calendar getInstance(TimeZone zone)使用指定时区和默认语言环境获得一个日历。
public static Calendar getInstance(Locale aLocale)使用默认时区和指定语言环境获得一个日历。
public static Calendar getInstance(TimeZone zone,Locale aLocale)使用指定时区和语言环境获得一个日历。
    
修改和获取 YEAR、MONTH、DAY_OF_MONTH、HOUR  等 日历字段对应的时间值。
void add(int field,int amount)
int get(int field)
void set(int field, int value)
```



Java1.0中包含了一个Date类，但是它的大多数方法已经在Java 1.1引入Calendar类之后被弃用了。而Calendar并不比Date好多少。它们面临的问题是：



- 可变性：象日期和时间这样的类对象应该是不可变的。Calendar类中可以使用三种方法更改日历字段：set()、add() 和 roll()。

- 偏移性：Date中的年份是从1900开始的，而月份都是从0开始的。

- 格式化：格式化只对Date有用，Calendar则不行。

- 此外，它们也不是线程安全的，不能处理闰秒等。



可以说，对日期和时间的操作一直是Java程序员最痛苦的地方之一。第三次引入的API是成功的，并且java 8中引入的java.time API 已经纠正了过去的缺陷，将来很长一段时间内它都会为我们服务。



Java 8 吸收了 Joda-Time 的精华，以一个新的开始为 Java 创建优秀的 API。



- java.time – 包含值对象的基础包

- java.time.chrono – 提供对不同的日历系统的访问。

- java.time.format – 格式化和解析时间和日期

- java.time.temporal – 包括底层框架和扩展特性

- java.time.zone – 包含时区支持的类



Java 8 吸收了 Joda-Time 的精华，以一个新的开始为 Java 创建优秀的 API。新的 java.time 中包含了所有关于时钟（Clock），本地日期（LocalDate）、本地时间（LocalTime）、本地日期时间（LocalDateTime）、时区（ZonedDateTime）和持续时间（Duration）的类。



1、本地日期时间：LocalDate、LocalTime、LocalDateTime

| 方法                                                         | **描述**                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| now() / now(ZoneId zone)                                     | 静态方法，根据当前时间创建对象/指定时区的对象                |
| of(参数)                                                     | 静态方法，根据指定日期/时间创建对象                          |
| getDayOfMonth()/getDayOfYear()                               | 获得月份天数(1-31) /获得年份天数(1-366)                      |
| getDayOfWeek()                                               | 获得星期几(返回一个 DayOfWeek 枚举值)                        |
| getMonth()                                                   | 获得月份, 返回一个 Month 枚举值                              |
| getMonthValue() / getYear()                                  | 获得月份(1-12) /获得年份                                     |
| getHours()/getMinute()/getSecond()                           | 获得当前对象对应的小时、分钟、秒                             |
| withDayOfMonth()/withDayOfYear()/withMonth()/withYear()      | 将月份天数、年份天数、月份、年份修改为指定的值并返回新的对象 |
| with(TemporalAdjuster  t)                                    | 将当前日期时间设置为校对器指定的日期时间                     |
| plusDays(), plusWeeks(), plusMonths(), plusYears(),plusHours() | 向当前对象添加几天、几周、几个月、几年、几小时               |
| minusMonths() / minusWeeks()/minusDays()/minusYears()/minusHours() | 从当前对象减去几月、几周、几天、几年、几小时                 |
| plus(TemporalAmount t)/minus(TemporalAmount t)               | 添加或减少一个 Duration 或 Period                            |
| isBefore()/isAfter()                                         | 比较两个 LocalDate                                           |
| isLeapYear()                                                 | 判断是否是闰年（在LocalDate类中声明）                        |
| format(DateTimeFormatter  t)                                 | 格式化本地日期、时间，返回一个字符串                         |
| parse(Charsequence text)                                     | 将指定格式的字符串解析为日期、时间                           |



2、指定时区日期时间：ZondId和ZonedDateTime



常见时区ID：



```java
Asia/Shanghai
UTC
America/New_York
```



可以通过ZondId获取所有可用的时区ID。



3、持续日期/时间：Period和Duration



Period:用于计算两个“日期”间隔



Duration:用于计算两个“时间”间隔



4、DateTimeFormatter：日期时间格式化



该类提供了三种格式化方法：



-  预定义的标准格式。如：ISO_DATE_TIME;ISO_DATE 

-  本地化相关的格式。如：ofLocalizedDate(FormatStyle.MEDIUM) 

-  自定义的格式。如：ofPattern(“yyyy-MM-dd hh:mm:ss”) 



## 10.3 系统信息相关类



```java
java.lang.System：系统类
java.lang.Runtime：JVM运行环境类
```



```java
1、java.lang.System类：系统类
- static long currentTimeMillis() ：返回当前系统时间距离1970-1-1 0:0:0的毫秒值
- static void exit(int status) ：退出当前系统
- static void gc() ：运行垃圾回收器。
- static String getProperty(String key)：获取某个系统属性，例如：java.version、user.language、user.country、file.encoding、user.name、os.version、os.name等等

System.in
System.out
System.err

2、java.lang.Runtime类：JVM运行环境类
    Runtime类是一个单例类，构造器私有化。有一个唯一的对象。
public static Runtime getRuntime()： 返回与当前 Java 应用程序相关的运行时对象。
public long totalMemory()：返回 Java 虚拟机中的内存总量。此方法返回的值可能随时间的推移而变化，这取决于主机环境。
public long freeMemory()：回 Java 虚拟机中的空闲内存量。调用 gc 方法可能导致 freeMemory 返回值的增加。
```



## 10.4 数组工具类



```java
1、java.util.Arrays数组工具类
2、java.lang.System类的
    public static void arraycopy(Object src,
                             int srcPos,
                             Object dest,
                             int destPos,
                             int length)
```



```java
1、java.util.Arrays数组工具类
java.util.Arrays数组工具类，提供了很多静态方法来对数组进行操作，而且如下每一个方法都有各种重载形式，以下只列出int[]和Object[]类型的，其他类型的数组依次类推：

- 数组元素拼接
- static String toString(int[] a) ：字符串表示形式由数组的元素列表组成，括在方括号（"[]"）中。相邻元素用字符 ", "（逗号加空格）分隔。形式为：[元素1，元素2，元素3。。。]
-static String toString(Object[] a) ：字符串表示形式由数组的元素列表组成，括在方括号（"[]"）中。相邻元素用字符 ", "（逗号加空格）分隔。元素将自动调用自己从Object继承的toString方法将对象转为字符串进行拼接，如果没有重写，则返回类型@hash值，如果重写则按重写返回的字符串进行拼接。
- 数组排序
  - static void sort(int[] a) ：将a数组按照从小到大进行排序
  - static void sort(int[] a, int fromIndex, int toIndex) ：将a数组的[fromIndex, toIndex)部分按照升序排列
  - static void sort(Object[] a) ：根据元素的自然顺序对指定对象数组按升序进行排序。
  - static <T> void sort(T[] a, Comparator<? super T> c) ：根据指定比较器产生的顺序对指定对象数组进行排序。
- 数组元素的二分查找
  - static int binarySearch(int[] a, int key) 
  - static int binarySearch(Object[] a, Object key) ：要求数组有序，在数组中查找key是否存在，如果存在返回第一次找到的下标，不存在返回负数
- 数组的复制
  - static int[] copyOf(int[] original, int newLength)  ：根据original原数组复制一个长度为newLength的新数组，并返回新数组
  - static <T> T[] copyOf(T[] original,int newLength)：根据original原数组复制一个长度为newLength的新数组，并返回新数组
  - static int[] copyOfRange(int[] original, int from, int to) ：复制original原数组的[from,to)构成新数组，并返回新数组
  - static <T> T[] copyOfRange(T[] original,int from,int to)：复制original原数组的[from,to)构成新数组，并返回新数组
- 比较两耳数组是否相等
  - static boolean equals(int[] a, int[] a2) ：比较两个数组的长度、元素是否完全相同
  - static boolean equals(Object[] a,Object[] a2)：比较两个数组的长度、元素是否完全相同
- 填充数组
  - static void fill(int[] a, int val) ：用val值填充整个a数组
  - static void fill(Object[] a,Object val)：用val对象填充整个a数组
  - static void fill(int[] a, int fromIndex, int toIndex, int val)：将a数组[fromIndex,toIndex)部分填充为val值
  - static void fill(Object[] a, int fromIndex, int toIndex, Object val) ：将a数组[fromIndex,toIndex)部分填充为val对象
```



```java
2、java.lang.System类中有一个数组复制方法：
    public static void arraycopy(Object src,
                             int srcPos,
                             Object dest,
                             int destPos,
                             int length)

      第一个参数：原数组
      第二个参数：原数组起始下标（从左边数的起始下标值，实际复制过程可能是从右边开始先复制）
      第三个参数：目标数组
      第四个参数：目标数组起始下标（从左边数的起始下标值，实际复制过程可能是从右边开始先复制）
      第五个参数：要复制的元素个数

  注意：
  （1）为什么它声明在System类中？
    因为Arrays类是JDK1.2才有的，而这个方法存在的时间比较早，JDK1.0就有了，当时没有数组工具类，
    这个方法又比较基础，重要，当时最早期的很多集合类型（数据结构）的底层实现都用它了，所以放在System类中。

  （2）为什么参数1和参数3表示数组，却声明为Object类型，
  不是int[]，不是Object[]类型呢？
  因为如果是int[]，那么只能接收int[]类型的数组，其他类型的数组，需要提供多个重载方法。
  如果是Object[]类型，那么只能接收对象数组，不能接收基本数据类型的一维数组，同样需要提供多个重载方法。
  设计为Object类型，可以接收任何一个类型的数组对象。

  （3）如果第一个参数和第三个参数，传入的是同一个数组对象，该方法表示在本数组中移动元素。
    		第二个参数 > 第四个参数，表示元素往前移动，一般用于删除
    		第二个参数 < 第四个参数，表示元素往后移动，一般用于插入
     如果第一个参数和第三个参数，传入的不是同一个数组对象，该方法表示在两个数组之间复制元素。
```



## 10.5 比较器



```java
1、自然比较接口：java.lang.Comparable
    	int compareTo(Object obj)
2、定制比较接口：java.util.Comparator
    	int compare(Object o1, Object o2)
3、java.text.Collator，实现了Comparator接口
    	int compare(String s1, String s2)方法
```



Java中涉及到两个对象比较大小，都优先考虑让这个对象的类实现Comparable接口，重写compareTo方法。



Java的核心类库中，常用的类都实现了这个接口：



- java.lang.String

- java.lang.Integer

- java.lang.Enum类（所有枚举类的公共父类），表示所有枚举类都是实现这个接口了，都可以比较大小

- 。。。



当要比较大小的类无法修改代码（可能用的第三方的类），无法让对象的类实现Comparable接口，可以考虑单独编写一个类实现Comparator接口，实现对两个对象的大小比较。



或者是，当对象的类已经实现了Comparable接口，也重写了compareTo方法，这是方法体中实现的比较大小的方案，不适合当前的新需求，而compareTo方法中的代码还要用，此时就可以考虑单独编写一个类（通常用匿名内部类）实现Comparator接口，实现对两个对象的大小比较的新需求。



## 10.6 String类



### 10.6.1 String类以及String对象的特点



（1）String类本身是final修饰，它不能被继承



```java
和String一样是final修饰的还有Math，System类等
```



（2）String类的对象也是不可变对象，因为不可变，意味着字符串常量对象是可以共享的。字符串常量对象存储在常量池。



```java
和String类的对象一样不可变的还有，包装类、第三代的日期时间对象等
```



（3）String类的内部使用什么来表示一串字符



JDK1.9之前：private final char[] value;



JDK1.9之后：private final byte[] value;



（4）字符串对象的创建



```java
（1）直接""，它是在常量池，可能会出现共享的情况，如果和常量池中的另一个字符串内容相同，就会共享。
（2）构造器，会在堆中new一个，如果()中还有""，还会引用常量池中内容
（3）String类的valueOf()，和构造器一样
（4）任意引用数据类型的对象调用toString()
（5）任意类型的数据与字符串“+”拼接结果都是字符串。其中引用数据类型与字符串“+”时，也会自动调用引用数据类型的toString方法。
```



（5）字符串对象的个数



```java
字符串常量池中的可以共享，只要字符串内容相同，就是同一个对象。==比较地址值相同。
非字符串常量池中的字符串不共享。就算内容相同，==比较地址值结果也不同。

    凡是用new的，或者用valueOf方法等方式，每次都是产生新对象。
    凡是直接""构建的字符串，内容相同，就是同一个字符串对象。
    
提醒：
    无论是否是字符串常量，开发中涉及到两个字符串的比较是否“相等”问题，都调用equals,equalsIgnoreCase方法，不要用==，除非你确实是要比较地址值。
```



### 10.6.2 String类的API



1、空字符串



```java
空字符串是指，一个字符串对象，但是它的长度为0，没有任意字符。不能是null，不能是未初始化。
空字符的判断：
    假设String类型的变量是str
    "".equals(str)
    str!=null && str.equals("")
    str!=null && str.isEmpty()
    str!=null && str.length()==0
    
如果null值也被视为空的话：
    str==null || str.equals("")
    str==null || str.isEmpty()
    str==null || str.length()==0
```



2、字符串的比较



```java
（1）==：比较两个字符串的地址，只有都是字符串常量池中两个字符串==，并且内容相同比较结果才会true
（2）equals
    严格区分大小写：boolean equals(Object obj)，重写Object类的equals方法
    不区分大小写：boolean equalsIgnoreCase(String anotherString) ，这String类自己扩展的方法
（3）compare
    严格区分大小写：int compareTo（String str）：按照字符的Unicode值，因为String类实现java.lang.Comparable接口。
    不区分大小写：int compareToIgnoreCase(String str) ，这是String自己扩展的方法。
（4）定制比较大小
    java.text.Collator里面的compare方法，按照语言的自然顺序，例如中文按照拼音的顺序。这个类实现了java.util.Comparator接口。
    或者是自定义类实现Comparator接口，自己写的比较规则。
```



```java
package com.atguigu.review;

import org.junit.Test;

import java.text.Collator;
import java.util.Arrays;
import java.util.Comparator;
import java.util.Locale;

public class TestStringCompare {
    @Test
    public void test02(){
        String[] arr = {"张三","李四","王五","柴林燕"};

        System.out.println("初始顺序：" + Arrays.toString(arr));
        Arrays.sort(arr);
        System.out.println("自然顺序：" + Arrays.toString(arr));
        Arrays.sort(arr, new Comparator(){
            @Override
            public int compare(Object o1, Object o2) {
                Collator collator = Collator.getInstance(Locale.CHINA);
                return collator.compare(o1,o2);
            }
        });
        System.out.println("定制顺序（自然语言，字典顺序）：" + Arrays.toString(arr));
        //中文，类似于拼音的顺序，对于多音字来说，可能识别上不一定非常准确
    }

    @Test
    public void test01(){
        String[] arr = {"hello","Hi","atguigu","Java","WORLD"};

        System.out.println("初始顺序：" + Arrays.toString(arr));
        /*
        自然排序规则：实现了java.lang.Comparable接口，
                    重写int compareTo(Object obj)

         java.util.Arrays类：
            public static void sort(Object[] a)：根据元素的自然顺序对指定对象数组按升序进行排序。数组中的所有元素都必须实现 Comparable 接口。

          String类实现了java.lang.Comparable接口，
                    重写int compareTo(Object obj)：按照每一个字符的Unicode编码值，严格区分大小写
         */
        Arrays.sort(arr);
        System.out.println("自然顺序：" + Arrays.toString(arr));

        /*
        定制排序：实现了java.util.Comparator接口
                重写int compare(Object o1, Object o2)

         java.util.Arrays类：
                public static <T> void sort(T[] a, Comparator<? super T> c)
                T：暂时看成Object类型
                <? super T>：先忽略它
                类似于：
                public static void sort(Object[] a, Comparator c)

         String类中提供了一个
            不区分大小写：int compareToIgnoreCase(String str)
         */
        Arrays.sort(arr, new Comparator(){
            @Override
            public int compare(Object o1, Object o2) {
                String s1 = (String) o1;
                String s2 = (String) o2;
                return s1.compareToIgnoreCase(s2);
            }
        });

        System.out.println("定制顺序1（不区分大小写，比较编码值）：" + Arrays.toString(arr));

        /*
        java.text.Collator：文本校对器，执行区分语言环境的 String 比较。
            这个类实现了java.util. Comparator接口，重写了 int compare(Object o1, Object o2)
            这是一个抽象类，它的对象需要直接new子类对象，或调用静态方法getInstance来获取子类对象
         */
        Arrays.sort(arr, new Comparator(){
            @Override
            public int compare(Object o1, Object o2) {
                String s1 = (String) o1;
                String s2 = (String) o2;
                Collator collator = Collator.getInstance(Locale.US);
                return collator.compare(s1,s2);
            }
        });

        System.out.println("定制顺序2（按照语言的自然顺序，字典顺序）：" + Arrays.toString(arr));
        //对于英文来说，类似于不区分大小写的效果

    }
}
```



3、字符串的拼接



```java
（1）+：支持字符串与字符串的拼接，也支持字符串与其他类型的拼接
    
    A：当两个字符串常量池中的字符串常量 + ，结果仍然在字符串常量池中寻找
    ① 两个直接""引起来的字符串拼接，
    		例如："hello"+"world" 直接等价于"helloworld"
    ② 两个final声明的最终变量拼接，并且它们被赋值为直接""引起来的字符串
    		例如：final String s1 = "hello";
                 final String s2 = "world";
					 s1 + s2 或 "hello" + s2 或 s1 +"world" 结果都在字符串常量池中寻找
    B：当有非final的变量参与，那么结果一定在堆
            例如：String s1 = "hello";
                 String s2 = "world";
					 s1 + s2 或 "hello" + s2 或 s1 +"world" 结果
    C：当有new的String对象参与，那么结果也一定在堆
    
（2）concat：只支持两个字符串的拼接
    字符串1.concat(字符串2)，结果也一定在堆
                         
（3）intern:无论哪种字符串拼接之后调用intern()，结果都在常量池中。
```



4、其他常用API



（1）boolean isEmpty()：字符串是否为空



（2）int length()：返回字符串的长度



（3）String concat(xx)：拼接，运行效果等价于+



（4）boolean equals(Object obj)：比较字符串是否相等，区分大小写



（5）boolean equalsIgnoreCase(Object obj)：比较字符串是否相等，不区分大小写



（6）int compareTo(String other)：比较字符串大小，区分大小写，按照Unicode编码值比较大小



（7）int compareToIgnoreCase(String other)：比较字符串大小，不区分大小写



（8）String toLowerCase()：将字符串中大写字母转为小写



（9）String toUpperCase()：将字符串中小写字母转为大写



（10）String trim()：去掉字符串前后空白符



（11）public String intern()：结果在常量池中共享



（12）boolean contains(xx)：是否包含xx



（13）int indexOf(xx)：从前往后找当前字符串中xx，即如果有返回第一次出现的下标，要是没有返回-1



（14）int lastIndexOf(xx)：从后往前找当前字符串中xx，即如果有返回最后一次出现的下标，要是没有返回-1



（15）String substring(int beginIndex) ：返回一个新的字符串，它是此字符串的从beginIndex开始截取到最后的一个子字符串。



（16）String substring(int beginIndex, int endIndex) ：返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex(不包含)的一个子字符串。



（17）char charAt(index)：返回[index]位置的字符



（18）char[] toCharArray()： 将此字符串转换为一个新的字符数组返回



（19）String(char[] value)：返回指定数组中表示该字符序列的 String。



（20）String(char[] value, int offset, int count)：返回指定数组中表示该字符序列的 String。



（21）static String copyValueOf(char[] data)： 返回指定数组中表示该字符序列的 String



（22）static String copyValueOf(char[] data, int offset, int count)：返回指定数组中表示该字符序列的 String



（23）static String valueOf(char[] data, int offset, int count) ： 返回指定数组中表示该字符序列的 String



（24）static String valueOf(char[] data)  ：返回指定数组中表示该字符序列的 String



（25）byte[] getBytes()：编码，把字符串变为字节数组，按照平台默认的字符编码方式进行编码



​	     byte[] getBytes(字符编码方式)：按照指定的编码方式进行编码



（26）new String(byte[] ) 或 new String(byte[], int, int)：解码，按照平台默认的字符编码进行解码



​      new String(byte[]，字符编码方式 ) 或 new String(byte[], int, int，字符编码方式)：解码，按照指定的编码方式进行解码



（27）boolean startsWith(xx)：是否以xx开头



（28）boolean endsWith(xx)：是否以xx结尾



（29）boolean matchs(正则表达式)：判断当前字符串是否匹配某个正则表达式。



（30）String replace(被替换字符 ,  替换后的字符)：不支持正则



（31）String replaceFirst(正则，value)：替换第一个匹配部分，当前字符串中第一个匹配正则的部分会被替换为value



（32）String repalceAll(正则， value)：替换所有匹配部分，当前字符串中所有匹配正则的部分会被替换为value



（33）String[] split(正则)：按照某种规则进行拆分



## 10.7 StringBuffer和StringBuilder字符串缓冲区



常用的API，StringBuilder、StringBuffer的API是完全一致的



（1）StringBuffer append(xx)：拼接，追加



（2）StringBuffer insert(int index, xx)：在[index]位置插入xx



（3）StringBuffer delete(int start, int end)：删除[start,end)之间字符



StringBuffer deleteCharAt(int index)：删除[index]位置字符



（4）void setCharAt(int index, xx)：替换[index]位置字符



（5）StringBuffer reverse()：反转



（6）void setLength(int newLength) ：设置当前字符序列长度为newLength



（7）StringBuffer replace(int start, int end, String str)：替换[start,end)范围的字符序列为str



（8）int indexOf(String str)：在当前字符序列中查询str的第一次出现下标



​      int indexOf(String str, int fromIndex)：在当前字符序列[fromIndex,最后]中查询str的第一次出现下标



​     int lastIndexOf(String str)：在当前字符序列中查询str的最后一次出现下标



​     int lastIndexOf(String str, int fromIndex)：在当前字符序列[fromIndex,最后]中查询str的最后一次出现下标



（9）String substring(int start)：截取当前字符序列[start,最后]



（10）String substring(int start, int end)：截取当前字符序列[start,end)



（11）String toString()：返回此序列中数据的字符串表示形式



## 10.8 String类和StringBuffer、StringBuilder区别



String类的对象是不可变，因为不可变，只要是字符串常量就可以共享，又是因为不可变，对字符串的修改就会返回新对象，注意接收。



StringBuffer、StringBuilder都是可变字符串，无论是增、删、改都是针对原对象进行修改，不需要重新接收。在大量的修改（特别是拼接）时，StringBuffer、StringBuilder的效率相对高一点。



StringBuffer、StringBuilder之间的区别：



StringBuffer旧一点，线程安全的。



StringBuilder新一点，线程不安全的。