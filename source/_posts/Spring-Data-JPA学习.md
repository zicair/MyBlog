---
title: Spring Data JPA学习
tags:
  - JPA
  - SpringBoot
categories:
  - SpringBoot
date: 2020-08-24 16:41:22
---

全称Java Persistence API，可以通过注解或者XML描述【对象-关系表】之间的映射关系，并将实体对象持久化到数据库中。

<!--more-->

# 1. 回顾JDBC操作数据库

![image-20200825114031552](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/Spring-Data-JPA学习/image-20200825114031552.png)

## 1.1 ORM思想

**ORM思想的引入：操作实体类，就相当于操作数据库表**

建立两个映射关系：

>实体类和表的映射关系

>实体类中属性和表中字段的映射关系

有了这个ORM思想，我们不再重点关注SQL语句

**实现了ORM思想的框架：MyBatis，Hibernate**

## 1.2 JPA规范

jpa规范，由SUN公司定义，内部是由接口和抽象类组成，**JPA只是规范，虽然它也它体现了ORM的思想，但具体的实现是由一些具体的框架实现的。**

![image-20200825114610394](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/Spring-Data-JPA学习/image-20200825114610394.png)

![image-20200825114652198](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/Spring-Data-JPA学习/image-20200825114652198.png)

## 1.3 Spring Data JPA

Spring Data JPA是Spring提供的一套对JPA操作更加高级的封装，是在JPA规范下的专门用来进行数据持久化的解决方案。底层默认的是依赖 Hibernate JPA 来实现的。

## 1.4 Spring Data

Spring Data是Spring 的一个子项目。用于简化数据库访问，支持NoSQL和关系数据库存储。其主要目标是使数据库的访问变得方便快捷。而`Spring Data JPA`只是其中的一个。

![image-20200824170242205](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/Spring-Data-JPA学习/image-20200824170242205.png)

# 2. Spring Data JPA

全称Java Persistence API，可以通过注解或者XML描述【对象-关系表】之间的映射关系，并将实体对象持久化到数据库中。JPA仅仅是一种规范，也就是说JPA仅仅定义了一些接口，而接口是需要实现才能工作的。

## 2.1 JPA的特点

![image-20200824170812538](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/Spring-Data-JPA学习/image-20200824170812538.png)

1. Spring Data JPA使得那些以JPA接口为规范的应用更加方便， 致力于减少数据访问层（DAO）的开发量。

2. Spring Data JPA 底层默认的使用的是 Hibernate 来做的 JPA 实现。

3. 其技术特点：我们只需要定义接口并集成 Spring Data JPA 中所提供的接口就可以了。不需要编写接口实现类。

## 2.2 基本注解

### @Entity

```java
@Entity 写在类上，用于指明一个类与数据库表相对应。
    属性：
    name，可选，用于自定义映射的表名。若没有，则默认以类名为表名。

【举例1：默认类名为表名】
@Entity
public class Blog {
}

【举例2：自定义表名】
@Entity(name="t_blog")
public class Blog {
}
```

### @Table

```java
@Table 写在类上，一般与 @Entity 连用，用于指定数据表的相关信息。
    属性：
    name， 对应数据表名。
    catalog， 可选，对应关系数据库中的catalog。
    schema，可选，对应关系数据库中的schema。

【举例：】
import javax.persistence.Entity;
import javax.persistence.Table;

@Entity(name = "blog")
@Table(name = "t_blog")
public class Blog {
}

注：若 @Entity 与 @Table 同时定义了 name 属性，那以 @Table 为主。
```

### @Id、@GeneratedValue

```java
@Id 写在类中的变量上，用于指定当前变量为主键 Id。一般与 @GeneratedValue 连用。
@GeneratedValue 与 @Id 连用，用于设置主键生成策略（自增主键，依赖数据库）。
注：
    @GeneratedValue(strategy = GenerationType.AUTO） 主键增长方式由数据库自动选择，当数据库选择AUTO方式时就会自动生成hibernate_sequence表。
    
    @GeneratedValue(strategy = GenerationType.IDENTITY) 要求数据库选择自增方式，oracle不
支持此种方式，mysql支持。
    
    @GeneratedValue(strategy = GenerationType.SEQUENCE) 采用数据库提供的sequence机制生
成主键，mysql不支持。


【举例：】
import lombok.Data;
import javax.persistence.*;

@Entity
@Table(name = "t_blog")
@Data
public class Blog {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
}
```

### @Column

```java
@Column 写在类的变量上，用于指定当前变量映射到数据表中的列的属性（列名，是否唯一，是否允许为空，是否允许更新等）。
属性：
    name: 列名。 
    unique: 是否唯一 
    nullable: 是否允许为空 
    insertable: 是否允许插入 
    updatable: 是否允许更新 
    length: 定义长度
    
【举例：】
import lombok.Data;
import javax.persistence.*;

@Entity
@Table(name = "t_blog")
@Data
public class Blog {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "name", length = 36, unique = false, nullable = false, insertable = true, updatable = true)
    private String name;
}
```

### @Temporal

```java
@Temporal 用于将 java.util 下的时间日期类型转换 并存于数据库中（日期、时间、时间戳）。
属性：
    TemporalType.DATE       java.sql.Date日期型，精确到年月日，例如“2019-12-17”
    TemporalType.TIME       java.sql.Time时间型，精确到时分秒，例如“2019-12-17 00:00:00”
    TemporalType.TIMESTAMP  java.sql.Timestamp时间戳，精确到纳秒，例如“2019-12-17 00:00:00.000000001”

【举例：】
import lombok.Data;
import javax.persistence.*;
import java.util.Date;

@Entity
@Table(name = "t_blog")
@Data
public class Blog {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "name", length = 36, unique = false, nullable = false, insertable = true, updatable = true)
    private String name;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createTime;

    @Temporal(TemporalType.DATE)
    private Date updateTime;
}
```

# 3. Spring Data JPA的使用

## 3.1 配置环境

在 `application.yml` 文件中添加我们需要的配置信息

```yaml
spring:
  datasource:
    url: jdbc:mysql:///db?serverTimezone=GMT
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update # 数据库没有表时自动构建，
    database: mysql # 指定数据库类型
    generate-ddl: true # 自动生成
    show-sql: true # 现实sql到控制台
    database-platform: org.hibernate.dialect.MySQL8Dialect # 数据库方言 DataBbase枚举内获取
```

## 3.2 实体类映射

创建映射实体类 `DeptEntity` , 这里用来 `lombok` 插件省去了 `get/set` 方法，对于 `@Column` 注解是可以放在 `get` 方法上的

```java
@Entity
@Table(name = "dept", schema = "db", catalog = "")
@ToString
@Data
public class DeptEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "DEPTNO", nullable = false)
    private int deptno;

    // name : 数据库中的列名  nullable  : 是否为空  length:长度
    @Column(name = "DNAME", nullable = true, length = 14)
    private String dname;

    @Column(name = "LOC", nullable = true, length = 13)
    private String loc;

    @Column(name = "flag", nullable = true)
    private Integer flag;

    @Column(name = "type", nullable = true, length = 20)
    private String type;
    
}
```

编写完实体后，然后添加相应的 `Dao` 层接口，需要继承 `JPA` 的接口 `JpaRepository` , 代码如下：

```java
public interface DeptRepository extends JpaRepository<DeptEntity,Integer> {
    
}
```

> 我们的 dao 层是一个接口，没有具体的实现，在继承的接口中已经定义了一些常用的方法供我们使用，后面详细介绍

## 3.3 查询方法

### 1) 直接调用接口中的方法

```java
@SpringBootTest
public class SpringJpaTest {
    @Autowired
    private DeptRepository deptRepository;

    @Test
    public void jpaTest(){
        List<DeptEntity> list = deptRepository.findAll();
        System.out.println(list);
    }
    
    // 相应的添加与查询排序方法的测试全部如下：
    @Test
    public void query(){
        Sort deptno = Sort.by(Sort.Direction.DESC,"deptno");
        List<DeptEntity> all = deptRepository.findAll(deptno);
        System.out.println(all);
    }

    @Test
    public void insert(){
        DeptEntity entity = new DeptEntity();
        entity.setDname("质量控制部门");
        entity.setFlag(1);
        entity.setType("test");
        //保存同时将保存结果返回
        DeptEntity save = deptRepository.save(entity);
        System.out.println(save);
    }
}

```

### 2) jpql的查询方式

- jpql ： jpa query language （jpq查询语言）

- 特点：

  语法或关键字和sql语句类似
  查询的是类和类中的属性

- 需要将JPQL语句配置到接口方法上

  1.特有的查询：需要在dao接口上配置方法
  2.在新添加的方法上，使用注解的形式配置jpql查询语句
  3.注解 ： @Query

- 代码实现如下：

在dao接口中自定义方法并添加注解

```java
    /**
     * 通过 deptno查询
     * 使用@Query注解来编写jpql 语句  是面向对象的操作
     * sql:  select * from dept where deptno = ?
     * jpql: select * from DeptEntity where deptno = ?1
     *
     * 在jpql中占位符好指定索引，与参数列表对应，从1开始
     * @param id
     * @return
     */
    @Query(value="select * from DeptEntity where deptno = ?1")
    DeptEntity queryDept(Integer id);

    /**
     * DML 操作， 需要加@Modifying 注解
     * 在多个条件时要注意占位符下标的数字要和参数列表对应
     * 需要注意，DML 语句需要事务配置，需要加 @Transactional 注解，一般在业务层，而不再数据层，
     * @param name
     * @param id
     */
    @Query(value="update DeptEntity set dname=?1 where deptno=?2")
    @Modifying
    void updateName(String name,Integer id);
// JAVA
```

### 3) sql语句查询方式

- 特有的查询：需要在dao接口上配置方法
- 在新添加的方法上，使用注解的形式配置sql查询语句
- 注解 ：@Query

```java
value ：jpql语句 | sql语句
nativeQuery ：false（使用jpql查询） | true（使用本地查询：sql查询）   是否使用本地查询
```

示例 dao接口代码：

```java
 	/**
     * 需要添加 nativeQuery 参数来设置是都是sql查询， 默认是false ，是jpql查询
     * @param num
     * @return
     */
    @Query(value="select * from dept where flag = ?",nativeQuery=true)
    List<DeptEntity> queryList(Integer num);
```

### 4) 方法名规则查询

- 是对jpql查询，更加深入一层的封装

- 我们只需要按照SpringDataJpa提供的方法名称规则定义方法，不需要再配置jpql语句，完成查询

- 规则：

  - findBy开头： 代表查询

    对象中属性名称（首字母大写）

  - 含义：根据属性名称进行查询

规则：

![image-20200825122320853](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/Spring-Data-JPA学习/image-20200825122320853.png)

### 5) 自定义条件查询

JpaSpecificationExecutor 是一个接口。查询语句都定义在 Specification 中。搭配`JpaRepository`可以实现自定义的复杂条件查询。

步骤：
　　Step1：实现 Specification 接口（定义泛型，为查询的对象类型），重写 toPredicate() 方法。
　　Step2：定义 CriteriaBuilder 查询条件。

查看`JpaSpecificationExecutor`中定义的方法：

```java
public interface JpaSpecificationExecutor<T> {
    // 查询单个对象
  Optional<T> findOne(@Nullable Specification<T> var1);

    // 查询对象列表
  List<T> findAll(@Nullable Specification<T> var1);

    // 查询对象列表，并返回分页数据
  Page<T> findAll(@Nullable Specification<T> var1, Pageable var2);

    // 查询对象列表，并排序
  List<T> findAll(@Nullable Specification<T> var1, Sort var2);

    // 统计查询的结果
  long count(@Nullable Specification<T> var1);
}
```

查看参数`Specification`：

```java
public interface Specification<T> extends Serializable {

	long serialVersionUID = 1L;

	static <T> Specification<T> not(Specification<T> spec) {
		return Specifications.negated(spec);
	}

	static <T> Specification<T> where(Specification<T> spec) {
		return Specifications.where(spec);
	}

	default Specification<T> and(Specification<T> other) {
		return Specifications.composed(this, other, AND);
	}

	default Specification<T> or(Specification<T> other) {
		return Specifications.composed(this, other, OR);
	}
    
	@Nullable
	Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder);

```

使用：

```java
@Test
   void testSpecification() {
      // 定义内部类，泛型为 查询的对象
      Specification<Employee> specification = new
         Specification<Employee>() {

            @Override
            public Predicate toPredicate(Root root, CriteriaQuery criteriaQuery, CriteriaBuilder criteriaBuilder) {
               // 获取比较的属性
               Path<Object> name = root.get("name");
               // 构建查询条件, select * from emp where name = "tom";
               Predicate predicate = criteriaBuilder.equal(name, "tom");
               return predicate;
            }


         };
      List<Employee> employeeList = employeeDao.findAll(specification);
      for (Employee employee : employeeList) {
         System.out.println(employee);
      }
   }
```



