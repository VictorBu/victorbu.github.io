---
title: 基于 MongoDB 动态字段设计的探索
date: 2019-08-01 09:30:00
updated: 2019-08-04 09:30:00
categories: [IT]
tags: [Spring Boot, MongoDB]
---

# 一、业务需求

假设某学校课程系统，不同专业课程不同 (可以动态增删)，但是需要根据专业不同显示该专业学生的各科课程的成绩，如下：

|  专业   | 姓名  | 高等数学 | 数据结构 |
|  ----  | ----  | ----  | ----  |
| 计算机  | 张三 | 90 | 85 |
| 计算机  | 李四 | 78 | 87 |

|  专业   | 姓名  | 高等数学 |
|  ----  | ----  | ----  |
| 数学  | 王五 | 86 |
| 数学  | 赵六 | 95 |

# 二、设计思路

开始的思路是根据配置的课程动态生成文档字段，使用非映射方式直接操作 MongoCollection, 有以下问题：

1. 存取数据日期序列化问题 (亦可能是本人没有找到正确的处理方式)
1. 返回结果集不能转换成实体对象，不方便做二次处理

所以最终使用内嵌数组的方式

# 三、代码示例

## 3.1 实体类

```
public class Student {

    private String name;
    private String major;
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime createTime;
    @JsonIgnore
    private List<Course> courseList;

    /* getter setter*/

}

public class Course {

    private String name;
    private Float score;

    /* getter setter*/
}
```

## 3.2 添加测试专业及课程

```
public class MajorConfig {

    private static MajorConfig computer;
    private static MajorConfig math;

    static {
        computer = new MajorConfig();
        computer.setName("计算机");

        List<String> courseList = new ArrayList<>();
        courseList.add("高等数学");
        courseList.add("数据结构");
        computer.setCourseList(courseList);

        math = new MajorConfig();
        math.setName("数学");
        courseList = new ArrayList<>();
        courseList.add("高等数学");
        math.setCourseList(courseList);


    }

    private String name;
    private List<String> courseList;

    /* getter setter*/
}

```

## 3.3 添加测试数据到 MongoDB

初始化数据 (createTime 在 MongoDB 存储为 ISODate)：

```
[
  {
    "name": "张三",
    "major": "计算机",
    "createTime": "2018-01-20 08:00:00",
    "courseList": [
      {
        "name": "高等数学",
        "score": 20.283026
      },
      {
        "name": "数据结构",
        "score": 30.612194
      }
    ]
  },
  {
    "name": "王五",
    "major": "数学",
    "createTime": "2018-01-20 08:00:00",
    "courseList": [
      {
        "name": "高等数学",
        "score": 91.78229
      }
    ]
  },
  {
    "name": "李四",
    "major": "计算机",
    "createTime": "2019-10-01 07:10:50",
    "courseList": [
      {
        "name": "高等数学",
        "score": 60.488556
      },
      {
        "name": "数据结构",
        "score": 80.66098
      }
    ]
  },
  {
    "name": "赵六",
    "major": "数学",
    "createTime": "2019-10-01 07:10:50",
    "courseList": [
      {
        "name": "高等数学",
        "score": 29.595625
      }
    ]
  }
]
```

## 3.4 根据专业获取学生课程分数列表

首先查询到对应的数据，然后根据配置的课程动态添加字段：

```
  public Object list(String major){

      MajorConfig majorConfig = getMajorConfig(major);

      Query query = new Query(Criteria.where("major").is(major));
      List<Student> studentList = mongoTemplate.find(query, Student.class);

      List<Object> result = new ArrayList<>();

      for(Student student:studentList){

          Map<String,Object> properties = Maps.newHashMap();
          for(String name : majorConfig.getCourseList()){
              properties.put(name,null);
              for(Course course : student.getCourseList()){
                  if(name.equals(course.getName())){
                      properties.put(name,course.getScore());
                      break;
                  }
              }
          }

          result.add(ReflectUtil.getTarget(student,properties));
      }

      return result;
  }
```

各专业学生各科分数数据：

```
[
  {
    "name": "王五",
    "major": "数学",
    "createTime": "2018-01-20 08:00:00",
    "高等数学": 91.78229
  },
  {
    "name": "赵六",
    "major": "数学",
    "createTime": "2019-10-01 07:10:50",
    "高等数学": 29.595625
  }
]
```

```
[
  {
    "name": "张三",
    "major": "计算机",
    "createTime": "2018-01-20 08:00:00",
    "高等数学": 20.283026,
    "数据结构": 30.612194
  },
  {
    "name": "李四",
    "major": "计算机",
    "createTime": "2019-10-01 07:10:50",
    "高等数学": 60.488556,
    "数据结构": 80.66098
  }
]
```

ReflectUtil 代码：

```
public class ReflectUtil {

    public static Object getTarget(Object dest, Map<String, Object> addProperties) {
        PropertyUtilsBean propertyUtilsBean = new PropertyUtilsBean();
        PropertyDescriptor[] descriptors = propertyUtilsBean.getPropertyDescriptors(dest);

        Map<String, Class> propertyMap = Maps.newHashMap();

        for (PropertyDescriptor d : descriptors) {
            if (!"class".equalsIgnoreCase(d.getName())) {
                propertyMap.put(d.getName(), d.getPropertyType());
            }

        }
        // add extra properties
        addProperties.forEach((k, v) -> {
            if(v != null){
                propertyMap.put(k, v.getClass());
            }else{
                propertyMap.put(k, Object.class);
            }
        });
        // new dynamic bean
        DynamicBean dynamicBean = new DynamicBean(dest.getClass(), propertyMap);
        // add old value
        propertyMap.forEach((k, v) -> {
            try {
                // filter extra properties
                if (!addProperties.containsKey(k)) {
                    dynamicBean.setValue(k, propertyUtilsBean.getNestedProperty(dest, k));
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
        // add extra value
        addProperties.forEach((k, v) -> {
            try {
                dynamicBean.setValue(k, v);
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
        Object target = dynamicBean.getTarget();
        return target;
    }
}
```

DynamicBean 代码：

```
public class DynamicBean {

    private Object target;

    private BeanMap beanMap;

    public DynamicBean(Class superclass, Map<String, Class> propertyMap) {
        this.target = generateBean(superclass, propertyMap);
        this.beanMap = BeanMap.create(this.target);
    }

    public void setValue(String property, Object value) {
        beanMap.put(property, value);
    }

    public Object getValue(String property) {
        return beanMap.get(property);
    }

    public Object getTarget() {
        return this.target;
    }

    private Object generateBean(Class superclass, Map<String, Class> propertyMap) {
        BeanGenerator generator =new BeanGenerator();
        if(null != superclass) {
            generator.setSuperclass(superclass);
        }
        BeanGenerator.addProperties(generator, propertyMap);
        return generator.create();
    }
}
```

## 3.5 添加课程并修改数据库

```
public Object add(String major, String name, Boolean updateDatabase){

        MajorConfig majorConfig = getMajorConfig(major);

        List<String> courseList = majorConfig.getCourseList();

        if(!courseList.contains(name))
        {
            courseList.add(name);
            majorConfig.setCourseList(courseList);
        }

        if(updateDatabase){

            Random random = new Random();

            Course course = new Course();
            course.setName(name);
            course.setScore(random.nextFloat() * 100);

            Update update = new Update();
            update.addToSet("courseList", course);
            Query query = Query.query(Criteria.where("major").is(majorConfig.getName()));
            mongoTemplate.updateMulti(query, update, Student.class);

        }

        return majorConfig;
    }
```

为数学专业添加计算机基础：

```
[
  {
    "name": "王五",
    "major": "数学",
    "createTime": "2018-01-20 08:00:00",
    "计算机基础": 9.042096,
    "高等数学": 91.78229
  },
  {
    "name": "赵六",
    "major": "数学",
    "createTime": "2019-10-01 07:10:50",
    "计算机基础": 9.042096,
    "高等数学": 29.595625
  }
]
```

## 3.6 删除课程并修改数据库

```
public Object del(String major, String name, Boolean updateDatabase){

        MajorConfig majorConfig = getMajorConfig(major);

        List<String> courseList = majorConfig.getCourseList();

        if(courseList.contains(name)){
            courseList.remove(name);
            majorConfig.setCourseList(courseList);
        }


        if(updateDatabase){
            Update update = new Update();
            update.pull("courseList", new BasicDBObject("name", name));
            Query query = Query.query(Criteria.where("major").is(majorConfig.getName()));
            mongoTemplate.updateMulti(query,update,Student.class);
        }

        return majorConfig;
    }
```

把高等数学从计算机专业删除：

```
[
  {
    "name": "张三",
    "major": "计算机",
    "createTime": "2018-01-20 08:00:00",
    "数据结构": 30.612194
  },
  {
    "name": "李四",
    "major": "计算机",
    "createTime": "2019-10-01 07:10:50",
    "数据结构": 80.66098
  }
]
```

## 3.7 修改某学生的某课程分数

```
public Object update(String name, String courseName, Float score){

    Query query = Query.query(Criteria.where("name").is(name).and("courseList.name").is(courseName));

    Update update = new Update();
    update.set("courseList.$.score", score);

    mongoTemplate.updateFirst(query, update, Student.class);

    return null;
}
```

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-boot-2-mongodb)


