# 概述
Spring 框架中的一个定时任务模块

# 注解
- 在类上方加上@EnableScheduling 表示开启定时任务,这个类要使用@Component 注解交由 Spring 管理
- 在方法上方添加@Scheduled(cron = "") 并通过 cron 七域表达式设置执行的规则,*cron 表达式在 [[Linux#cront 定时任务设置]]学习过*
- Cron 表达式中的年是默认忽略的, 因为它是无法跨年使用的
- 秒/分/时/日/月/周