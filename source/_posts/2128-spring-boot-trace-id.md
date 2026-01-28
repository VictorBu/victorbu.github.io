---
title: Spring Boot 日志中增加 trace id 用于链路追踪
date: 2023-06-21 10:00:00
updated: 2023-06-21 10:00:00
categories: [IT]
tags: [Logback, Spring Boot, Java]
---

使用 Logback 的 MDC(Mapped Diagnostic Contexts) 可以实现此功能。

# 新增 MDC 工具类

```
public class MDCUtil {
    private static final String TRACE_ID = "TRACE_ID";

    public static void setTraceId(String traceId) {
        set(TRACE_ID, traceId);
    }

    public static void set(String key, String value) {
        MDC.put(key, value);
    }
}
```

# 修改日志配置 (logback-spring.xml)

增加 %X{TRACE_ID}，例如：

```
<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%15.15thread] %-40.40logger Line:%-3L [%X{TRACE_ID}] - %msg%n</pattern>
```

其中 TRACE_ID 对应 MDC 工具类里的 TRACE_ID。

# 添加 AOP

```
@Slf4j
@Aspect
@Component
public class ControllerAspect {
    @Autowired
    HttpServletRequest request;

    private final String traceIdHeaderName = "traceId";

    @Pointcut("execution(public * com.karonda.controller.*.*(..))")
    public void pointcut(){}

    @Around("pointcut()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        Object result;
        try {
            this.before(pjp);
            result = pjp.proceed();
        } catch (Throwable e) {
            this.exception(pjp, e);
            throw e;
        }
        this.after(pjp, result);
        return result;
    }

    private void before(ProceedingJoinPoint pjp) {
        if(request != null && request.getHeader(traceIdHeaderName) != null) {
            MDCUtil.setTraceId(request.getHeader(traceIdHeaderName));
        } else {
            String traceId = UUID.randomUUID().toString().replace("-", "");
            MDCUtil.setTraceId(traceId);
        }
        log.info("before log");
    }

    private void after(ProceedingJoinPoint pjp, Object result) {
        log.info("after log");
    }

    private void exception(ProceedingJoinPoint pjp, Throwable e) {
        log.error("exception log", e);
    }
}
```

至此就可以在日志中看到 trace id 输出了，但是 MDC 是基于 ThreadLocal 不能跨线程传递，在异步调用的情况下 trace id 为空。

需要做如下改造：

# 修改 MDC 工具类

```
public class MDCUtil {
    private static final String TRACE_ID = "TRACE_ID";

    public static Runnable wrap(final Runnable runnable, final Map<String, String> context) {
        return () -> {
            if (context != null) {
                MDC.setContextMap(context);
            }
            try {
                runnable.run();
            } finally {
                MDC.clear();
            }
        };
    }

    public static <T> Callable<T> wrap(final Callable<T> callable, final Map<String, String> context) {
        return () -> {
            if (context != null) {
                MDC.setContextMap(context);
            }
            try {
                return callable.call();
            } finally {
                MDC.clear();
            }
        };
    }

    public static void setTraceId(String traceId) {
        set(TRACE_ID, traceId);
    }

    public static void set(String key, String value) {
        MDC.put(key, value);
    }
}
```

# 使用示例

```
@Slf4j
@Service
public class TraceService {

    private final Executor executor;

    public TraceService() {
        executor = new ThreadPoolExecutor(
                1
                , Runtime.getRuntime().availableProcessors() * 2
                , 60, TimeUnit.MINUTES
                , new LinkedBlockingDeque<>(100)
                , Executors.defaultThreadFactory()
                , new ThreadPoolExecutor.CallerRunsPolicy());
    }

    public void trace() {
        log.info("service log");

//        executor.execute(() -> log.info("thread log"));
        executor.execute(MDCUtil.wrap(()->log.info("thread log"), MDC.getCopyOfContextMap()));
    }
}
```

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-boot-trace-id)