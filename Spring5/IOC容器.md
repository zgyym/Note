# 概念和原理

## 什么是IOC

1. 控制反转，把创建对象和对象之间的调用过程，交给Spring进行管理
2. 使用IOC的目的，是为了解耦合

## IOC底层原理

***xml解析，工厂模式，反射***

### 原理图解

1. 创建对象的**原始方式**：![](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220429083728884.png)

2. 使用**工厂模式**创建对象![image-20220429084015052](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220429084015052.png)

3. **IOC过程**创建对象：![图2](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/%E5%9B%BE2.png)

​				再配合工厂模式进行对象创建。

### IOC（接口）

1. IOC思想是基于IOC容器实现的，IOC容器底层就是对象工厂
2. Spring提供了IOC的两种实现方式（两个接口）
   1. BeanFactory：IOC容器的基本实现，是Spring内部使用的接口，一般不提供给开发人员进行使用：
      - **BeanFactory在加载xml配置文件的时候不会创建对象，而是在获取对象（使用）的时候才去创建对象。**
   2. ApplicationContext：BeanFactory接口的子接口，提供了更多更强大的功能，一般由开发人员进行使用
      - **ApplicationContext在加载xml配置文件的时候就会把在配置文件中的对象进行创建。**
   3. ApplicationContext的实现类：![image-20220429104738314](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220429104738314.png)

## IOC操作Bean管理

### 概念

1. 什么是Spring管理？
   - Spring管理指的是两个操作
   - Spring创建对象
   - Spring注入属性
2. Spring管理操作有两种方式
   - 基于xml配置文件方式实现
   - 基于注解方式实现

### 基于xml配置文件方式

![image-20220429195752818](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220429195752818.png)

1. 基于xml方式创建对象
   1. 在Spring的xml配置文件中，使用bean标签，标签里添加对应的属性，就可以实现对象的创建
   2. 在bean标签中有很多属性：
      - **id**属性：这个属性不是创建的对象的名字，而是用来唯一标识对象的
      - **class**属性：创建对象所需类的全类名
   3. 创建对象的时候，默认执行无参数的构造方法完成对象的创建
   
2. 基于xml方式注入属性

   **DI**：依赖注入，就是注入属性，在创建对象的基础上注入属性。

3. 第一种注入方式：使用 set 方法进行注入

   1. 创建类，定义属性和对应的 set 方法

      ```java
      /**
      * 演示使用 set 方法进行注入属性
      */
      public class Book {
           //创建属性
           private String bname;
           private String bauthor;
           //创建属性对应的 set 方法
           public void setBname(String bname) {
           	this.bname = bname;
           }
           public void setBauthor(String bauthor) {
           	this.bauthor = bauthor;
           }
      }
      ```

   2. 在 Spring 配置文件配置对象创建，配置属性注入

      ```xml
      <!--2 set 方法注入属性-->
      <bean id="book" class="com.pojo.Book">
           <!--使用 property 完成属性注入
           name：类里面属性名称
           value：向属性注入的值
           -->
           <property name="bname" value="易筋经"></property>
           <property name="bauthor" value="达摩老祖"></property>
      </bean>
      
      ```

4. 第二种注入方式：使用有参数构造进行注入

   1. 创建类，定义属性，创建属性对应有参数构造方法

      ```java
      
      /**
       * 使用有参数构造注入
       */
      public class Orders {
          //属性
          private String oname="";
          private String address;
          //有参数构造
          public Orders(String oname,String address) {
              this.oname = oname;
              this.address = address;
          }
      }
      ```

   2. 在 Spring 配置文件中进行配置

      ```xml
      <bean id="orders" class="com.pojo.Orders">
                  <constructor-arg name="oname" value="电脑"></constructor-arg>
                  <constructor-arg name="address" value="China"></constructor-arg>
      </bean>
      ```

5. xml注入其他类型的属性

   

### 基于注解方式



