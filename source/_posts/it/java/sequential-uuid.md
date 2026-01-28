---
title: Java 生成有序 UUID
date: 2019-06-27 10:30:00
updated: 2019-06-27 10:30:00
categories: [IT]
tags: [Java, UUID, Hibernate]
---

UUID.randomUUID() 生成的 UUID 是无序的，如果作为数据主键，不利于索引

Hibernate 的 UUIDHexGenerator.generate() 方法可以生成有序的 UUID, 本文参考其实现：

```
public class SequentialUuidHexGenerator extends AbstractUUIDGenerator{

    private static final String sep = "-";

    public static String generate() {
        return
                format( getJVM() ) + sep
                + format( getHiTime() ) + sep
                + format( getLoTime() ) + sep
                + format( getIP() ) + sep
                + format( getCount() );
    }

    protected static String format(int intValue) {
        String formatted = Integer.toHexString( intValue );
        StringBuilder buf = new StringBuilder( "00000000" );
        buf.replace( 8 - formatted.length(), 8, formatted );
        return buf.toString();
    }

    protected  static String format(short shortValue) {
        String formatted = Integer.toHexString( shortValue );
        StringBuilder buf = new StringBuilder( "0000" );
        buf.replace( 4 - formatted.length(), 4, formatted );
        return buf.toString();
    }
}
```

> UUIDHexGenerator.generate() 将 IP 放在首位，考虑到在不同的机器生成，本文将时间戳放在了首位

AbstractUUIDGenerator 代码：

```
public abstract class AbstractUUIDGenerator {

    private static final int IP;
    static {
        int ipadd;
        try {
            ipadd = BytesHelper.toInt( InetAddress.getLocalHost().getAddress() );
        }
        catch (Exception e) {
            ipadd = 0;
        }
        IP = ipadd;
    }

    private static short counter = (short) 0;
    private static final int JVM = (int) ( System.currentTimeMillis() >>> 8 );

    public  AbstractUUIDGenerator() {
    }

    protected static int getJVM() {
        return JVM;
    }

    protected static short getCount() {
        synchronized(AbstractUUIDGenerator.class) {
            if ( counter < 0 ) {
                counter=0;
            }
            return counter++;
        }
    }

    protected static int getIP() {
        return IP;
    }

    protected static short getHiTime() {
        return (short) ( System.currentTimeMillis() >>> 32 );
    }

    protected static int getLoTime() {
        return (int) System.currentTimeMillis();
    }
}
```

BytesHelper 代码：

```
public final class BytesHelper {

    private BytesHelper() {
    }

    public static int toInt(byte[] bytes) {
        int result = 0;
        for ( int i = 0; i < 4; i++ ) {
            result = ( result << 8 ) - Byte.MIN_VALUE + (int) bytes[i];
        }
        return result;
    }
}
```

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/sequential-uuid)

