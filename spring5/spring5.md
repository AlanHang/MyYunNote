[TOC]

# Spring 框架概述

1. Spring框架是轻量级的开源的JavaEE框架。

2. Spring可以解决企业应用开发的复杂性。

3. Spring有两个核心部分：IOC和Aop

   - IOC : 控制反转，把创建对象过程交给Spring进行管理。
   - Aop : 面向切面，不修改源码进行功能增强。

4. Spring框架的特点：

   - 方便解耦，简化开发。
   - Aop编程支持。
   - 方便程序测试。
   - 方便集成其他框架。
   - 方便事务操作。
   - 方便API开发难度。


# IOC 容器

## IOC 概念和原理

1. 什么是IOC

   - 控制反转，把对象创建和对象之间的调用过程，交给Spring进行管理。
   - 使用IOC目的：为了降低耦合度。

2. IOC底层原理

   - xml解析、工厂模式、反射

3. IOC过程 进一步降低耦合度

   ```java
   //第一步 xml配置文件，配置创建的对象
   <bean id = "dao" class = "com.bonc.UserDao"></bean>
   //第二步，有service类和到类，创建工厂类
   class UserFactory{
       public static UserDao getDao(){
           String classValue = class属性值;//xml解析
           Class clazz = Class.forName(classValue);//2.通过反射创建对象
           return (UserDao)clazz.newInstance();
       }
   }
   ```

4. bean xml文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
   </beans>
   ```

   
## IOC 接口

   - IOC思想基于IOC容器完成，IOC容器底层就是对象工厂。

   - Spring提供IOC容器实现的两种方式：（两个接口）

     - BeanFactory:IOC容器基本实现，是Spring内部的使用的接口，不提供给开发人员使用。、

       > 加载配置文件时不会创建对象，在获取(使用)对象时才创建对象。

     - ApplicationContext：BeanFactory的子接口，提供了更多更强大的功能，一般由开发人员使用。

       > 加载配置文件时候就会把配置文件对象进行创建。

   - ApplicationContext的实现类

     ![image-20210908150123134](spring5.assets/image-20210908150123134.png)

     - FileSystemXmlApplicationContext 绝对路径
     - ClassPathXmlApplicationContext 相对路径

## IOC操作（Bean管理）

1. 什么是Bean管理

   Bean管理指的是两个操作：

   1. Spring创建对象
   2. Spring注入属性

2. Bean管理操作有两种操作方式

   1. 基于xml配置文件方式实现
   2. 基于注解方式实现

### 基于xml方式管理

1. 基于xml方式创建对象

   ```xml
   <bean id="user" class="com.bonc.User"></bean>
   ```

   1. 在spring配置文件中，使用bean标签，在标签里面添加对于属性，就可以实现对象创建。
   2. 在bean标签有很多属性，介绍常用的属性。
      - id属性：唯一标识
      - class属性：类全路径（包类路径）
      - name属性：与id属性相同，其中可以加一些特殊符号
   3. 创建对象时，默认也是执行无参数构造方法完成对象创建。

2. 基于xml方式注入属性

   1. DI:依赖注入，是IOC中的一种具体实现

      - 使用set方法注入

        ```java
        //1.创建属性并创建构造方法
        public class Book{
            private String bname;
            private String bauthor;
            public void setBname(String bname){
                this.bname = bname;
            }
                public void setBauthor(String bauthor){
                this.bauthor = bauthor;
            }
        }
        ```

        

        ```xml
        <!--2.set方法注入属性-->
        <bean id="book" class="com.bonc.Book">
            <property name="bname" value="天龙八部"></property>
                <property name="bauthor" value="金庸"></property>
        </bean>

      - 使用构造器方法注入

        ```java
        //1.创建属性和构造方法
        public class Order {
            private String oname;
            private String address;
        
            public Order(String oname, String address) {
                this.oname = oname;
                this.address = address;
            }
        }
        ```

        ```xml
            <!-- 2.构造器注入属性   -->
            <bean id="order" class="com.bonc.Order">
                <constructor-arg name="oname" value="abc"></constructor-arg>
                <constructor-arg name="address" value="China"></constructor-arg>
            </bean>
        ```

        ```java
        //测试
        @Test
            public void test1(){
                ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
                Order order = context.getBean("order", Order.class);
                System.out.println(order);
            }
        ```

        

