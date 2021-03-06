---
layout: post
title: 敏感数据脱敏
image:
  feature: abstract-4.jpg
tags: [脱敏, java]
comments: false
share: true
---

现在越来越多的公司，特别是第三方支付，金融等 对用户敏感信息的输出是越来越严格。常见的需对日志输出客户敏感数据进行脱敏，现在一般解决的方式，无非是

1. 重写toString
2. 加入脱敏插件
3. 不输出

**以下是自己利用fastjson + commons-lang3实现的一种 1. 重写toString**

## 工具类
[MarkToString.java](https://github.com/enaihei/enaihei/blob/master/src/main/java/com/kit/enaihei/commons/MarkToString.java)

## 用例
详细测试用例：[MarkToStringTest](https://github.com/enaihei/enaihei/blob/master/src/test/java/com/kit/enaihei/MarkToStringTest.java)
```java
public class MarkToStringTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Test
    public void testObject() {
        A<C> a = new A();
        a.setA("aaaaaaaaaaa");
        a.setB("bbbbbbbbbbb");
        C c = new C();
        c.setD("ddddddddddddd");
        c.setE("eeeeeeeeeeeee");
        a.setC(c);
        logger.info("=======================>{}", a);
    }

    @Test
    public void testJson() {
        String json = "{\"idCard\":\"522227199001291889\",\"name\":\"张三\",\"mobile\":\"13883482726\"," +
        "\"contactList\":[{\"mobile\":\"13883482726\",\"name\":\"李四\"}]}";
        logger.info("======================>{}", MarkToString.markToJSON(json));
    }

    class A<T> implements Serializable {
        private static final long serialVersionUID = 1612279976323528282L;

        @MarkToString.Mark
        private String a;

        @MarkToString.Hide
        private String b;

        @MarkToString.Mark
        private T c;

        省略set,get...

        @Override
        public String toString() {
            return MarkToString.toString(this);
        }
    }
```

## 数据输出
```java
16:58:00.937 [main] INFO com.kit.enaihei.test.MarkToStringTest - =======================>MarkToStringTest.A[a=aaa****aaaa,c=MarkToStringTest.C[d=ddddddddddddd,e=eee******eeee]]
16:56:03.030 [main] INFO com.kit.enaihei.test.MarkToStringTest - ======================>MarkToString.MarkData[map={idCard=522227********1889,contactList=[{"name":*******************3482726"}],name=*三,mobile=138****2726}]
```


**以下是自己利用fastjson实现的另一种 1. 重写toString**

## 工具类
[ToString.java](https://github.com/enaihei/enaihei/blob/master/src/main/java/com/kit/enaihei/commons/ToString.java)

## 用例
详细测试用例：[ToStringTest](https://github.com/enaihei/enaihei/blob/master/src/test/java/com/kit/enaihei/ToStringTest.java)
```java
@Test
public void testObject() {
    Entity entity = new Entity();
    entity.setId(1);
    entity.setName("张三");
    entity.setAverage(4f);
    Map<String, Object> map = Maps.newHashMap();
    map.put("name", "张三");
    map.put("mobile", "15112355647");
    entity.setMap(map);
    Entity _entity = new Entity();
    _entity.setId(2);
    _entity.setName("李四");
    _entity.setAverage(5f);
    Map<String, Object> _map = Maps.newHashMap();
    _map.put("name", "李四");
    _map.put("mobile", "15234568795");
    _entity.setMap(_map);
    entity.setEntities(Lists.newArrayList(_entity));

    logger.info("==================>{}", entity);
    logger.info("==================>{}", entity.getMap());

}
...
```

## 数据输出
```java
14:33:49.284 [main] INFO com.kit.enaihei.ToStringTest - ==================>{name:"*三",id:1,map:{name:"*三",mobile:"151****5647"},average:4,entities:[{name:"*四",id:2,map:{name:"*四",mobile:"152****8795"},average:5,entities:[]}]}
14:33:49.390 [main] INFO com.kit.enaihei.ToStringTest - ==================>{name=张三, mobile=15112355647}
```

