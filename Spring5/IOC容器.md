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

1. **基于xml方式创建对象**
   
   1. 在Spring的xml配置文件中，使用bean标签，标签里添加对应的属性，就可以实现对象的创建
   2. 在bean标签中有很多属性：
      - **id**属性：这个属性不是创建的对象的名字，而是用来唯一标识对象的
      - **class**属性：创建对象所需类的全类名
   3. 创建对象的时候，默认执行无参数的构造方法完成对象的创建
   
2. **基于xml方式注入属性**

   **DI**：依赖注入，就是注入属性，在创建对象的基础上注入属性。

3. **第一种注入方式：使用 set 方法进行注入**

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

4. **第二种注入方式：使用有参数构造进行注入**

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

5. **xml注入其他类型的属性（对象）**

   1. 字面量
   
      1. null值
   
         ```xml
         <property name="address">
         	<null/>
         </property>
         ```
   
      2. 属性包含特殊符号
   
         ```xml
         <!--属性值包含特殊符号
          1 把<>进行转义 &lt; &gt;
          2 把带特殊符号内容写到 CDATA
         -->
         <property name="address">
             <value><![CDATA[<<南京>>]]></value>
         </property>
         ```
   
   2. 注入**外部bean**
   
      <u>**通过set注入，所以注入时要在类中给出属性的set方法，web中不需要是没有引入框架，可以直接赋值。**</u>
   
      1. 创建两个类 service 类和 dao 类，service里给出set()
   
         ```java
         public class UserDAOImpl implements UserDAO {
             @Override
             public void update() {
                 System.out.println("dao update....");
             }
         }
         ```
   
         
   
         ```java
         public class UserServiceImpl implements UserService {
         
             private UserDAO userDAO;
         
             @Override
             public void add() {
                 System.out.println("service add ......");
             }
         
             public void setUserDAO(UserDAO userDAO) {
                 this.userDAO = userDAO;
             }
         }
         
         ```
   
      2. 在 Spring 配置文件中进行配置       
   
         ```xml
          <bean id="userDAO" class="com.dao.impl.UserDAOImpl"></bean>
         
         <!--对象的赋值要用ref-->
         <bean id="userService"class="com.service.impl.UserServiceImpl">
                <!-- name属性：类里面属性名称
                 ref：创建 userDao 对象 bean 标签 id 值-->
                 <property name="userDAO" ref="userDAO"></property>
         </bean>
         ```
   
   3. 注入**内部bean**
   
      1. 一对多关系：部门和员工
   
      2. 创建实体类，在实体类里表示一对多的关系
   
         ```java
         
         public class Dept {
             private String name;
         
             public void setName(String name) {
                 this.name = name;
             }
         
             @Override
             public String toString() {
                 return "Dept{" +
                         "name='" + name + '\'' +
                         '}';
             }
         }
         
         ```
   
         ```java
         public class Emp {
             private Integer id;
             private String name;
             private String gender;
         
             private Dept dept;
         
         
             public void setId(Integer id) {
                 this.id = id;
             }
         
             public void setName(String name) {
                 this.name = name;
             }
         
             public void setGender(String gender) {
                 this.gender = gender;
             }
         
             public void setDept(Dept dept) {
                 this.dept = dept;
             }
         
             @Override
             public String toString() {
                 return "Emp{" +
                         "id=" + id +
                         ", name='" + name + '\'' +
                         ", gender='" + gender + '\'' +
                         ", dept=" + dept +
                         '}';
             }
         }
         
         ```
   
      3. 在 Spring 配置文件中进行配置
   
         ```xml
         <bean id="emp" class="com.bean.Emp">
             <property name="id" value="1"></property>
             <property name="name" value="言魔"></property>
             <property name="gender" value="男"></property>
             <property name="dept">
                 <bean id="dept" class="com.bean.Dept">
                     <property name="name" value="安保部门"></property>
                 </bean>
             </property>
         </bean>
         
         ```
   
   4. 级联赋值（了解）
   
      1. 第一种写法
   
         ```xml
         !--级联赋值-->
         <bean id="emp" class="com.atguigu.spring5.bean.Emp">
              <!--设置两个普通属性-->
              <property name="ename" value="lucy"></property>
              <property name="gender" value="女"></property>
              <!--级联赋值-->
              <property name="dept" ref="dept"></property>
         </bean>
         <bean id="dept" class="com.atguigu.spring5.bean.Dept">
          	<property name="dname" value="财务部"></property>
         </bean>
         
         ```
   
      2. 第二种写法
   
         该 写法中需要提供的dept的get()方法，因为需要向emp的dept对象中设置值
   
         ![image-20220430183002215](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220430183002215.png)
         
         ```xml
         <!--级联赋值-->
         <bean id="emp" class="com.atguigu.spring5.bean.Emp">
              <!--设置两个普通属性-->
              <property name="ename" value="lucy"></property>
              <property name="gender" value="女"></property>
              <!--级联赋值-->
              <property name="dept" ref="dept"></property>
              <property name="dept.dname" value="技术部"></property>
         </bean>
         <bean id="dept" class="com.atguigu.spring5.bean.Dept">
          	<property name="dname" value="财务部"></property>
         </bean>
         <!--技术部将财务部覆盖-->
         ```
   
6. **xml注入集合类型的属性**

   1. 注入数组类型的属性

      ```xml
      <property name="courses">
          <array>
              <value>java</value>
              <value>mysql</value>
              <value>spring</value>
          </array>
      </property>
      ```

      

   2. 注入List集合类型的属性

      ```xml
      <property name="list">
          <list>
              <value>言魔</value>
              <value>yanmo</value>
          </list>
      </property>
      ```

      

   3. 注入Map集合类型的属性

      ```xml
      <property name="map">
          <map>
              <entry key="JAVA" value="java"></entry>
              <entry key="PHP" value="php"></entry>
          </map>
      </property>
      ```

      

   4. 注入Set集合类型的属性

      ```xml
      <property name="set">
          <set>
              <value>Mysql</value>
              <value>Redis</value>
          </set>
      </property>
      ```

   5. 在集合里面设置对象类型的值

      ```xml
      <property name="courseList">
          <list>
              <ref bean="course1"></ref>
              <ref bean="course2"></ref>
          </list>
      </property>
      
      
      <bean id="course1" class="com.collectiontype.Course">
          <property name="name" value="Spring5 框架"></property>
      </bean>
      
      <bean id="course2" class="com.collectiontype.Course">
          <property name="name" value="MyBatis 框架"></property>
      </bean>
      ```

   6. 把集合注入部分提取出来

      1. 配置Spring：在Spring配置文件中引入名称空间util

         ```xml
         <?xml version="1.0" encoding="UTF-8"?>
         <beans xmlns="http://www.springframework.org/schema/beans"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xmlns:util="http://www.springframework.org/schema/util"
                xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                                    http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
         
         </beans>
         
         ```

      2. 代码实现

         ```xml
         <!-- 提取 list 集合类型属性注入 -->
         <util:list id="bookList">
             <!-- 对象属性：<ref bean="???"></ref> -->
             <value>易筋经</value>
             <value>九阴真经</value>
         </util:list>
         
         <!-- 提取出的 list 集合类型属性的注入使用-->
         <bean id="book" class="com.collectiontype.Book">
             <property name="list" ref="bookList"></property>
         </bean>
         ```

7. **FactoryBean（工厂Bean）**

   1. Spring有两种类型的bean，一种是普通bean，一种是工厂bean（FactoryBean）

   2. 普通bean：在配置文件中定义的类型就是返回值类型

   3. 工厂bean：在配置文件中定义的类型可以和返回值类型不一样

      ![image-20220501161041354](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220501161041354.png)

      - 第一步：创建类，让这个类做为工厂bean，事项接口FactoryBean
      - 第二步：实现接口里的方法，在实现方法中定义但会的bean类型

      ```java
      package com.factorybean;
      
      import com.collectiontype.Course;
      import org.springframework.beans.factory.FactoryBean;
      
      public class MyBean implements FactoryBean<Course> {
      
          //定义返回 bean
          @Override
          public Course getObject() throws Exception {
              Course course = new Course();
              course.setName("JavaSE");
              return course;
          }
      
          @Override
          public Class<?> getObjectType() {
              return null;
          }
      
          @Override
          public boolean isSingleton() {
              return FactoryBean.super.isSingleton();
          }
      }
      
      
      
      @Test
      public void testDemo(){
          ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext03.xml");
          Course myBean = context.getBean("myBean", Course.class); //配置文件里定义的是Mybean类型，但是实现了FactoryBean接口后，返回的是Course类型，所以传参时也应传Course.class
          System.out.println(myBean);
      }
      
      ```

      ```xml
      <bean id="myBean" class="com.factorybean.MyBean"></bean>
      ```

8. **bean的作用域**

   1. bean的作用域：在SPringle中，设置创建的bean实例是单实例还是多实例

   2. 默认情况下是单实例

      ```xml
      <util:list id="bookList">
          <!-- 对象属性：<ref bean="???"></ref> -->
          <value>易筋经</value>
          <value>九阴真经</value>
      </util:list>
      
      
      
      <!--2 提取出的 list 集合类型属性的注入使用-->
      <bean id="book" class="com.collectiontype.Book">
          <property name="list" ref="bookList"></property>
      </bean>
      ```

      ![image-20220503090829685](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220503090829685.png)

   3. 如何设置单实例还是多实例

      在Spring配置文件的bean标签中有一个属性（scope）用于设置单实例或者多实例.

      scope属性的值：

      - singleton：默认值，表示是单实例对象

      - prototype：表示是多实例对象

        ```xml
        <bean id="book" class="com.collectiontype.Book" scope="prototype">
            <property name="list" ref="bookList"></property>
        </bean>
        ```

        ![image-20220503091433181](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220503091433181.png)

      singleton和prototype的区别：

      1. singleton表示单实例对象，prototype表示多实例对象

      2. 设置scope的值为singleton时，在加载Spring配置文件时就会创建单实例对象

         设置scope的值为prototype时，不是在加载Spring配置文件时创建对象，而是在调用getBean()方法时才会创建多实例对象

9. **bean的生命周期**

   生命周期：对象从创建到销毁的过程。

   1. bean的生命周期

      - 第一步：通过构造器船舰bean实例（无参构造器）
      - 第二步：为 bean 的属性设置值和对其他 bean 引用（属性注入，调用set方法）
      - 第三步：调用 bean 的初始化的方法（这个初始化方法需要在实体类<!--也就是bean对应的class-->中手动写，然后在配置文件中进行配置）
      - 第四步：bean 可以使用了（获取到对象）
      - 第五步：当容器关闭时候，调用 bean 的销毁的方法（这个初始化方法也需要在实体类中手动写，然后在配置文件中进行配置，最后需要把IOC容器关闭）<!--((ClassPathXmlApplicationContext)context).close();-->

   2. 代码实现

      ```java
      package com.bean;
      
      public class Order {
          private String name;
      
          public Order() {
              System.out.println("第一步：通过构造器船舰bean实例");
          }
      
          public void setName(String name) {
              this.name = name;
              System.out.println("第二步：属性注入");
          }
      
      
          //创建执行的初始化的方法
          public void init(){
              System.out.println("第三步：执行初始化方法");
          }
      
          //创建执行的销毁的方法
          public void destory(){
              System.out.println("第五步：执行销毁的方法");
          }
      }
      
      ```

      ```xml
      <bean id="order" class="com.bean.Order" init-method="init" destroy-method="destory">
          <property name="name" value="Java核心基础卷"></property>
      </bean>
      ```

      ```java
      @Test
      public void testBean(){
          ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext04.xml");
          Order order = context.getBean("order", Order.class);
          System.out.println("第四步：获取创建bean实例对象");
          System.out.println(order);
      
          //ApplicationContext中没有close方法
          ((ClassPathXmlApplicationContext)context).close();
      }
      ```

   3. 输出结果

      ![image-20220503103244840](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220503103244840.png)

   4. bean的后置处理器，bean的生命周期有七步

      - 第一步：通过构造器船舰bean实例（无参构造器）
      - 第二步：为 bean 的属性设置值和对其他 bean 引用（属性注入，调用set方法）
      - 第三步：在bean初始化之前，把 bean 实例传递 bean 后置处理器的方法 postProcessBeforeInitialization 
      - 第四步：调用 bean 的初始化的方法（这个初始化方法需要在实体类<!--也就是bean对应的class-->中手动写，然后在配置文件中进行配置）
      - 第五步：在bean初始化之后，把 bean 实例传递 bean 后置处理器的方法 postProcessAfterInitializatio
      - 第六步：bean 可以使用了（获取到对象）
      - 第七步：当容器关闭时候，调用 bean 的销毁的方法（这个初始化方法也需要在实体类中手动写，然后在配置文件中进行配置，最后需要把IOC容器关闭）<!--((ClassPathXmlApplicationContext)context).close();-->

   5. 添加后置处理器效果

      ```java
      package com.bean;
      
      import org.springframework.beans.BeansException;
      import org.springframework.beans.factory.config.BeanPostProcessor;
      
      public class BeanPost implements BeanPostProcessor {
          @Override
          public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
              System.out.println("bean初始化之前");
              return bean;
          }
      
          @Override
          public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
              System.out.println("bean初始化之后");
              return bean;
          }
      }
      
      ```

      ```xml
      <bean id="order" class="com.bean.Order" init-method="init" destroy-method="destory">
          <property name="name" value="Java核心基础卷"></property>
      </bean>
      
      <!-- bean的后置处理器配置 -->
      <!--BeanPost类实现了BeanPostProcessor接口，Spring可以将其自动识别为后置处理器-->
      <!--这样配置后置处理器后，该配置文件中的所有bean都添加后置处理器的处理-->
      <bean id="beanPost" class="com.bean.BeanPost"></bean>
      ```

      ![image-20220503105343038](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220503105343038.png)
   
10. **xml自动装配**（使用较少）

   根据指定的装配规则（属性名称或者属性类型），Spring会自动将匹配的属性值进行注入

   ```xml
   <!--自动装配：
         bean标签中autowire属性，配置自动装配
         auowire有两个值：
            byName：根据属性名称注入，要注入的bean的id值要和类中的属性名称一样，例如：Emp中的dept属性名和Dept的bean的id值一样
            byType：根据属性类型注入，要注意使用该方式时，同一类型的bean只能存在一个-->
   
   <bean id="emp" class="com.autowire.Emp" autowire="byName">
       <!--<property name="dept" ref="dept"></property>-->
   </bean>
   <bean id="dept" class="com.autowire.Dept"></bean>
   ```

11. 引入外部属性文件

    1. 直接配置数据库信息

       1. 配置德鲁伊连接池

       2. 引入德鲁伊连接池依赖 jar 包

          ```xml
          <!--直接配置连接池-->
          <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
               <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
               <property name="url" value="jdbc:mysql://localhost:3306/bookdb"></property>
               <property name="username" value="root"></property>
               <property name="password" value="zgy1314521"></property>
          </bean>
          ```

    2. 引入外部属性文件配置数据库连接池

       1. 创建外部属性文件，properties 格式文件，写数据库信息

          ![](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220503175025256.png)

       2. 把外部 properties 属性文件引入到 spring 配置文件中

          引入 context 名称空间

          ```xml
          <beans xmlns="http://www.springframework.org/schema/beans"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xmlns:context="http://www.springframework.org/schema/context"
                 xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                                     http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
              
          </beans>
          ```

          在 spring 配置文件使用标签引入外部属性文件

          ```xml
          <?xml version="1.0" encoding="UTF-8"?>
          <beans xmlns="http://www.springframework.org/schema/beans"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xmlns:context="http://www.springframework.org/schema/context"
                 xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                                     http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
          
              <context:property-placeholder location="classpath*:jdbc.properties"/>
          
              <bean id="dataSourse" class="com.alibaba.druid.pool.DruidDataSource">
                  <property name="driverClassName" value="${jdbc.driver}"></property>
                  <property name="url" value="${jdbc.url}"></property>
                  <property name="username" value="${jdbc.user}"></property>
                  <property name="password" value="${jdbc.pwd}"></property>
              </bean>
          </beans>
          ```

          

### **基于注解方式**



