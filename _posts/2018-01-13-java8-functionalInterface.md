---
title: Java8内置函数式接口解析与理解
layout: post
author: 岳浩
catalog: true
data:2018-01-13 13:00:00
tags: 
 - java java8 函数式编程 
---


java8增加了对函数式编程的支持，那么如果我们要自定义函数式接口，应该如何来做呢？


话不多说看代码示例

第一个示例
``` java
/**
 * Created by yuehao on 2018/1/15.
 * <p>
 * 消费型接口，就是说你传进来一个参数T，进行逻辑运算或者处理，无返回值
 */

@FunctionalInterface
public interface ConsumerTest<T> {


    //测试供给型接口
    void test(T t);
}


//调用示例 调用自定义函数式接口，进行输出
private static void TestConsumer() throws Exception {
    List<Integer> lists = Arrays.asList(1, 23, 4, 56, 78, 90, 94, 4, 43);
    ConsumerTest<Integer> consumerTest = (p) -> System.out.println(p);
    lists.stream().forEach(consumerTest::test);

    ConsumerTest<List> consumerList = (p) -> p.stream().forEach(s -> System.out.println(s));
    consumerList.test(lists);
}
```


第二个示例
``` java
/**
 * Created by yuehao on 2018/1/15.
 * <p>
 * 供给型接口，不需要参数，返回值是你需要的数据
 */
@FunctionalInterface
public interface SupplierTest<T> {
    T get();
}

//调用示例 生成随机数
private static void TestSupplier() throws Exception {
    SupplierTest<List> supplierTest = () -> {
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            list.add(new Random().nextInt(10000));
        }
        return list;
    };
    supplierTest.get().stream().forEach(s -> System.out.println(s));
}
```


第三个示例
``` java
/**
 * Created by yuehao on 2018/1/15.
 *
 * 函数型接口，传递一个参数，返回需要的数据
 */
@FunctionalInterface
public interface FunctionTest<T, R> {
    R apply(T t);
}

//调用示例 传递Integer 每个值 + 10，返回
private static void TestFunction() throws Exception {
    List<Integer> lists = Arrays.asList(1, 23, 4, 56, 78, 90, 94, 4, 43);
    FunctionTest functionTest = (p) -> (Integer) p + 10;
    lists.stream().forEach(s -> {
        System.out.println(functionTest.apply(s));
    });
}
```


第四个示例
``` java
/**
 * Created by yuehao on 2018/1/15.
 *
 * 断言型接口，传值进来，返回boolean类型结果
 */
@FunctionalInterface
public interface PredicateTest<T> {
    boolean test(T t);
}

//调用示例 判断传入的值是否>50,返回boolean结果

private static void TestPredicate() throws Exception {
    List<Integer> lists = Arrays.asList(1, 23, 4, 56, 78, 90, 94, 4, 43);
    PredicateTest<Integer> predicateTest = (s) -> s > 50;
    lists.stream().forEach(s->{
        System.out.println(predicateTest.test(s));
    });
}
```


其实上面这几个示例，都是java8自带的函数接口，我们可以根据自己的需要去定义函数式接口，函数式接口有个注解@FunctionalInterface
作用其实就是判断这个接口是否符合Java8的函数式接口规范，函数式接口只允许有一个抽象方法，然后使用的时候根据需要去编写实现即可

常用的几种类型是这几种，可以根据需求自己来定义即可


关注公众号获取更多干货文章
<img src="/img/weixin/qrcode_jishujiagou.jpg" width="100" height="100" alt="AltText" />

