# 统一返回数据格式
1. 在不确定返回结果的类型是可以使用Map,他的优势是取值存值都非常方便
# 条件分页查询接口
1. 配置分页插件
2. 使用Page直接查询封装后返回即可
3. **条件查询类仅仅是将查询条件封装为一个对象以便于处理查询,而不是其他的目的**
4. 实现了普通的增删改查接口
# [[统一异常处理]]
```java
import com.atguigu.yygh.common.result.R;  
import org.springframework.web.bind.annotation.ControllerAdvice;  
import org.springframework.web.bind.annotation.ExceptionHandler;  
import org.springframework.web.bind.annotation.ResponseBody;  
  
/**  
 * @author: Wei  
 * @date: 2022/10/18,11:16  
 * @description: 全局异常处理  
 */  
@ControllerAdvice // 以通知的形式处理异常  
public class GlobalExceptionHandler {  
      
    /**  
     * 处理全局异常  
     *  
     * @param e  
     * @return  
     */  
    @ExceptionHandler(Exception.class)  
    @ResponseBody // 响应Json数据  
    public R error(Exception e) {  
        e.printStackTrace();  
        return R.error().message(e.getMessage());  
    }  
      
    @ExceptionHandler(ArithmeticException.class)  
    @ResponseBody  
    public R error(ArithmeticException e){  
        return R.error().message("发生了除0异常");  
    }  
      
}
```
# [[日志]]

# [[前端基础知识]]