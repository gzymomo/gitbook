- [Phoenix + HBase](https://blog.51cto.com/simplelife/2483684)



## Phoenix关联HBase的操作（三种情况）

#### 情况一：Hbase已经有已存在的表了，可在Phoenix中创建对应的视图，视图只能做查询操作，不能做增删改

- hbase中已创建表且有数据，表名：phoenix

![Phoenix + HBase,让你像操作MySQL一样操作HBase](https://s4.51cto.com/images/blog/202003/31/e5fd2bae3b5a69a68733d64861624f07.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

- 在phoenix中创建对应视图

```mysql
create view "phoenix"(
pk varchar primary key,
"info"."name" varchar,
"info"."age" varchar
);
```

- 查询视图数据

![Phoenix + HBase,让你像操作MySQL一样操作HBase](https://s4.51cto.com/images/blog/202003/31/0b7c651f0644c82a180b5530375f567a.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

#### 情况二：Hbase已存在表，可在Phoenix中创建对应的表，对表的操作可读可写，意味着可以对Hbase的表进行插入查询，删除操作。

- hbase中已存在表并且有数据
  ![Phoenix + HBase,让你像操作MySQL一样操作HBase](https://s4.51cto.com/images/blog/202003/31/bd0b0bdf864c4b5389a912a14f6dcb2a.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

- 在phoenix中创建对应的表

  ```mysql
  create table "t_person"(
  id VARCHAR PRIMARY KEY,
  "f"."id" VARCHAR,
  "f"."name" VARCHAR,
  "f"."age" VARCHAR
  ) column_encoded_bytes=0;
  ```

- 在phoenix中查询数据
  ![Phoenix + HBase,让你像操作MySQL一样操作HBase](https://s4.51cto.com/images/blog/202003/31/9656f15e1b72ce23baf2f8186735ea51.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

- 在phoenix中插入数据
  ![Phoenix + HBase,让你像操作MySQL一样操作HBase](https://s4.51cto.com/images/blog/202003/31/d8cf0a8e23ee6653211ef7378a0843fa.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

此时查看hbase中对应的t_person表数据
![Phoenix + HBase,让你像操作MySQL一样操作HBase](https://s4.51cto.com/images/blog/202003/31/bc3c252735d440787f6889144cb09447.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

#### 情况三：Hbase没有表；可直接在Phoeinx中创建表，对应的Habse中也会自动建表，在Phoenix中对表操作就是直接对Hbase中的表进行操作。

- 在phoenix中直接建表，t_test表会在hbase中同步创建
  ![Phoenix + HBase,让你像操作MySQL一样操作HBase](https://s4.51cto.com/images/blog/202003/31/04fa13910c50ca69ffa45387e5fbb9c3.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

- 插入数据看phoenix和hbase中的效果
  ![Phoenix + HBase,让你像操作MySQL一样操作HBase](https://s4.51cto.com/images/blog/202003/31/96b75a6520df265bca5401985a1a7e68.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

  ![Phoenix + HBase,让你像操作MySQL一样操作HBase](https://s4.51cto.com/images/blog/202003/31/caf1a5c9c3b8af8a1c4a96a001776284.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

- 通过Java客户端操作phoenix

```java
package com.fwmagic.hbase;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.sql.*;

/**
 * 通过Phoenix操作Hbase
 */
public class PhoenixQueryHbase {
    Connection connection = null;

    PreparedStatement ps = null;

    ResultSet rs = null;

    @Before
    public void init() throws Exception {
        connection = DriverManager.getConnection("jdbc:phoenix:hd1,hd2,hd3:2181");
    }

    /**
     * 建表并查询
     *
     * @throws Exception
     */
    @Test
    public void create() throws Exception {
        Statement statement = connection.createStatement();
        statement.executeUpdate("create  table test(id integer primary key ,animal varchar )");

        //新增和更新都是一个操作：upsert
        statement.executeUpdate("upsert into test values (1,'dog')");
        statement.executeUpdate("upsert into test values (2,'cat')");
        connection.commit();

        PreparedStatement preparedStatement = connection.prepareStatement("select * from  test");
        rs = preparedStatement.executeQuery();
        while (rs.next()) {
            String id = rs.getString("id");
            String animal = rs.getString("animal");
            String format = String.format("id:%s,animal:%s", id, animal);
            System.out.println(format);
        }
    }

    /**
     * 查询已有的表
     *
     * @throws Exception
     */
    @Test
    public void testQuery() throws Exception {
        String sql = "select * from tc";
        try {
            ps = connection.prepareStatement(sql);
            rs = ps.executeQuery();
            while (rs.next()) {
                String id = rs.getString("ID");
                String name = rs.getString("NAME");
                String age = rs.getString("AGE");
                String sex = rs.getString("SEX");
                String format = String.format("id:%s,name:%s,age:%s,sex:%s", id, name, age, sex);
                System.out.println(format);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            if (rs != null) rs.close();
            if (ps != null) ps.close();
            if (connection != null) connection.close();
        }
    }

    /**
     * 删除数据
     *
     * @throws Exception
     */
    @Test
    public void delete() throws Exception {
        try {
            ps = connection.prepareStatement("delete from test where id=2");
            ps.executeUpdate();
            connection.commit();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            if (rs != null) rs.close();
            if (ps != null) ps.close();
            if (connection != null) connection.close();
        }
    }

    /**
     * 删除表
     *
     * @throws Exception
     */
    @Test
    public void dropTable() throws Exception {
        try {
            ps = connection.prepareStatement("drop table test");
            ps.execute();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            if (rs != null) rs.close();
            if (ps != null) ps.close();
            if (connection != null) connection.close();
        }
    }

    @After
    public void close() throws Exception {
        if (rs != null) rs.close();
        if (ps != null) ps.close();
        if (connection != null) connection.close();

    }
}
```