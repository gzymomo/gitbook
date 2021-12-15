> MySQL在处理join查询时，遍历驱动表的记录，把驱动表的记录传递给被驱动表，然后根据join连接条件进行匹配。优化器通常会将更小的表作为驱动表，通过在驱动表上做额外的where条件过滤（Condition Filtering），能够将驱动表限制在一个更小的范围，以便优化器能够做出更优的执行计划。

#### 1. 什么是条件过滤(Condition Filtering)

如果没有使用条件过滤，join查询的驱动表预估扫描的记录数与索引条件相关，比如一个二级索引 idx_name(name)，name='abc' 的记录数有100个，那么执行计划中的预估扫描记录数就是100左右。如果此时where条件中关于驱动表有另外一个条件限制，比如age=20，满足name='abc'且age=20的记录数为10，通过条件过滤后，实际参与到join运算的驱动表记录数只有10条左右。

条件过滤有一些限制：

- 条件只能是常量
- 条件过滤中的where条件不在索引条件中

#### 2. 条件过滤在explain中的表现

在explain的输出中，rows字段表示所选择的索引访问方式预估的扫描记录数，filtered字段反映了条件过滤，filtered值是一个百分比，最大值是100，表示没有进行任何过滤，该值越小，说明过滤效果越好。

如果一个SQL的执行计划，rows为100，filtered为10%，那么最终预估的扫描记录数为 100*10%=10。

#### 3. 条件过滤案例

有两张表做join查询，employee 为雇员表，department为部门表，查询SQL如下：

```
SELECT *
  FROM employee JOIN department ON employee.dept_no = department.dept_no
  WHERE employee.first_name = 'John'
  AND employee.hire_date BETWEEN '2018-01-01' AND '2018-06-01';
```

- employee表记录总数：1024
- department表记录总数：12
- 两张表在dept_no字段上都有索引。
- employee表在first_name上有索引。
- 满足 employee.first_name = 'John' 的记录数：8
- 满足 employee.hire_date BETWEEN '2018-01-01' AND '2018-06-01' 的记录数：150
- 满足 employee.first_name = 'John' AND employee.hire_date BETWEEN '2018-01-01' AND '2018-06-01' 记录数：1

**（1）如果没有使用条件过滤，explain执行计划如下：**

```
+----+------------+--------+------------------+---------+---------+------+----------+
| id | table      | type   | possible_keys    | key     | ref     | rows | filtered |
+----+------------+--------+------------------+---------+---------+------+----------+
| 1  | employee   | ref    | name,h_date,dept | name    | const   | 8    | 100.00   |
| 1  | department | eq_ref | PRIMARY          | PRIMARY | dept_no | 1    | 100.00   |
+----+------------+--------+------------------+---------+---------+------+----------+
```

**（2）使用了条件过滤，explain执行计划如下：**

```
+----+------------+--------+------------------+---------+---------+------+----------+
| id | table      | type   | possible_keys    | key     | ref     | rows | filtered |
+----+------------+--------+------------------+---------+---------+------+----------+
| 1  | employee   | ref    | name,h_date,dept | name    | const   | 8    | 16.31    |
| 1  | department | eq_ref | PRIMARY          | PRIMARY | dept_no | 1    | 100.00   |
+----+------------+--------+------------------+---------+---------+------+----------+
```

很明显，表employee上的filtered 由 100 变为了 16.31，8 × 16.31% = 1.3，过滤效果非常好。

#### 4. 条件过滤开关

MySQL提供了参数来控制是否打开条件过滤，默认是打开的。

SET optimizer_switch = 'condition_fanout_filter=on';

打开条件过滤有时并不总是能提高性能，优化器可能会高估条件过滤的影响，个别场景下使用条件过滤反而会导致性能下降。在排查类似性能问题时，可参考以下思路：

1. join连接的字段是否有索引，如果没有索引，则应当先加上索引，以便优化器能够掌握字段值的分布情况，更准确的预估行数。
2. 表的join顺序是否合适，通过改变表的join顺序，让更小的表作为驱动表。可以考虑使用STRAIGHT_JOIN，强制优化器使用指定的表join顺序。
3. 如果不使用条件过滤，性能会更好，那么可以关闭会话级条件过滤功能。
   SET optimizer_switch = 'condition_fanout_filter=off';