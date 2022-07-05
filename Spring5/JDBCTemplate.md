# 概念和准备

1. 什么是 JdbcTemplate

   Spring 框架对 Jdbc 进行了封装，使用 JdbcTemplate 方便实现对数据库的操作

2. 准备工作

   1. 引入相关 jar 包

      ![image-20220705100459920](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220705100459920.png)

   2. 在 Spring 配置文件中配置数据库连接池

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      
          <!-- 数据库连接池的配置 -->
          <!--<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
              &lt;!&ndash;url对编码进行设置,否则中文格式有误&ndash;&gt;
              <property name="url" value="jdbc:mysql:///book?useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false"/>
              <property name="username" value="root"/>
              <property name="password" value="zgy1314521"/>
              <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
          </bean>-->
      
          <context:property-placeholder location="classpath:jdbc.properties"/>
      
          <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
              <property name="driverClassName" value="${jdbc.driver}"></property>
              <property name="url" value="${jdbc.url}"></property>
              <property name="username" value="${jdbc.user}"></property>
              <property name="password" value="${jdbc.pwd}"></property>
          </bean>
      
      </beans>
      ```
   
   3. 配置 JdbcTemplate 对象，并注入 dataSourse 属性
   
      ```xml
      <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
              <property name="dataSource" value="dataSourse"></property>
      </bean>
      ```
   
   4. 创建 service 和 dao 类，并且在 dao中注入 jdbcTemplate 对象
   
      - 配置文件
   
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns:context="http://www.springframework.org/schema/context"
               xmlns:aop="http://www.springframework.org/schema/aop"
               xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                                   http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                                    http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
        
        <!-- 开启组件扫描 -->
            <context:component-scan base-package="com.service,com.dao"></context:component-scan>
        
        
            <!-- 数据库连接池的配置 -->
            <!--<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
                &lt;!&ndash;url对编码进行设置,否则中文格式有误&ndash;&gt;
                <property name="url" value="jdbc:mysql:///book?useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="zgy1314521"/>
                <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
            </bean>-->
        
            <context:property-placeholder location="classpath:jdbc.properties"/>
        
            <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
                <property name="driverClassName" value="${jdbc.driver}"></property>
                <property name="url" value="${jdbc.url}"></property>
                <property name="username" value="${jdbc.user}"></property>
                <property name="password" value="${jdbc.pwd}"></property>
            </bean>
        
            <!--  JdbcTemplate 对象 -->
            <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
                <property name="dataSource" ref="dataSource"></property>
            </bean>
        
        </beans>
        ```
      
      - DAO
      
        ```java
        package com.dao.impl;
        
        import com.dao.BookDAO;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.beans.factory.annotation.Qualifier;
        import org.springframework.jdbc.core.JdbcTemplate;
        import org.springframework.stereotype.Repository;
        
        @Repository(value = "bookDAO")      //默认值为bookDAOImpl
        public class BookDAOImpl implements BookDAO {
        
            @Autowired
            @Qualifier(value = "jdbcTemplate")
            private JdbcTemplate jdbcTemplate;
            
        }
        
        ```
      
      - Service
      
        ```java
        package com.service.impl;
        
        import com.dao.BookDAO;
        import com.service.BookService;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.beans.factory.annotation.Qualifier;
        import org.springframework.stereotype.Service;
        
        @Service(value = "bookService")     //默认值为bookServiceImpl
        public class BookServiceImpl implements BookService {
        
            @Autowired
            @Qualifier(value = "bookDAO")
            private BookDAO bookDAO;
        }
        
        ```

# JdbcTemplate 操作数据库

## 1、添加

1. 创建对应实体类

   ```java
   package com.pojo;
   
   public class Book {
       private int id;
       private String bookName;
       private String bookStatus;
   
       public Book() {
       }
   
       public Book(String bookName, String bookStatus) {
           this.bookName = bookName;
           this.bookStatus = bookStatus;
       }
   
       public int getId() {
           return id;
       }
   
       public void setId(int id) {
           this.id = id;
       }
   
       public String getBookName() {
           return bookName;
       }
   
       public void setBookName(String bookName) {
           this.bookName = bookName;
       }
   
       public String getBookStatus() {
           return bookStatus;
       }
   
       public void setBookStatus(String bookStatus) {
           this.bookStatus = bookStatus;
       }
   }
   
   ```

   

2. 编写 service 和 dao 

   1. 在 dao 进行数据库的添加操作

   2. 调用 JdbcTemplate 对象里的 update 方法实现添加操作

      ![image-20220705110629650](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220705110629650.png)

      - 第一个参数：sql 语句
   - 第二个参数：可变参数，设置 sql 语句值，是一个数组
      

3. 测试类

   ```java
   @Test
   public  void testAdd() {
       //有配置文件new ClassPathXmlApplicationContext()
       //完全注解开发new AnnotationConfigApplicationContext()
       ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
       BookServiceImpl bookService = context.getBean("bookService", BookServiceImpl.class);
   
       bookService.add(new Book("C语言","正常"));
   }
   ```

## 2、删除和修改



与添加类似，调用 JdbcTemplate 对象里的 update 方法实现添加操作

测试类：

```java
@Test
public void testDel(){
    ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
    BookServiceImpl bookService = context.getBean("bookService", BookServiceImpl.class);

    bookService.del(7);
}

@Test
public void testUpdate(){
    ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
    BookServiceImpl bookService = context.getBean("bookService", BookServiceImpl.class);

    Book book = new Book("言魔", "正常");
    book.setId(5);
    bookService.update(book);
}
```

## 3、查询

### 1、 返回某个值（例如查询表中有多少条记录）（特殊查询）

调用 JdbcTemplate 对象里的 queryForObject 方法实现添加操作

![image-20220705160927178](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220705160927178.png)

- 第一个参数：sql 语句
- 第二个参数：返回类型的 class

测试类：

```java
@Test
public void testSelectCount(){
    ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
    BookServiceImpl bookService = context.getBean("bookService", BookServiceImpl.class);

    int count = bookService.selectCount();
    System.out.println(count);
}
```



### 2、 返回对象

![image-20220705163905502](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220705163905502.png)

- 第一个参数：sql 语句
- 第二个参数：RowMapper 是接口，针对返回不同类型的数据，使用这个接口的实现类完成对数据封装
- 第三个参数：sql 语句值

测试类：

```java
@Test
public void testPojo(){
    ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
    BookServiceImpl bookService = context.getBean("bookService", BookServiceImpl.class);

    Book book = bookService.getBookById(5);
    System.out.println(book);
}
```

### 3、 返回集合

![image-20220705163621198](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220705163621198.png)

- 第一个参数：sql 语句
- 第二个参数：RowMapper 是接口，针对返回不同类型的数据，使用这个接口的实现类完成对数据封装
- 第三个参数：sql 语句值

测试类：

```java
@Test
public void testList(){
    ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
    BookServiceImpl bookService = context.getBean("bookService", BookServiceImpl.class);

    List<Book> bookList = bookService.getBookList();
    System.out.println(bookList);
}
```

## 4、批量操作（操作表里的多条记录）

### 批量添加

![image-20220705173212675](https://typora-yanmo.oss-cn-zhangjiakou.aliyuncs.com/img/image-20220705173212675.png)

- 第一个参数：sql 语句
- 第二个参数：List 集合，用来添加多条记录，一个 Object 数组存储着一条记录的 sql 语句值

测试类：

```java
@Test
public void testBatchAdd(){
    ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
    BookServiceImpl bookService = context.getBean("bookService", BookServiceImpl.class);

    List<Object[]> list = new ArrayList<>();
    Object[] o1 = {"java","a"};
    Object[] o2 = {"C++","b"};
    Object[] o3 = {"MySQL","c"};

    list.add(o1);
    list.add(o2);
    list.add(o3);

    bookService.batchAdd(list);
}
```

### 批量删除和批量修改

与批量删除类似。

测试类：

```java
@Test
public void testBatchDel(){
    ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
    BookServiceImpl bookService = context.getBean("bookService", BookServiceImpl.class);

    List<Object[]> list = new ArrayList<>();
    //数字和字符串都可以  1 或者 ”1“
    Object[] o1 = {10};
    Object[] o2 = {11};

    list.add(o1);
    list.add(o2);

    bookService.batchDel(list);
}


@Test
public void batchUpdate(){
    ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
    BookServiceImpl bookService = context.getBean("bookService", BookServiceImpl.class);

    List<Object[]> list = new ArrayList<>();
    Object[] o1 = {"C#","OK",12};
    Object[] o2 = {"PHP","OK",13};

    list.add(o1);
    list.add(o2);

    bookService.batchUpdate(list);

}
```

## 5、完整代码

### DAO

```java
package com.dao;

import com.pojo.Book;

import java.util.List;

public interface BookDAO {
    void add(Book book);
    void del(int id);
    void update(Book book);

    int selectCount();
    Book getBookById(int id);
    List<Book> getBookList();

    void batchAdd(List<Object[]> batchArgs);
    void batchDel(List<Object[]> batchArgs);
    void BatchUpdate(List<Object[]> batchArgs);
}
```

### DAOImpl

```java
package com.dao.impl;

import com.dao.BookDAO;
import com.pojo.Book;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository(value = "bookDAO")      //默认值为bookDAOImpl
public class BookDAOImpl implements BookDAO {

    @Autowired
    @Qualifier(value = "jdbcTemplate")
    private JdbcTemplate jdbcTemplate;

    @Override
    public void add(Book book) {
        //sql语句
        String sql = "insert into t_book values(0,?,?)";
        Object[] args = {book.getBookName(), book.getBookStatus()};
        jdbcTemplate.update(sql, args);
    }

    @Override
    public void del(int id) {
        String sql = "delete from t_book where id = ?";
        jdbcTemplate.update(sql, id);

    }

    @Override
    public void update(Book book) {
        String sql = "update t_book set bookName = ?,bookStatus = ? where id = ?";
        Object[] args = {book.getBookName(), book.getBookStatus(),book.getId()};
        jdbcTemplate.update(sql, args);

    }

    //查询表中记录数
    @Override
    public int selectCount() {
        String sql = "select count(*) from t_book";
        Integer count = jdbcTemplate.queryForObject(sql, Integer.class);
        return count;
    }

    @Override
    public Book getBookById(int id) {
        String sql = "select * from t_book where id = ?";
        Book book = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Book>(Book.class), id);
        return book;
    }

    @Override
    public List<Book> getBookList() {

        String sql = "select * from t_book";
        List<Book> bookList = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Book>(Book.class));
        return bookList;
    }

    @Override
    public void batchAdd(List<Object[]> batchArgs) {
        String sql = "insert into t_book values(0,?,?)";
        jdbcTemplate.batchUpdate(sql,batchArgs);
    }

    @Override
    public void batchDel(List<Object[]> batchArgs) {
        String sql = "delete from t_book where id = ?";
        jdbcTemplate.batchUpdate(sql,batchArgs);
    }

    @Override
    public void BatchUpdate(List<Object[]> batchArgs) {
        String sql = "update t_book set bookName = ?,bookStatus = ? where id = ?";
        jdbcTemplate.batchUpdate(sql,batchArgs);
    }

}
```

### Service

```java
package com.service;

import com.pojo.Book;

import java.util.List;

public interface BookService {
    void add(Book book);
    void del(int id);
    void update(Book book);

    int selectCount();
    Book getBookById(int id);
    List<Book> getBookList();

    void batchAdd(List<Object[]> batchArgs);
    void batchDel(List<Object[]> batchArgs);
    void batchUpdate(List<Object[]> batchArgs);
}
```

### ServiceImpl

```java
package com.service.impl;

import com.dao.BookDAO;
import com.pojo.Book;
import com.service.BookService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

import java.util.List;

@Service(value = "bookService")     //默认值为bookServiceImpl
public class BookServiceImpl implements BookService {

    @Autowired
    @Qualifier(value = "bookDAO")
    private BookDAO bookDAO;

    @Override
    public void add(Book book) {
        bookDAO.add(book);
    }

    @Override
    public void del(int id) {
        bookDAO.del(id);
    }

    @Override
    public void update(Book book) {
        bookDAO.update(book);
    }

    //查询表中记录数
    @Override
    public int selectCount() {
        return bookDAO.selectCount();
    }

    @Override
    public Book getBookById(int id) {
        return bookDAO.getBookById(id);
    }

    @Override
    public List<Book> getBookList() {
        return bookDAO.getBookList();
    }

    @Override
    public void batchAdd(List<Object[]> batchArgs) {
        bookDAO.batchAdd(batchArgs);
    }

    @Override
    public void batchDel(List<Object[]> batchArgs) {
        bookDAO.batchDel(batchArgs);
    }

    @Override
    public void batchUpdate(List<Object[]> batchArgs) {
        bookDAO.BatchUpdate(batchArgs);
    }

}
```
