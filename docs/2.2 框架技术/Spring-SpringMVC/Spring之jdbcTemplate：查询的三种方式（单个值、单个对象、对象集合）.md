[TOC]

[星瑞:Spring之jdbcTemplate：查询的三种方式（单个值、单个对象、对象集合）](https://www.cnblogs.com/gongxr/p/8053010.html)


# JdbcTemplateDemo2.java
```java
package helloworld.jdbcTemplate;

import org.springframework.jdbc.core.JdbcTemplate;

import java.sql.*;
import java.util.List;

/**
 * 功能：通过JdbcTemplate实现查询操作
 * 查询结果需要自己封装(实现RowMapper接口)
 */

public class JdbcTemplateDemo2 {
// JdbcTemplate使用步骤：
// 1、导入jar包；2、设置数据库信息；3、设置数据源；4、调用jdbcTemplate对象中的方法实现操作

    public static void main(String[] args) {
        // 设置数据库信息和据源
        JdbcTemplateObject jdbcTemplateObject = new JdbcTemplateObject();
        JdbcTemplate jdbcTemplate = jdbcTemplateObject.getJdbcTemplate();

//        插入数据
//        insertData();

//        查询返回某一个值：查询表中数据总数
        queryForOne(jdbcTemplate);

//        查询返回对象
        queryForObject(jdbcTemplate);

//        查询返回list集合
        queryForList(jdbcTemplate);

//        使用JDBC底层实现查询
        queryWithJDBC();
    }

    //  插入数据
    public static void insertData() {
        JdbcTemplateObject jdbcTemplateObject = new JdbcTemplateObject();
        JdbcTemplate jdbcTemplate = jdbcTemplateObject.getJdbcTemplate();
//        调用jdbcTemplate对象中的方法实现操作
        String sql = "insert into user value(?,?,?)";
        //表结构：id（int、自增）,name(varchar 100),age(int 10)
        int rows = jdbcTemplate.update(sql, null, "Tom", 35);
        System.out.println("插入行数：" + rows);
    }

    /**
     * 查询返回某一个值：查询表中数据总数
     */
    public static void queryForOne(JdbcTemplate jdbcTemplate) {
        String sql = "select count(*) from user";
//        调用方法获得记录数
        int count = jdbcTemplate.queryForObject(sql, Integer.class);
        System.out.println("数据总数：" + count);
    }

    /**
     * 功能：查询返回单个对象
     * 步骤：新建MyRowMapper类实现RowMapper接口，重写mapRow方法，指定返回User对象
     */
    public static void queryForObject(JdbcTemplate jdbcTemplate) {
        String sql = "select * from user where name = ?";
//        新建MyRowMapper类实现RowMapper接口，重写mapRow方法，指定返回User对象
        User user = jdbcTemplate.queryForObject(sql, new MyRowMapper(), "Tom");
        System.out.println(user);
    }

    /**
     * 功能：查询返回对象集合
     * 步骤：新建MyRowMapper类实现RowMapper接口，重写mapRow方法，指定返回User对象
     */
    public static void queryForList(JdbcTemplate jdbcTemplate) {
        String sql = "select * from user";
//        第三个参数可以省略
        List<User> users = jdbcTemplate.query(sql, new MyRowMapper());
        System.out.println(users);
    }

    /**
     * 使用JDBC底层实现查询
     */
    public static void queryWithJDBC() {
        Connection conn = null;
        PreparedStatement psmt = null;
        ResultSet rs = null;
        String jdbcUrl = "jdbc:mysql://192.168.184.130:3306/gxrdb";

        try {
//            加载驱动
            Class.forName("com.mysql.jdbc.Driver");
//            创建连接
            conn = DriverManager.getConnection(jdbcUrl, "root", "root");
            String sql = "select * from user where name = ?";
//            预编译sql
            psmt = conn.prepareStatement(sql);
//            从1开始，没有就不需要
            psmt.setString(1, "Tom");
//            执行sql
            rs = psmt.executeQuery();
//            int num = psmt.executeUpdate(); //增删改，返回操作记录数

//            遍历结果集
            while (rs.next()) {
                //根据列名查询对应的值，也可以是位置序号
                String name = rs.getString("name");
                String age = rs.getString("age");
                System.out.println(name);
                System.out.println(age);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                rs.close();
                psmt.close();
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

}
```

# MyRowMapper.java
```java
package helloworld.jdbcTemplate;

import org.springframework.jdbc.core.RowMapper;

import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * 实现RowMapper接口，返回User对象
 * */
public class MyRowMapper implements RowMapper<User>{

    @Override
    public User mapRow(ResultSet resultSet, int i) throws SQLException {
//        获取结果集中的数据
        String name = resultSet.getString("name");
        String age = resultSet.getString("age");
//        把数据封装成User对象
        User user = new User();
        user.setName(name);
        user.setAge(age);
        return user;
    }
}
```

# JdbcTemplateObject.java
```java
package helloworld.jdbcTemplate;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DriverManagerDataSource;

/**
 * 功能：设置数据库信息和数据源
 *
 * JdbcTemplat使用
 * 1、导入jar包；2、设置数据库信息；3、设置数据源；4、调用jdbcTemplate对象中的方法实现操作
 */
public class JdbcTemplateObject {
    DriverManagerDataSource dataSource;
    JdbcTemplate jdbcTemplate;

    public JdbcTemplateObject() {
        //        设置数据库信息
        this.dataSource = new DriverManagerDataSource();
        this.dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        this.dataSource.setUrl("jdbc:mysql://192.168.184.130:3306/gxrdb");
        this.dataSource.setUsername("root");
        this.dataSource.setPassword("root");

//        设置数据源
        this.jdbcTemplate = new JdbcTemplate(dataSource);

    }

    public DriverManagerDataSource getDataSource() {
        return dataSource;
    }

    public void setDataSource(DriverManagerDataSource dataSource) {
        this.dataSource = dataSource;
    }

    public JdbcTemplate getJdbcTemplate() {
        return jdbcTemplate;
    }

    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
}
```

# User.java
```java
package helloworld.jdbcTemplate;

/**
 * 数据封装类
 * */
public class User {
    private String name;
    private String age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{姓名：" + name + "; 年龄：" + age + "}";
    }
}
```