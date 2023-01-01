
# spring-security

## 用户验证核心组件
* Authenticatoin 存储了认证信息,代表当前登录用户,它存储的信息有
	* Principal 用户信息,在用户还未认证时存储着用户名,认证后存储的是用户对象
	* Credentials 用户凭证,密码
	* Authorities 用户权限
* SecurityContext 上下文对象,可以在通过它获取到Authentication
* SecurityContextHolder 上下文管理对象,用在程序任何地方获取SecurityContext对象*
* `Authentication authentication = SecurityContextHolder.getContext().getAuthentication();`

## spring-security如何进行用户验证
### 简要
1. 在UsernamePasswordAuthenticationFilter过滤器`attemptAuthentication`方法内拿到请求的用户名及密码
	1. 获得一个authRequest未认证对象,这个对象内有Principal,Credentials



## 在项目内添加spring-security
* 创建spring-security模块,添加依赖
* 创建配置类,继承WebSecurityConfigurerAdapter
* 在业务模块引入spring-security模块依赖

	*spring-security有默认的`UserDetialsService / UserDetils / PasswordEncoder`这三个组件的实现,但这些实现一般无法满足项目需求,所以要对其进行扩展*

### 拓展AuthenticationManager校验所调用的三个组件
* 实现PasswordEncoder接口,重写拓展两个方法
* 对UserDetails组件进行拓展,继承Spring-Security提供的User类,在里面拓展我们的用户实体类
* 对业务对象UserDetailsService拓展,这个接口是只有一个方法,通过用户名获取对象,创建一个UserDetailsServiceImpl类,实现这个接口,并给他返回一个拓展后的UserDetails*

### 自定义认证
* 自定义登陆过滤器,继承UsernamePasswordAuthenticationFileter,重写身份验证/验证成功/验证失败三个方法
* 自定义Token过滤器继承OncePerRequestFile,获取前台传入的token并解析token.**这个过滤器会在每次请求时执行,并将用户的可信的token令牌信息存放到上下文中**
* 重写配置类中的方法*
