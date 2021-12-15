- [Hive + HBase](https://blog.51cto.com/simplelife/2483754)



## Hive整合HBase：数据实时写Hbase,实现在Hive中用sql查询

```
以下操作的 Hive版本：2.3.6 ,HBase版本：2.0.4
```

- 在HBase中创建表：t_hbase_stu_info

  ```mysql
  create 't_hbase_stu_info','st1'
  ```

- 在Hive中创建外部表：t_hive_stu_info

  ```mysql
  create external table t_hive_stu_info
  (id int,name string,age int,sex string)
  stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
  with serdeproperties("hbase.columns.mapping"=":key,st1:name,st1:age,st1:sex")
  tblproperties("hbase.table.name"="t_hbase_stu_info");
  ```

- 在Hbase中给t_hbase_stu_info插入数据

  ```mysql
  put 't_hbase_stu_info','1001','st1:name','zs'
  put 't_hbase_stu_info','1001','st1:age','23'
  put 't_hbase_stu_info','1001','st1:sex','man'
  put 't_hbase_stu_info','1002','st1:name','ls'
  put 't_hbase_stu_info','1002','st1:age','56'
  put 't_hbase_stu_info','1002','st1:sex','woman'
  ```

- 查看Hbase中的数据

  ```mysql
  scan 't_hbase_stu_info'
  ```

  ![Hive + HBase，用HQL查询HBase](https://s4.51cto.com/images/blog/202003/31/df6437327459698a44a19d60c31b4572.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

1. 查看Hive中的数据

   ```mysql
   select * from t_hive_stu_info;
   ```

   ![Hive + HBase，用HQL查询HBase](https://s4.51cto.com/images/blog/202003/31/f604b10a7c6bd29a2ff5f3e0a39d5b16.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)