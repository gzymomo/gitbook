[TOC]

部分优质内容来源：
博客园：毕业男孩：[Spring Data JPA](https://www.cnblogs.com/biyenanhai/p/13167246.html)

# 1、JPA框架简介
JPA(Java Persistence API)意即Java持久化API，是Sun官方在JDK5.0后提出的Java持久化规范。主要是为了简化持久层开发以及整合ORM技术，结束Hibernate、TopLink、JDO等ORM框架各自为营的局面。JPA是在吸收现有ORM框架的基础上发展而来，易于使用，伸缩性强。

JPA的全称是Java Persistence API， 即Java 持久化API，是SUN公司推出的一套基于ORM的规范，内部是由一系列的接口和抽象类构成。JPA通过JDK 5.0注解描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。
JPA的优势：标准化、容器级特性的支持、简单方便、查询能力、高级特性。

## 1.1 JPA与Hibernate的关系：

JPA规范本质上就是一种ORM规范，注意不是ORM框架——因为JPA并未提供ORM实现，它只是制订了一些规范，提供了一些编程的API接口，但具体实现则由服务厂商来提供实现。JPA和Hibernate的关系就像JDBC和JDBC驱动的关系，JPA是规范，Hibernate除了作为ORM框架之外，它也是一种JPA实现。

![](https://img2020.cnblogs.com/blog/2059433/202006/2059433-20200619165727630-1660352068.png)

# 2、SpringBoot整合JPA

## Spring Data JPA概述：
　　Spring Data JPA 是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套JPA应用框架，可使开发者用极简的代码即可实现对数据库的访问和操作。它提供了包括增删改查等在内的常用功能，且易于扩展！学习并使用 Spring Data JPA 可以极大提高开发效率！Spring Data JPA 让我们解脱了DAO层的操作，基本上所有CRUD都可以依赖于它来实现,在实际的工作工程中，推荐使用Spring Data JPA + ORM（如：hibernate）完成操作，这样在切换不同的ORM框架时提供了极大的方便，同时也使数据库层操作更加简单，方便解耦。

## Spring Data JPA 与 JPA和hibernate之间的关系：
　　JPA是一套规范，内部是有接口和抽象类组成的。hibernate是一套成熟的ORM框架，而且Hibernate实现了JPA规范，所以也可以称hibernate为JPA的一种实现方式，我们使用JPA的API编程，意味着站在更高的角度上看待问题（面向接口编程）。Spring Data JPA是Spring提供的一套对JPA操作更加高级的封装，是在JPA规范下的专门用来进行数据持久化的解决方案。

## 2.1 核心依赖
```xml
<!-- JPA框架 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
## 2.2 配置文件
```yml
spring:
  application:
    name: node09-boot-jpa
  datasource:
    url: jdbc:mysql://localhost:3306/data_jpa?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

- show_sql: true 在控制台显示jpa生成的sql
- format_sql: true 控制台显示生成的sql的时候进行格式化
- ddl-auto: update 这种配置方式意思是没有表的时候新建表，有表的话就不会删除再新建，字段有更新的时候会自动更新表结构


**ddl-auto几种配置说明**

1. create
每次加载hibernate时都删除上一次的生成的表，然后根据bean类重新来生成新表，容易导致数据丢失，（建议首次创建时使用）。

2. create-drop
每次加载hibernate时根据bean类生成表，但是sessionFactory一关闭,表就自动删除。

3. update
第一次加载hibernate时根据bean类会自动建立起表的结构，以后加载hibernate时根据bean类自动更新表结构，即使表结构改变了但表中的行仍然存在不会删除以前的行。

4. validate
每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。

## 2.3 实体类对象

编写实体类和数据表的映射配置，创建实体类以后，使用对应的注释配置映射关系：
```java
@Entity
作用：指定当前类是实体类。
@Table
作用：指定实体类和表之间的对应关系。
属性：
name：指定数据库表的名称
@Id
作用：指定当前字段是主键。
@GeneratedValue
作用：指定主键的生成方式。。
属性：
strategy ：指定主键生成策略。
@Column
作用：指定实体类属性和数据库表之间的对应关系
属性：
name：指定数据库表的列名称。
unique：是否唯一
nullable：是否可以为空
inserttable：是否可以插入
updateable：是否可以更新
columnDefinition: 定义建表时创建此列的DDL
secondaryTable: 从表名。如果此列不建在主表上（默认建在主表），该属性定义该列所在从表的名字搭建开发环境[重点]
```

### 2.3.1 spring data jpa创建有表注释，字段注释的数据库表
```java
@Entity
@Table(name = "user")
@org.hibernate.annotations.Table(appliesTo = "user",comment = "用户表")
public class User{

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = true,columnDefinition = "varchar(100) default '' comment '用户名'")
    private String name;

    @Column(nullable = true,columnDefinition = "varchar(100) default '' comment '邮箱'")
    private String email;

    @Column(columnDefinition = "int(11) default 0 comment '性别 0:男 1:女'")
    private int sex;

}
```

## 2.4 JPA框架的用法
定义对象的操作的接口，继承JpaRepository核心接口。

```java
import com.boot.jpa.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;
@Repository
public interface UserRepository extends JpaRepository<User,Integer> {

    // 但条件查询
    User findByAge(Integer age);
    // 多条件查询
    User findByNameAndAge(String name, Integer age);
    // 自定义查询
    @Query("from User u where u.name=:name")
    User findSql(@Param("name") String name);
}
```

编写符合Spring Data JPA规范的Dao层接口，继承JpaRepository<T,ID>和JpaSpecificationExecutor<T>

```java
 *     JpaRepository<操作的实体类类型，实体类中主键属性的类型>
 *              *封装了基本CURD操作
 *     JpaSpecificationExecutor<操作的实体类类型>
 *              *封装了复杂查询（分页）
```

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import top.biyenanhai.domain.Customer;

import java.util.List;

/**
 * Created with IntelliJ IDEA.
 *
 * @Auther: 毕业男孩
 *
 * 符合SpringDataJpa的dao接口规范
 *      JpaRepository<操作的实体类类型，实体类中主键属性的类型>
 *              *封装了基本CURD操作
 *      JpaSpecificationExecutor<操作的实体类类型>
 *              *封装了复杂查询（分页）
 *
 */
public interface CustomerDao extends JpaRepository<Customer, Long>, JpaSpecificationExecutor<Customer> {

    /**
     * 案例：根据客户名称查询客户
     *      使用jpql的形式查询
     *
     *  jpql:from Customer where custName = ?
     *
     *  配置jpql语句，使用@Query注解
     */
    @Query(value="from Customer where custName = ? ")
    Customer findJpql(String custName);

    /**
     * 案例：根据客户名称和客户id查询客户
     *      jqpl:from Customer where cutName = ？ and custId = ?
     *
     *   对于多个占位符参数
     *          赋值的时候，默认的情况下，占位符的位置需要和方法参数中的位置保持一致
     *   可以指定占位符参数的位置
     *          ？索引的方式，指定此占位的取值来源
     */
    @Query(value = "from Customer where custName=?2 and custId=?1")
    Customer findCustNameAndCustId(Long id,String name);


    /**
     * 使用jpql完成更新操作
     *      案例：根据id更新，客户的名称
     *          更新4号客户的名称，将名称改为“老男孩”
     *
     *
     * sql:update cst_customer set cust_name = ?where cust_id=?
     * jpql:update Customer set custName=? where custId=?
     *
     * @Query:代表的是进行查询
     *      声明此方法是用来更新操作
     * @Modifying：当前执行的是一个更新操作
     */
    @Query(value = "update Customer set custName=?2 where custId=?1")
    @Modifying
    void updateCustomer(long id, String custName);


    /**
     * 使用sql的形式查询：
     *      查询全部的客户
     *      sql:select * from cst_custimer;
     *      Query:配置sql查询
     *          value: sql语句
     *          nativeQuery: 查询方式
     *              true：sql查询
     *              false：jpql查询
     */
//    @Query(value = "select * from cst_customer", nativeQuery = true)  //查询全部

    @Query(value = "select * from cst_customer where cust_name like ?1",nativeQuery = true) //条件查询
    List<Object[]> findSql(String name);


    /**
     * 方法名的约定：
     *      findBy：查询
     *          对象中的属性名（首字母大写）：查询的条件
     *          CustName
     *          *默认情况：使用 等于的方式查询
     *              特殊的查询方式
     * findByCustName --  根据客户名称查询
     *
     * 在springdataJpa的运行阶段
     *      会根据方法名称进行解析  findBy  from xxx(实体类)
     *                                  属性名称    where custName =
     *
     *
     *      1、findBy + 属性名称（根据属性名称进行完成匹配的查询=）
     *      2、findBy + 属性名称 + “查询方式（Like|isnull）”
     *      3、多条件查询
     *          findBy + 属性名 + "查询条件" + "多条件的连接符（and|or）" + 属性名 + “查询方式”
     *
     */
    Customer findByCustName(String custName);

    List<Customer> findByCustNameLike(String name);

    List<Customer> findByCustNameLikeAndCustIndustry(String name, String industry);

}
```


5、封装一个服务层逻辑
```java

import com.boot.jpa.entity.User;
import com.boot.jpa.repository.UserRepository;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;
@Service
public class UserService {
    @Resource
    private UserRepository userRepository ;
    // 保存
    public void addUser (User user){
        userRepository.save(user) ;
    }
    // 根据年龄查询
    public User findByAge (Integer age){
        return userRepository.findByAge(age) ;
    }
    // 多条件查询
    public User findByNameAndAge (String name, Integer age){
        return userRepository.findByNameAndAge(name,age) ;
    }
    // 自定义SQL查询
    public User findSql (String name){
        return userRepository.findSql(name) ;
    }
    // 根据ID修改
    public void update (User user){
        userRepository.save(user) ;
    }
    //根据id删除一条数据
    public void deleteStudentById(Integer id){
        userRepository.deleteById(id);
    }
}
```

## 2.5 测试代码块

```java
import com.boot.jpa.JpaApplication;
import com.boot.jpa.entity.User;
import com.boot.jpa.service.UserService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import javax.annotation.Resource;
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = JpaApplication.class)
public class UserJpaTest {
    @Resource
    private UserService userService ;
    @Test
    public void addUser (){
        User user = new User() ;
        user.setName("知了一笑");
        user.setAge(22);
        userService.addUser(user);
        User user1 = new User() ;
        user1.setName("cicada");
        user1.setAge(23);
        userService.addUser(user1);
    }
    @Test
    public void findByAge (){
        Integer age = 22 ;
        // User{id=3, name='知了一笑', age=22}
        System.out.println(userService.findByAge(age));
    }
    @Test
    public void findByNameAndAge (){
        System.out.println(userService.findByNameAndAge("cicada",23));
    }
    @Test
    public void findSql (){
        // User{id=4, name='cicada', age=23}
        System.out.println(userService.findSql("cicada"));
    }
    @Test
    public void update (){
        User user = new User() ;
        // 如果这个主键不存在，会以主键自增的方式新增入库
        user.setId(3);
        user.setName("哈哈一笑");
        user.setAge(25);
        userService.update(user) ;
    }
    @Test
    public void deleteStudentById (){
        userService.deleteStudentById(5) ;
    }
}
```

# 3、Specifications动态查询：

　　有时我们在查询某个实体的时候，给定的条件是不固定的，这时就需要动态构建相应的查询语句，在Spring Data JPA中可以通过JpaSpecificationExecutor接口查询。相比JPQL,其优势是类型安全,更加的面向对象。对于JpaSpecificationExecutor，这个接口基本是围绕着Specification接口来定义的。我们可以简单的理解为，Specification构造的就是查询条件。

```java
/**
*    root    ：Root接口，代表查询的根对象，可以通过root获取实体中的属性
*    query    ：代表一个顶层查询对象，用来自定义查询
*    cb        ：用来构建查询，此对象里有很多条件方法
**/
public Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb);
```

案例：使用Specifications完成条件查询
```java
//依赖注入customerDao
    @Autowired
    private CustomerDao customerDao;    
    @Test
    public void testSpecifications() {
          //使用匿名内部类的方式，创建一个Specification的实现类，并实现toPredicate方法
        Specification <Customer> spec = new Specification<Customer>() {
            public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                //cb:构建查询，添加查询方式   like：模糊匹配
                //root：从实体Customer对象中按照custName属性进行查询
                return cb.like(root.get("custName").as(String.class), "航天航空%");
            }
        };
        Customer customer = customerDao.findOne(spec);
        System.out.println(customer);
    }
```

案例：基于Specifications的分页查询

```java
@Test
    public void testPage() {
        //构造查询条件
        Specification<Customer> spec = new Specification<Customer>() {
            public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                return cb.like(root.get("custName").as(String.class), "航天%");
            }
        };

        /**
         * 构造分页参数
         *         Pageable : 接口
         *             PageRequest实现了Pageable接口，调用构造方法的形式构造
         *                 第一个参数：页码（从0开始）
         *                 第二个参数：每页查询条数
         */
        Pageable pageable = new PageRequest(0, 5);

        /**
         * 分页查询，封装为Spring Data Jpa 内部的page bean
         *         此重载的findAll方法为分页方法需要两个参数
         *             第一个参数：查询条件Specification
         *             第二个参数：分页参数
         */
        Page<Customer> page = customerDao.findAll(spec,pageable);

    }
```

对于Spring Data JPA中的分页查询，是其内部自动实现的封装过程，返回的是一个Spring Data JPA提供的pageBean对象。其中的方法说明如下：
```java
//获取总页数
int getTotalPages();
//获取总记录数
long getTotalElements();
//获取列表数据
List<T> getContent();
```

# 4、JPA中的一对多
实体类（一对多，一的实体类）
```java
/**
 * 客户的实体类
 * 明确使用的注解都是JPA规范的
 * 所以导包都要导入javax.persistence包下的
 */
@Entity//表示当前类是一个实体类
@Table(name="cst_customer")//建立当前实体类和表之间的对应关系
public class Customer implements Serializable {

    @Id//表明当前私有属性是主键
    @GeneratedValue(strategy=GenerationType.IDENTITY)//指定主键的生成策略
    @Column(name="cust_id")//指定和数据库表中的cust_id列对应
    private Long custId;
    @Column(name="cust_name")//指定和数据库表中的cust_name列对应
    private String custName;

    //配置客户和联系人的一对多关系
    @OneToMany(targetEntity=LinkMan.class)
    @JoinColumn(name="lkm_cust_id",referencedColumnName="cust_id")
    private Set<LinkMan> linkmans = new HashSet<LinkMan>();...get,set方法
```

实体类（一对多，多的实体类）
```java
/**
 * 联系人的实体类（数据模型）
 */
@Entity
@Table(name="cst_linkman")
public class LinkMan implements Serializable {
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    @Column(name="lkm_id")
    private Long lkmId;
    @Column(name="lkm_name")
    private String lkmName;
    @Column(name="lkm_gender")
    private String lkmGender;
    @Column(name="lkm_phone")
    private String lkmPhone;

    //多对一关系映射：多个联系人对应客户
    @ManyToOne(targetEntity=Customer.class)
    @JoinColumn(name="lkm_cust_id",referencedColumnName="cust_id")
    private Customer customer;//用它的主键，对应联系人表中的外键...get,set方法
```

# 5、JPA中的多对多

```java
/**
 * 用户的数据模型
 */
@Entity
@Table(name="sys_user")
public class SysUser implements Serializable {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    @Column(name="user_id")
    private Long userId;
    @Column(name="user_name")
    private String userName;

    //多对多关系映射
    @ManyToMany(mappedBy="users")
    private Set<SysRole> roles = new HashSet<SysRole>();
```

```java
/**
 * 角色的数据模型
 */
@Entity
@Table(name="sys_role")
public class SysRole implements Serializable {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    @Column(name="role_id")
    private Long roleId;
    @Column(name="role_name")
    private String roleName;

    //多对多关系映射
    @ManyToMany
    @JoinTable(name="user_role_rel",//中间表的名称
              //中间表user_role_rel字段关联sys_role表的主键字段role_id
              joinColumns={@JoinColumn(name="role_id",referencedColumnName="role_id")},
              //中间表user_role_rel的字段关联sys_user表的主键user_id
              inverseJoinColumns={@JoinColumn(name="user_id",referencedColumnName="user_id")}
    )
    private Set<SysUser> users = new HashSet<SysUser>();
```

映射的注解说明
```java
@OneToMany:
       作用：建立一对多的关系映射
    属性：
        targetEntityClass：指定多的多方的类的字节码
        mappedBy：指定从表实体类中引用主表对象的名称。
        cascade：指定要使用的级联操作
        fetch：指定是否采用延迟加载

@ManyToOne
    作用：建立多对一的关系
    属性：
        targetEntityClass：指定一的一方实体类字节码
        cascade：指定要使用的级联操作
        fetch：指定是否采用延迟加载
        optional：关联是否可选。如果设置为false，则必须始终存在非空关系。

@JoinColumn
     作用：用于定义主键字段和外键字段的对应关系。
     属性：
        name：指定外键字段的名称
        referencedColumnName：指定引用主表的主键字段名称
        unique：是否唯一。默认值不唯一
        nullable：是否允许为空。默认值允许。
        insertable：是否允许插入。默认值允许。
        updatable：是否允许更新。默认值允许。
        columnDefinition：列的定义信息。
```

# 6、Spring Data JPA中的多表查询
对象导航查询：对象导航检索方式是根据已经加载的对象，导航到他的关联对象。它利用类与类之间的关系来检索对象。例如：我们通过ID查询方式查出一个客户，可以调用Customer类中的getLinkMans()方法来获取该客户的所有联系人。对象导航查询的使用要求是：两个对象之间必须存在关联关系。
```java
//查询一个客户，获取该客户下的所有联系人
@Autowired
private CustomerDao customerDao;

@Test
//由于是在java代码中测试，为了解决no session问题，将操作配置到同一个事务中
@Transactional 
public void testFind() {
    Customer customer = customerDao.findOne(5l);
    Set<LinkMan> linkMans = customer.getLinkMans();//对象导航查询
    for(LinkMan linkMan : linkMans) {
          System.out.println(linkMan);
    }
}
```

```java
//查询一个联系人，获取该联系人的所有客户
@Autowired
private LinkManDao linkManDao;

@Test
public void testFind() {
    LinkMan linkMan = linkManDao.findOne(4l);
    Customer customer = linkMan.getCustomer(); //对象导航查询
    System.out.println(customer);
}
```

采用延迟加载的思想。通过配置的方式来设定当我们在需要使用时，发起真正的查询。

配置方式：
```java
/**
 * 在客户对象的@OneToMany注解中添加fetch属性
 *     FetchType.EAGER    ：立即加载
 *     FetchType.LAZY            ：延迟加载
 */
 @OneToMany(mappedBy="customer",fetch=FetchType.EAGER)
 private Set<LinkMan> linkMans = new HashSet<>();
```

```java
/**
 * 在联系人对象的@ManyToOne注解中添加fetch属性
 *         FetchType.EAGER    ：立即加载
 *         FetchType.LAZY    ：延迟加载
 */
 @ManyToOne(targetEntity=Customer.class,fetch=FetchType.EAGER)
 @JoinColumn(name="cst_lkm_id",referencedColumnName="cust_id")
 private Customer customer;
```
使用Specification查询
```java
/**
 * Specification的多表查询
 */
@Test
public void testFind() {
    Specification<LinkMan> spec = new Specification<LinkMan>() {
        public Predicate toPredicate(Root<LinkMan> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
            //Join代表链接查询，通过root对象获取
            //创建的过程中，第一个参数为关联对象的属性名称，第二个参数为连接查询的方式（left，inner，right）
            //JoinType.LEFT : 左外连接,JoinType.INNER：内连接,JoinType.RIGHT：右外连接
            Join<LinkMan, Customer> join = root.join("customer",JoinType.INNER);
            return cb.like(join.get("custName").as(String.class),"航空航天");
        }
    };
    List<LinkMan> list = linkManDao.findAll(spec);
    for (LinkMan linkMan : list) {
        System.out.println(linkMan);
    }
}
```