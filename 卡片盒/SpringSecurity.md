
> [!info] Spring-Security
> Spring-Security是一个强大的高度自定义的访问控制框架,他为[[Spring]]应用提供访问时身份验证及授权

# 开启基于Controller的权限控制
-   Controller层的@PreAuthorize(**"hasAnyAuthority()"**)注解,指定调用此方法所需要的权限,如果用户没有这个指定的权限,是无法调用这个方法的

-   在Spring-Security配置类上使用 @EnableGlobalMethodSecurity(prePostEnabled = **true**) 注解，开启基于Controller的权限控制(没有相应权限无法调用controller)

-   hasAnyAuthority('**bnt.sysMenu.remove'**)内的属性是一个String字符串,所以要使用单引号引起来 