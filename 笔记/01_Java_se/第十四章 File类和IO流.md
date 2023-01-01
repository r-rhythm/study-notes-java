# 第十四章 File类和IO流



```java
在要获取和设置文件或文件夹的属性信息，那么必须使用File类。
在要创建、删除、重命名文件或文件夹，那么必须使用File类。
这个文件或文件夹可以是本地的，也可以是网络中其他机器上的，当然无论在哪里，必须有准确的路径可以导航到这个文件或文件夹。
```



```java
当我们要读取一个文件中的内容，那么需要用到IO流。
当我们要把数据保存到文件中，那么需要用到IO流。
当我们需要在两个应用之间进行数据的交互，也需要用到IO流。这两个应用可能在同一台机器上，也可能是在网络中不同机器上。
```



## 14.1 java.io.File类



1、File类是用来在程序中表示一个文件或目录，这个文件或目录可能是真实存在的，也可能暂时不存在的。
如果它真实存在，即根据创建File类的“路径”能够找到这个文件或目录的，那么系统会将文件或目录的相关信息自动赋值给File类对象。
否则除了路径名（path,name等）之外，所有其他属性都是默认值。



2、File类API
（1）String getName()：获取文件或目录名
（2）long length()：只能得到文件的大小，返回字节数，如果是目录（文件夹）需要把它下面的所有文件大小都累加起来
（3）long lastModified()：获取文件或目录的最后时间，返回毫秒值
（4）boolean isFile()：如果文件不存在，就算是File类的对象是表示一个文件，也仍然返回false
只有真实存在的，确实是一个文件的，才会返回true
File f3 = new File("D:\Download\img\美女\chailinyan.jpg");[//这个chailinyan.jpg没有](http://xn--chailinyan-ws2pt914d.xn--jpg-8y9fz0w)
（5）boolean isDirectory()：如果目录不存在，就算是File类的对象是表示一个目录，也仍然返回false
只有真实存在的，确实是一个目录的，才会返回true
（6）boolean exists()：是否存在
（7）boolean canRead()：是否可读
（8）boolean canWrite()：是否可写
（9）boolean isHidden()：是否隐藏
（10）路径
A：标准的，在所有的操作平台都支持的路径分隔符“/"
B：在创建File时，构造器的（）中描述的路径名称为构造路径，它可以是绝对路径也可以是相对路径，
还可以包含“../”非规范路径。
C：String getAbsolutePath()：获取绝对路径
D：String getPath()：获取构造路径
E：String String getCanonicalPath()throws IOException：获取规范路径，会把构造路径中的../进行解析
（11）boolean createNewFile()：只能创建一个文件
（12）boolean mkdir()：只能创建一个文件夹，如果父目录不存在，创建不成功，但不报异常
boolean mkdirs()：创建文件夹，如果父目录不存在，会一并创建
（13） boolean delete() :只能删除文件或者说空目录
（14）boolean renameTo(File dest) ：当前的File对象调用了renameTo方法之后，就会把当前File对象的相关属性赋值给dest的File对象。
（15）String[] list()：获取某个目录的下一级的文件或目录的名称
（16）File[] listFiles()获取某个目录的下一级的文件或目录的File对象
（17）File[] listFiles(FileFilter f)获取某个目录的下一级的文件或目录的File对象，可以过滤File对象，看是否满足xx条件



```java
路径的问题：
1、绝对路径：
    windows来说，从盘符开始导航到具体的文件或文件夹，例如：D:/atguigu/javaee/JavaSE20220509/note/JavaSE每日复习笔记.md
    其他平台，从/（根路径）开始导航到具体的文件或文件夹，例如：/atguigu/javaee/JavaSE20220509/note/JavaSE每日复习笔记.md
2、相对路径：
    不是以盘符或/开头的，都是相对路径。
    例如：练习/Hello.txt

    相对路径必须找好"参照物"，“起点”。
    JavaSE:
        (1)main：相对于project
        (2)JUnit的@Test测试方法：相对于模块
    web：
        后面再讲
```



```java
学生问题：
   路径：\ 和 /什么区别？
   标准的，各个平台通用的是 /。
   \：只有windows支持，早期windows只支持\，之后也支持/。
```



## 14.2 IO流



### （1）四个抽象父类



InputStream：字节输入流
OutputStream：字节输出流
Reader：字符输入流
Writer：字符输出流



### （2）四个抽象父类的方法



**InputStream：**



int read()：读取一个字节，返回读取的字节值，如果到达流末尾，再读就返回-1。
int read(byte[] data)：一次最多读取data.length个字节，返回实际本次读取的字节数量。如果到达流末尾，再读就返回-1。
读取的数据就临时的存储到data中，从data[0]开始存储，
如果data数组循环使用的话，每一次读取都会从data[0]开始覆盖，
如果本次读取的字节数量没有凑够data.length个，本次一共读取count个字节，
那么要注意剩下的data[count, data.length-1]元素可能默认值0，也可能是上次读取的内容。
int read(byte[] data, int off, int len)：一次最多读取len个字节，返回实际本次读取的字节数量。如果到达流末尾，再读就返回-1。
读取的数据就临时的存储到data中，从data[off]开始存储，
void close()：关闭IO流



**Reader：**



int read()：读取一个字符，返回读取的字符的Unicode值，如果到达流末尾，再读就返回-1。
int read(char[] data)：一次最多读取data.length个字符，返回实际本次读取的字符数量。如果到达流末尾，再读就返回-1。
读取的数据就临时的存储到data中，从data[0]开始存储，
如果data数组循环使用的话，每一次读取都会从data[0]开始覆盖，
如果本次读取的字符数量没有凑够data.length个，本次一共读取count个字节，
那么要注意剩下的data[count, data.length-1]元素可能默认值0，也可能是上次读取的内容。
int read(char[] data, int off, int len)：一次最多读取len个字符，返回实际本次读取的字符数量。如果到达流末尾，再读就返回-1。
读取的数据就临时的存储到data中，从data[off]开始存储，
void close()：关闭IO流



**OutputStream：**



void write(int d)：输出一个字节
void write(byte[] data)：输出整个字节数组
void write(byte[] data,int off, int len)：输出字节数组data的len个字节，从data[off]开始
void close()



**Writer：**



void write(int d)：输出一个字符
void write(char[] data)：输出整个字符数组
void write(char[] data,int off, int len)：输出字节数组data的len个字符，从data[off]开始
void write(String str)：输出整个字符串
void write(String str,int off, int len)：输出字符串一部分，len个字符，从str[off]开始
void close()：关闭IO流



void flush()：刷新流中的数据



## 14.3 文件IO流



```java
FileInputStream：文件字节输入流
FileOutputStream：文件字节输出流
FileReader：文件字符输入流，用于读取纯文本文件，而且要求文件的编码和程序编码一致。
FileWriter：文件字符输出流，用于写数据到纯文本文件，而且要求文件的编码和程序编码一致。

FileInputStream和FileOutputStream适用于操作任意类型的文件。
FileReader和FileWriter只适用于操作纯文本文件，文件中不能有图片，视频等各种多媒体素材，即文件中只有各种字符。

FileInputStream和FileReader只能用来读取文件的内容。
FileOutputStream和FileWriter只能用来输出数据到文件。
    
当纯文本文件的字符编码和程序编码不一致时，怎么办？
需要使用FileInputStream和FileOutputStream，配合InputStreamReader和OutputStreamWriter才可以解决。
```



```java
什么是纯文本文件？
文件中只能出现字符。
通常纯文本文件的类型：
    .txt，.java，.html，.js,.css，.sql,.md ......
不是纯文本的文件：
    word，excel，ppt
    图片文件
    视频文件
    ...
```



```java
在创建文件IO流时，必须指定文件的具体路径。
不能是文件夹路径，否则会报拒绝访问异常。
如果是 FileInputStream和FileReader，对应的文件还必须存在，否则报FileNotFoundException异常。
如果是 FileOutputStream和FileWriter，对应的文件不存在，会自动创建。如果存在，没有指定追加模式的话，会覆盖。
```



```java
文件IO流的程序编写的步骤：
    （1）先创建文件IO流的对象
    如果是从文件读数据到程序中操作，选择FileInputStream、FileReader之一。
    如果是纯文本文件，并且没有编码不一致问题，那么可以选择FileReader，否则都是选择FileInputStream。
    
    如果是把数据写到文件中，即把数据保存到文件中，选择FileOutputStream、FileWriter之一。
    如果是纯文本文件，并且没有编码不一致问题，那么可以选择FileWriter，否则都是选择FileOutputStream。
    
    （2）读/写操作
    如果IO流是FileInputStream、FileReader：只能调用read方法进行读。
    如果IO流是FileOutputStream、FileWriter：只能调用write方法进行写。
    
    FileInputStream：借助byte[]进行读。
    FileReader：借助char[]进行读。
    
    （3）关闭
    IO流对象.close
```



## 14.4 缓冲IO流



```java
使用缓冲流的目的：提高效率

BufferedInputStream：给InputStream系列的IO加缓冲功能
BufferedOutputStream：给OutputStream系列的IO加缓冲功能
    
BufferedReader：给Reader系列的IO加缓冲功能
				或者是需要按行读取  String readLine()方法
BufferedWriter：给Writer系列的IO加缓冲功能
			 或者是需要按行输出  void newLine()
```



## 14.5 转换流



```java
InputStreamReader：
	（1）需要把字节输入流的类型"包装为"字符输入流的类型。被包装的是字节流，InputStreamReader本身是字符流，之后的代码基于InputStreamReader的字符流继续后面的操作，数据的话，从被包装流中得到的是字节流数据，但是从InputStreamReader中再read时就已经是字符数据了。
	（2）当读纯文本文件的数据时，遇到了文件的编码和程序的编码不一致，需要它配合FileInputStream完成。
    
OutputStreamWriter：
	（1）需要把字节输出流的类型"包装为"字符输出流的类型。被包装的是字节流，OutputStreamWriter本身是字符流，之后的代码基于OutputStreamWriter的字符流继续后面的操作，数据的话，write的是字符数据，通过OutputStreamWriter把字符数据”编码“为被包装流中的字节数据。
	（2）当输出纯文本数据时，遇到了文件的编码和程序的编码不一致，需要它配合FileOutputStream完成。
```



## 14.6 读写Java各种数据类型的IO流



```java
DataOutputStream：要输出的数据除了字符串之外还有其他Java的基本数据类型的数据或者字节数组		
DataInputStream：用于读取用DataOutputStream输出的数据

ObjectOutputStream：支持输出Java的各种数据，包括基本数据类型和对象。要输出Java对象时，配合Serializable接口。
ObjectInputStream：支持读取Java的各种数据类型的数据，包括基本数据和对象。要读取Java对象时，读取ObjectOutputStream输出的数据时。
```



序列化：



- 把Java的对象转为字节数据进行输出。

- 需要Java对象类型实现java.io.Serializable接口。如果Java对象中某个属性也是引用数据类型，那么这个属性想要参与序列化也要求该类型实现Serializable接口。

- 序列化过程中某些 成员变量的值是不能序列化：static和transient修饰的成员变量值是不序列化

- 为了在类做了修改之后，原来的数据仍然可以被反序列化，通常要求在实现Serializable接口时，加一个序列化版本ID。



```java
private static final long serialVersionUID = -6849794470754667710L;
```



反序列化：



- 把字节数据转换为一个Java对象。

- 反序列化时，要求Java对象对应的类必须存在，否则会报ClassNotFoundException。



ObjectOutputStream流中支持序列化的方法是：



- `public final void writeObject (Object obj)` : 将指定的对象写出。



ObjectInputStream流中支持反序列化的方法是：



- `public final Object readObject ()` : 读取一个对象。



ObjectOutputStream还支持将各种Java数据类型的数据写入输出流中：



- public void writeBoolean(boolean val)：写入一个 boolean 值。

- public void writeByte(int val)：写入一个8位字节。

- public void writeShort(int val)：写入一个16位的 short 值。

- public void writeChar(int val)：写入一个16位的 char 值。

- public void writeInt(int val)：写入一个32位的 int 值。

- public void writeLong(long val)：写入一个64位的 long 值。

- public void writeFloat(float val)：写入一个32位的 float 值。

- public void writeDouble(double val)：写入一个64位的 double 值。

- public void writeUTF(String str)：将表示长度信息的两个字节写入输出流，后跟字符串 s 中每个字符的 UTF-8 修改版表示形式。如果 s 为 null，则抛出 NullPointerException。根据字符的值，将字符串 s 中每个字符转换成一个字节、两个字节或三个字节的字节组。注意，将 String 作为基本数据写入流中与将它作为 Object 写入流中明显不同。



ObjectInputStream还支持从输入流中读取各种Java数据类型的数据：



- public boolean readBoolean()：读取一个 boolean 值。

- public byte readByte()：读取一个 8 位的字节。

- public short readShort()：读取一个 16 位的 short 值。

- public char readChar()：读取一个 16 位的 char 值。

- public int readInt()：读取一个 32 位的 int 值。

- public long readLong()：读取一个 64 位的 long 值。

- public float readFloat()：读取一个 32 位的 float 值。

- public double readDouble()：读取一个 64 位的 double 值。

- public String readUTF()：读取 UTF-8 修改版格式的 String。



注意：读的顺序和方法与写的顺序和方法要一一对应。



## 14.7 IO流的选择



```java
IO流的分类：
（1）根据数据的流向分：
输入流：InputStream、Reader系列，
        只能从输入流中读取数据，调用read系列的方法
        read() readInt()  readUTF()  readObject()....
输出流：OutputStream、Writer系列、包括PrintStream、PrintWriter
        只能往输出流中写数据，调用write，print系列的方法
        write(...)  writeInt(...) writeUTF(...) print(..) println(...)

（2）根据操作数据的基本单位分：
字节流：以字节为单位，所有的操作都要转为字节、字节数组
    InputStream和OutputStream系列
    FileOutputStream、BufferedOutputStream、DataOutputStream、ObjectOutputStream
    FileInputStream、BufferedInputStream、DataInputStream、ObjectInputStream

   PrintStream：从流的类型来说是字节流，但是它里面的print，println方法支持各种数据类型。

字符流：以字符为单位，只支持纯文本数据的处理
    Reader和Writer
    FileReader、BufferedReader、InputStreamReader、Scanner
    FileWriter、BufferedWriter、OutputStreamWriter、PrintWriter

（3）根据流的角色分：
节点流：FileInputStream、FileOutputStream、FileReader、FileWriter
	 或  
	 	ByteArrayInputStream、ByteArrayOutputStream
        CharArrayReader、CharArrayWriter
        StringReader、StringWriter
        
处理流：BufferedInputStream、BufferedOutputStream、BufferedReader、BufferedWriter   装饰流，需要包装别人才能
        DataInputStream、DataOutputStream
        ObjectInputStream、ObjectOutputStream
        InputStreamReader、OutputStreamWriter
        PrintStream、PrintWriter

IO流的整个体系设计是采用了“装饰者设计模式”。

什么是装饰者设计模式？
（1）被装饰者
A：节点流只能作为被装饰者
节点流（例如节点流：FileInputStream、FileOutputStream、FileReader、FileWriter），
节点流（例如节点流：ByteArrayInputStream、ByteArrayOutputStream），只能作为被装饰者
节点流（例如节点流：CharArrayReader、CharArrayWriter），只能作为被装饰者
它们的构造器指定的是数据的来源或者数据的目的地，例如：文件、数组。

B：装饰者同时也可以作为被装饰者


（2）装饰者
处理流，都是装饰者，必须依赖于其他的IO流
例如：BufferedInputStream bis = new BufferedInputStream(其他InputStream流的对象);
```



## 14.8 try...catch的新语法（JDK1.7之后）



```java
try(需要自动关闭的资源类对象的声明和创建){
    可能发生异常的业务逻辑代码
}catch(异常类型 e){
    异常处理代码
}【finally{
    其他必须执行的代码。
}】
```



注意：



（1）写在 try()中的资源类对象的类型必须实现AutoCloseable或Closeable接口



（2）写在 try()中的资源类对象自动加final



（3）资源类类的类型有哪些，通常有IO流类，后面有数据库连接类，网络连接类。这些通常操作完都需要close,，它们实现AutoCloseable或Closeable接口之后，就可以自动关闭，不用手动写close方法了。当执行完try...catch部分的代码之后，就会自动关闭。



（4）这些资源类对象确定之后不用了，才能使用这种try...catch形式自动关闭