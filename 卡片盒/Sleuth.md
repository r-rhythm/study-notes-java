# 简介
> [!info] 分布式调用追踪
> * Sleuth是一个宽泛的思想,它集成了zipkin提供具体实现
> * 通过它可以得到当前这个服务的调用链
> * 通过这个调用链可以判断各个被调用的服务运行是否正常,利于问题的排查

# 详解
Span id:当前服务的id
Parent id:上层调用,谁调用了它