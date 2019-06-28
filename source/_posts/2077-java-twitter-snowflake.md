---
title: 雪花算法 Java 版
date: 2019-06-28 10:30:00
updated: 2019-06-28 10:30:00
categories: [IT]
tags: [Java, snowflake]
---

雪花算法根据时间戳生成有序的 64 bit 的 Long 类型的唯一 ID

各 bit 含义：

+ 1 bit: 符号位，0 是正数 1 是负数， ID 为正数，所以恒取 0
+ 41 bit: 时间差，我们可以选择一个参考点，用它来计算与当前时间的时间差 (毫秒数)，41 bit 存储时间差，足够使用 69 年
+ 10 bit: 机器码，能编码 1024 台机器；可以手动指定含义，比如前5 bit 作为机器编号、后 5 bit 作为进程编号
+ 12 bit: 序列号，同一机器同一毫秒内产生不同的序列号，12 bit 可以支持 4096 个序列号

优点：

+ 灵活配置：机器码可以根据需求灵活配置含义
+ 无需持久化：如果序号自增往往需要持久化，本算法不需要持久化
+ ID 有含义/可逆性：ID 可以反解出来，对 ID 进行统计分析，可以很简单的分析出整个系统的繁忙曲线，还可以定位到每个机器，在某段时间承担了多少工作，分析出负载均衡情况
+ 高性能：生成速度很快


```
public class Snowflake {

    /**
     * 每一部分所占位数
     */
    private final long unusedBits = 1L;
    private final long timestampBits = 41L;
    private final long datacenterIdBits = 5L;
    private final long workerIdBits = 5L;
    private final long sequenceBits = 12L;

    /**
     * 向左的位移
     */
    private final long timestampShift = sequenceBits + datacenterIdBits + workerIdBits;
    private final long datacenterIdShift = sequenceBits + workerIdBits;
    private final long workerIdShift = sequenceBits;

    /**
     * 起始时间戳，初始化后不可修改
     */
    private final long epoch = 1451606400000L; // 2016-01-01

    /**
     * 数据中心编码，初始化后不可修改
     * 最大值: 2^5-1 取值范围: [0,31]
     */
    private final long datacenterId;

    /**
     * 机器或进程编码，初始化后不可修改
     * 最大值: 2^5-1 取值范围: [0,31]
     */
    private final long workerId;

    /**
     * 序列号
     * 最大值: 2^12-1 取值范围: [0,4095]
     */
    private long sequence = 0L;

    /** 上次执行生成 ID 方法的时间戳 */
    private long lastTimestamp = -1L;

    /*
     * 每一部分最大值
     */
    private final long maxDatacenterId = -1L ^ (-1L << datacenterIdBits); // 2^5-1
    private final long maxWorkerId = -1L ^ (-1L << workerIdBits); // 2^5-1
    private final long maxSequence = -1L ^ (-1L << sequenceBits); // 2^12-1

    /**
     * 生成序列号
     */
    public synchronized long nextId() {
        long currTimestamp = timestampGen();

        if (currTimestamp < lastTimestamp) {
            throw new IllegalStateException(
                    String.format("Clock moved backwards. Refusing to generate id for %d milliseconds",
                            lastTimestamp - currTimestamp));
        }

        if (currTimestamp == lastTimestamp) {
            sequence = (sequence + 1) & maxSequence;
            if (sequence == 0) { // overflow: greater than max sequence
                currTimestamp = waitNextMillis(currTimestamp);
            }

        } else { // reset to 0 for next period/millisecond
            sequence = 0L;
        }

        // track and memo the time stamp last snowflake ID generated
        lastTimestamp = currTimestamp;

        return ((currTimestamp - epoch) << timestampShift) | //
                (datacenterId << datacenterIdShift) | //
                (workerId << workerIdShift) | // new line for nice looking
                sequence;
    }

    public Snowflake(long datacenterId, long workerId) {
        if (datacenterId > maxDatacenterId || datacenterId < 0) {
            throw new IllegalArgumentException(
                    String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
        }
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(
                    String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
        }

        this.datacenterId = datacenterId;
        this.workerId = workerId;
    }

    /**
     * 追踪调用 waitNextMillis 方法的次数
     */
    private final AtomicLong waitCount = new AtomicLong(0);

    public long getWaitCount() {
        return waitCount.get();
    }


    /**
     * 循环阻塞直到下一秒
     */
    protected long waitNextMillis(long currTimestamp) {
        waitCount.incrementAndGet();
        while (currTimestamp <= lastTimestamp) {
            currTimestamp = timestampGen();
        }
        return currTimestamp;
    }

    /**
     * 获取当前时间戳
     */
    public long timestampGen() {
        return System.currentTimeMillis();
    }
}
```

参考：[snowflake ID 生成算法](https://github.com/downgoon/snowflake/blob/master/docs/SnowflakeTutorial_zh_CN.md)


完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/twitter-snowflake)

