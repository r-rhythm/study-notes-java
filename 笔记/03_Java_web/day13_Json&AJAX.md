# Json

> JavaScript Object Notation
> Json 可以在不同语言,不同生态间进行数据交互

- 将 Json 对象转换为 Json 字符串 `JSON.stringify()`
  - `var jsonString = JSON.stringify(Obj)
- 将 Json 字符串转换为 Json 对象 `JSON.parse()`
  - `var jsonObj = JSON.parse(string)`
## 在 Java 语言环境下 Json 类型转换

> JavaWeb 阶段使用 Gson 处理 Java 对象与 Json 字符串转换问题
> SSM 使用 Jackson2


 * 将 Json 字符串转换为 Java 对象
  * `gson.fromJson(Str,Class clazz)`
* 将对象转换为 Json 字符串 `gson.toJson(Obj)`
```java
@Test
    public void testBeanToJson() {
        Gson gson = new Gson();
        Student s1 = new Student(123, "你好", 12);
         String Json1 = gson.toJson(s1);

         System.out.println("Json1 = " + Json1);

         //将 JsonString 转换为 JavaBean
         Student student = gson.fromJson(Json1, Student.class);

         System.out.println("studentName = " + student.getStuName());


     }
 ```


List<T>

> 将 List 字符串转化为 List 泛型需要一个匿名内部类 TypeToken
>
> * `gson.fromJson(Str,new TypeToken<List<Student>>(){}.getType())`
>
 ```java
 @Test
     public void testListToJson() {

         Gson gson = new Gson();

         List<Student> stu = new ArrayList<>();
         stu.add(new Student(123, "你好", 12));
         stu.add(new Student(321, "枫", 17));
         stu.add(new Student(789, "白落狄", 19));

         //将 List 转换为 Json 字符串
         String ListJson = gson.toJson(stu);

         System.out.println("ListJson = " + ListJson);

         //将 Json 字符串转换为 Java 对象
         List<Student> student = gson.fromJson(ListJson, new TypeToken<List<Student>>() {
         }.getType());

         System.out.println("student = " + student);
         System.out.println("student = " + student.get(2));

     }
 ```

Map<K,V>

> `gson.fromJson(Str,new TypeToken<Map<K,V>>(){}.getType())`
 ```java
 @Test
     public void testMapToJson() {
         Gson gson = new Gson();

         //创建 map
         Map<String, Student> map = new HashMap<>();
         map.put("y", new Student(123, "英宋", 19));
         map.put("m", new Student(321, "manG", 17));
         map.put("f", new Student(890, "枫", 18));

         //将 map 转换为 Json
         String mapJson = gson.toJson(map);
         System.out.println("mapJson = " + mapJson);


         //将 JsonString 转换为 Map
         Map<String, Student> stuInfo = gson.fromJson(mapJson, new TypeToken<Map<String, Student>>() {
         }.getType());

         Student f = stuInfo.get("f");
         System.out.println("f = " + f);
         System.out.println("f.getStuName() = " + f.getStuName());

     }
 ```

[[Axios]]

# Axios
 它是发送 Ajax 请求的框架

**Js 函数也是对象,也可调用方法**

## Axios 中两种参数
![[Axios#Axios 中两种参数]]
