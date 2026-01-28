---
title: 记一次被删库の数据恢复记录
date: 2020-05-25 11:00:00
updated: 2020-05-25 11:00:00
categories: [IT]
tags: [MySQL]
---

虽然工作挺多年了，也只是简单看了看数据恢复的方法，并未实操过，这次真摊上这事了，发现没那么简单。

# 问题描述

## 事件

生产环境某个表的数据被全部删除

## 问题

1. 数据库使用阿里云 RDS，数据隔天完整备份，没有单表备份
1. 系统运行中，不能停机维护
1. 期望只恢复被清空的表的数据，不影响其他表

# 一、寻找解决方案

## 1.1 直接从备份恢复 (不可行)

上一次完整备份时间是两天前，如果使用备份恢复那近两天的数据都要通过 binlog 慢慢恢复

存在问题：
1. 操作复杂，有许多 binlog文件
2. 会影响其他表的数据
3. 系统运行中，数据在一直增加。。。

## 1.2 直接使用 binlog 恢复 (不可行)

与方案一存在相同问题(或者我对 binlog 理解不透彻，没有找到正确的恢复方式)

## 1.3 工具恢复 (不可行)

简单尝试使用 [canal](https://github.com/alibaba/canal)，但 canal 只能基于业务 trigger 获取增量变更，无法解析历史的变更(或许是有该功能，只是我没 get 到)；也尝试了使用其他几款恢复工具，均存在这样或那样的问题

## 1.4 手动解析 binlog (最终方案)

寻找方案过程中，发现一个解析 binlog 的开源库：[mysql-binlog-connector-java](https://github.com/shyiko/mysql-binlog-connector-java)，于是决定自己手动写代码将 binlog 中的删除事件转化成 insert 语句，验证通过，最终使用此方案

# 二、方案具体实现

## 2.1 添加依赖

```
<dependency>
	<groupId>com.github.shyiko</groupId>
	<artifactId>mysql-binlog-connector-java</artifactId>
	<version>0.20.1</version>
</dependency>
```

## 2.2 具体代码

```
@Component
public class RestoreRunner implements CommandLineRunner {

    private final String filePath = "D:\\mysql-bin.******"; // binlog 路径
    private final String beginTime = "2020-05-25 08:10:23"; // 删除数据开始时间
    private final String endTime = "2020-05-25 08:10:26"; // 删除数据结束时间
    private final String databaseName = "test_db"; // 数据库名
    private final String tableName = "test_talbe"; // 数据表名

    private static Logger logger = LoggerFactory.getLogger("");

    private final SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    private long tableId;
    private int count;


    @Override
    public void run(String... args) throws Exception {

        long beginTimestamp = simpleDateFormat.parse(beginTime).getTime();
        long endTimestamp = simpleDateFormat.parse(endTime).getTime();

        File binlogFile = new File(filePath);
        EventDeserializer eventDeserializer = new EventDeserializer();
//        eventDeserializer.setCompatibilityMode(
//                EventDeserializer.CompatibilityMode.DATE_AND_TIME_AS_LONG,
//                EventDeserializer.CompatibilityMode.CHAR_AND_BINARY_AS_BYTE_ARRAY
//        );
        
        BinaryLogFileReader reader = new BinaryLogFileReader(binlogFile, eventDeserializer);
        try {
            for (Event event; (event = reader.readEvent()) != null; ) {

                switch (event.getHeader().getEventType()) {
                    case TABLE_MAP:
                        TableMapEventData eventData = event.getData();
                        if(databaseName.equals(eventData.getDatabase()) && tableName.equals(eventData.getTable())) {
                            tableId = eventData.getTableId();
                        }
                        break;
                    default:
                        break;
                }

                if(event.getHeader().getTimestamp() < beginTimestamp) {
                    continue;
                } else if(event.getHeader().getTimestamp() > endTimestamp) {
                    break;
                }

                switch (event.getHeader().getEventType()) {
                    case DELETE_ROWS:
                        this.handleDeleteRowsEvent(event.getData());
                        break;
                    case UPDATE_ROWS:
                        // todo
                        break;
                    case WRITE_ROWS:
                        // todo
                        break;
                    default:
                        break;
                }
            }

            logger.warn("/* {} */", count);
        } finally {
            reader.close();
        }
    }

    private void handleDeleteRowsEvent(DeleteRowsEventData deleteData) {
        if(deleteData.getTableId() != tableId) {
            return;
        }

        Iterator rows = deleteData.getRows().iterator();

        while(rows.hasNext()) {
            Object[] row = (Object[])rows.next();

            count++;
            StringBuilder sb = new StringBuilder(" insert into " + tableName + " values (");

            for(Object item : row) {
                if(item instanceof String) {
                    sb.append("'" + item + "'");
                } else if(item instanceof Date) {
                    String dateStr = this.adjustDateAndToString((Date) item);
                    sb.append("'" + dateStr + "'");
                }else {
                    sb.append(item);
                }
                sb.append(",");
            }
            sb.replace(sb.length() - 1, sb.length(), ");");

            logger.warn(sb.toString());
        }
    }

    private String adjustDateAndToString(Date date) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        calendar.add(Calendar.HOUR, -8); // 8 小时时差
        Date adjustDate = calendar.getTime();

        return simpleDateFormat.format(adjustDate);
    }
}
```

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/mysql-data-restore)


# 结束语

**一定要开启 binlog ! 一定要开启 binlog ! 一定要开启 binlog !**

目前只解析了数据被删除的情况，以后有时间可以将此代码功能丰富，做一个数据恢复小工具