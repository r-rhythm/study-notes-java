## SpringMVC中消息转换器

### 概述

* HttpMessageConverter<T>负责将请求信息转换为一个对象
* 此接口默认的消息转换器有7个

### 使用消息转换器,处理请求数据

* @RequestBody 获取请求体

  * 注意,只有POST才有请求体

  * ```java
    /**
         * 测试@RequestBody自动获取请求体
         *
         * @param requestBody
         * @return
         */
        @PostMapping("/requestBody")
        public String testRequestBody(@RequestBody(required = false) String requestBody) {
            System.out.println("requestBody = " + requestBody);
            //requestBody = name=liuwei&password=123qwe
            return "success";
        }
    ```

  * ```html
    <form th:action="@{/requestBody}" method="post">
        姓名:<input type="text" name="name"><br>
        密码:<input type="password" name="password"><br>
        <input type="submit" value="测试@RequestBody">
    </form>
    ```

    

* 使用HttpEntity<T> 获取请求头和请求体

  * ```java
    /**
         * 测试使用HttpEntity获取请求头/行信息
         *
         * @param httpEntity
         * @return
         */
        @PostMapping("/testHttpEntity")
        public String testHttpEntity(HttpEntity<String> httpEntity) {
            
            HttpHeaders headers = httpEntity.getHeaders();
            
            String body = httpEntity.getBody();
            
            System.out.println("headers = " + headers);
            System.out.println("body = " + body);
            
            return "success";
        }
    ```

  * ```html
    <div align="center">
    
        <form th:action="@{/testHttpEntity}" method="post">
            姓名:<input type="text" name="name"><br>
            密码:<input type="password" name="password"><br>
            <input type="submit" value="测试HttpEntity">
        </form>
    
    </div>
    ```

    

### 使用消息转换器,处理响应数据[重要]

> 使用响应流区响应数据

* @ResponseBody

  * 作用:使用响应流响应数据

    * ```java
      /**
           * 测试@ResponseBody
           * 作用:添加这个注解后,不再跳转到return的页面,
           * 而是直接使用响应流响应数据
           *
           * @return
           */
          @RequestMapping("/testResponseBody")
          @ResponseBody
          public String testResponseBody() {
              
              //将字符串success使用响应流响应到用户页面
              return "success";
              
          }
      ```

    * ```html
      <a th:href="@{/testResponseBody}">测试@ResponseBody</a>
      ```

      

  * **使用@ResponseBody处理Json数据[非常重要]**

    * 添加jar包

      * ```xml
        <dependency>
                    <groupId>com.fasterxml.jackson.core</groupId>
                    <artifactId>jackson-databind</artifactId>
                    <version>2.12.3</version>
                </dependency>
        ```

    * 在springmvc.xml中添加`<mvx:annotation-driven>`

      * ```xml
        <!--    装配annotation-driven-->
            <mvc:annotation-driven></mvc:annotation-driven>
        ```

      * ```java
        /**
             * 测试@ResponseBody处理Json数据
             *  1.导入<jackson-databind> jar包
             *  2.springmvc.xml中加入<mvx:annotation-driven>
             * @return
             */
            @RequestMapping("/testResponseJson")
            @ResponseBody
            public List<Employee> testResponseJson(){
            
                List<Employee> allEmps = employeeService.getAllEmps();
                //直接将结果集返回,@ResponseBody会自动处理为Json格式信息并响应
                return allEmps;
            }
        ```

        

* ResponseEntity<T>

  * 作用,实现文件的下载

  