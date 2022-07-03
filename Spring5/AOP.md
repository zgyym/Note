# AOP概念

什么是AOP？

1. 面向切面编程（面向方面编程），利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使得 业务逻辑各部分之间的**耦合度降低**，提高程序的可重用性，同时提高了开发的效率。

2. 通俗描述：就是在不修改源代码的基础上，在主干功能里添加新的功能

3. 使用登录例子说明 AOP

   ![图3](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/%E5%9B%BE3.png)

# AOP底层原理

1. AOP底层使用动态代理

   1. 有接口的情况，使用JDK动态代理

      - 创建接口实现类的代理对象，增强类中的方法

        ![image-20220702194845100](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220702194845100.png)

   2. 没有接口的情况，使用CGLIB动态代理

      - 创建子类的代理对象，增强类中的方法

        ![image-20220702195005606](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220702195005606.png)

        

## JDK动态代理

1. 使用Proxy类里的方法创建代理对象

![image-20220703081106804](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220703081106804.png)

​				调用 newProxyInstance 方法

![image-20220703081124313](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220703081124313.png)

​				第一个参数：类加载器

​				第二个参数：增强方法所在的类，这个类实现的**接口**；**数组形式**，支持多个接口

​				第三个参数：实现这个接口 InvocationHandler，创建代理对象，写增强的部分

2. 编写 JDK 动态代理代码

   1. 创建接口

      ```java
      package dao;
      
      public interface UserDAO {
          int add(int a,int b);
          String update(String id);
      }
      
      ```

      

   2. 实现接口

      ```java
      package dao.impl;
      
      import dao.UserDAO;
      
      public class UserDAOImpl implements UserDAO {
          @Override
          public int add(int a, int b) {
              return a + b;
          }
      
          @Override
          public String update(String id) {
              return id;
          }
      }
      
      ```

      

   3. 使用 Proxy 类创建接口代理对象

      ```java
      import dao.UserDAO;
      import dao.impl.UserDAOImpl;
      
      import java.lang.reflect.InvocationHandler;
      import java.lang.reflect.Method;
      import java.lang.reflect.Proxy;
      import java.util.Arrays;
      
      public class JDKProxy {
          public static void main(String[] args) {
              //创建代理对象
              Class[] objs = {UserDAO.class};
      
              UserDAOImpl userDAO = new UserDAOImpl();
      
              //这里创建的是 UserDAOImpl （接口实现类）的代理对象，所以 new Handler() 传入userDAO
              //强转
              UserDAO dao = (UserDAO) Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), objs, new Handler(userDAO));
      
              //通过代理对象dao调用UserDAO中的方法，传入参数后直接进入Handler中的invoke方法中
              int add = dao.add(1, 2);
              System.out.println("===========");
              dao.update("dawd");
          }
      }
      
      //创建代理对象代码
      class Handler implements InvocationHandler {
      
          //创建的是谁的代理对象，就把谁传过来
          //通过有参构造传递
          private Object obj;
      
          public Handler(Object obj) {
              this.obj = obj;
          }
      
          //增强的逻辑
          @Override
          public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
              
              //方法之前
              System.out.println("方法之前执行...." + method.getName() + " :传递的参数..." + Arrays.toString(args));
      
              //被增强的方法执行,此处的obj是UserDAOImpl userDAO
              Object res = method.invoke(obj, args);
              //方法之后
              System.out.println("方法之后执行...." + obj);
              return res;
      
          }
      }
      ```

   4. **通过代理对象 dao 调用 UserDAO 接口中的方法，传入参数后直接进入Handler中的invoke方法中，进行实现类中对应方法的增强**



# AOP术语

1. 连接点

   类里面哪些方法可以被增强，这些方法就被称为连接点

2. 切入点

   实际被真正增强的方法，称为切入点

3. 通知（增强）

   1. 实际增强的逻辑部分被称为通知（增强），比如登录时权限判断部分的代码
   2. 通知有多种类型
      - 前置通知
      - 后置通知
      - 环绕通知
      - 异常通知
      - 最终通知

4. 切面

   把通知应用到切入点的过程，是一个动作



# AOP操作

## 准备工作

1. Spring框架一般是基于AspectJ实现AOP操作

   AspectJ 不是Spring 的组成部分，而是独立的AOP框架，一般把 AspectJ 和Spring 矿建仪器使用，进行AOP操作 。

2. 基于AspectJ 实现AOP操作

   1. 基于注解方式实现
   2. 基于xml配置文件实现

3. 引入相关依赖

   ![image-20220703101732347](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220703101732347.png)

4. 切入点表达式

   1. 切入点表达式的作用：知道对那个类里的哪个方法进行增强

   2. 语法结构：execution([权限修饰符] [返回值类型] [被增强类全路径] [方法名称] ([参数列表])**)**

      **返回值类型可以省略**

      ![image-20220703102408552](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220703102408552.png)

## AspectJ注解

1. 创建类，在类中定义方法（被增强类和被增强方法）

   ```java
   package aop;
   
   public class User {
       public void add(){
           System.out.println("add........");
       }
   }
   ```

   

2. 创建增强类（编写增强逻辑）

   在增强类里面，创建方法，让不同方法代表不同通知类型

   ```java
   package aop;
   
   public class UserProxy {
       public void before(){
           System.out.println("before..........");
       }
   }
   
   ```

   

3. 进行通知的配置

   1. 开启 Spring 组件扫描

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:context="http://www.springframework.org/schema/context"
             xmlns:aop="http://www.springframework.org/schema/aop"
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                                 http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                                  http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
      
      
      
      <!--开启组件扫描-->
          <context:component-scan base-package="aop"></context:component-scan>
      </beans>
      ```

   2. 使用注解创建 User 和 UserProxy 对象

      ```java
      package aop;
      import org.springframework.stereotype.Component;
      
      @Component(value = "user")
      public class User {
          public void add(){
              System.out.println("add........");
          }
      }
      
      
      
      package aop;
      import org.springframework.stereotype.Component;
      
      @Component(value = "userProxy")
      public class UserProxy {
          public void before(){
              System.out.println("before..........");
          }
      }
      
      ```

   3. 在增强类 UserProxy上添加 @Aspect 注解

      ```java
      package aop;
      
      import org.aspectj.lang.annotation.Aspect;
      import org.springframework.stereotype.Component;
      
      @Component(value = "userProxy")
      @Aspect         //生成代理对象
      public class UserProxy {
          public void before(){
              System.out.println("before..........");
          }
      }
      
      ```

   4. 在 Spring 配置文件中开启生成代理对象

      ```xml
      <!-- 开启Aspect生成代理对象 -->
          <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
      ```

      

4. 配置不同类型的通知

   在增强类中，在做为通知的方法上面添加通知类型注解，使用切入点表达式配置

   ```java
   package aop;
   
   import org.aspectj.lang.ProceedingJoinPoint;
   import org.aspectj.lang.annotation.*;
   import org.springframework.stereotype.Component;
   
   @Component(value = "userProxy")
   @Aspect         //生成代理对象
   public class UserProxy {
   
       //前置通知
       @Before(value = "execution(* aop.User.add(..))")
       public void before(){
           System.out.println("before..........");
       }
   
       //后置通知
       @AfterReturning(value = "execution(* aop.User.add(..))")
       public void afterReturning(){
           System.out.println("AfterReturning..........");
       }
   
       //环绕通知
       @Around(value = "execution(* aop.User.add(..))")
       public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
           System.out.println("环绕之前.........");
           //被增强的方法执行
           proceedingJoinPoint.proceed();
           System.out.println("环绕之后.........");
       }
   
   
       //异常通知
       @AfterThrowing(value = "execution(* aop.User.add(..))")
       public void afterThrowing() {
           System.out.println("afterThrowing.........");
       }
   
       //最终通知
       @After(value = "execution(* aop.User.add(..))")
       public void after() {
           System.out.println("after.........");
       }
   }
   
   ```

5. 相同切入点的抽取

   ```java
   @Pointcut(value = "execution(* aop.User.add(..))")
       public void pointDemo(){
   
       }
   
       //前置通知
       @Before(value = "pointDemo()")
       public void before(){
           System.out.println("before..........");
       }
   ```

6. 有多个增强类对同一个方法进行增强，可以设置增强类的优先级

   在增强类上添加注解 @Order(数字类型值) ，数字类型值越小，优先级越高

   ```java
   package aop;
   
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.Before;
   import org.springframework.core.annotation.Order;
   import org.springframework.stereotype.Component;
   
   @Component(value = "personProxy")
   @Aspect
   @Order(1)
   public class PersonProxy {
   
       @Before(value = "execution(* aop.User.add(..))")
       public void personBefore(){
           System.out.println("personBefore......");
       }
   }
   
   
   
   
   
   
   
   package aop;
   
   import org.aspectj.lang.ProceedingJoinPoint;
   import org.aspectj.lang.annotation.*;
   import org.springframework.core.annotation.Order;
   import org.springframework.stereotype.Component;
   
   @Component(value = "userProxy")
   @Aspect         //生成代理对象
   @Order(2)
   public class UserProxy {
   
       //相同切入点的抽取
       @Pointcut(value = "execution(* aop.User.add(..))")
       public void pointDemo(){
   
       }
   
       //前置通知
       @Before(value = "pointDemo()")
       public void before(){
           System.out.println("before..........");
       }
   
       //后置通知
       @AfterReturning(value = "execution(* aop.User.add(..))")
       public void afterReturning(){
           System.out.println("AfterReturning..........");
       }
   
       //环绕通知
       @Around(value = "execution(* aop.User.add(..))")
       public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
           System.out.println("环绕之前.........");
           //被增强的方法执行
           proceedingJoinPoint.proceed();
           System.out.println("环绕之后.........");
       }
   
   
       //异常通知
       @AfterThrowing(value = "execution(* aop.User.add(..))")
       public void afterThrowing() {
           System.out.println("afterThrowing.........");
       }
   
       //最终通知
       @After(value = "execution(* aop.User.add(..))")
       public void after() {
           System.out.println("after.........");
       }
   
   
   
   }
   
   ```

7. 完全注解开发

   ```java
   package config;
   
   import org.springframework.context.annotation.ComponentScan;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.context.annotation.EnableAspectJAutoProxy;
   
   @Configuration      //作为配置类来替代xml配置文件
   @ComponentScan(basePackages = {"aop"})      //组件扫描
   @EnableAspectJAutoProxy(proxyTargetClass = true)    //Aspect生成代理对象
   public class ConfigAOP {
   }
   
   ```

   

## AspectJ配置文件（了解）

![image-20220703203404056](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220703203404056.png)

