# MYSQL 02

## 1.1.1 DQL  

```
MariaDB [(none)]> select 2*3;
+-----+
| 2*3 |
+-----+
|   6 |
+-----+
1 row in set (0.00 sec)

MariaDB [(none)]> select "hello";
+-------+
| hello |
+-------+
| hello |
+-------+
1 row in set (0.00 sec)

```

###    1.1.2 查询表中stuid name age

```
MariaDB [hellodb]> select stuid,name,age from students;
+-------+---------------+-----+

```

### 1.1.3 使用别名

```
MariaDB [hellodb]> select stuid,name as 姓名,age as 年龄 from students;
+-------+---------------+--------+
| stuid | 姓名          | 年龄   |
+-------+---------------+--------+
|     1 | Shi Zhongyu   |     22 |
|     2 | Shi Potian    |     22 |
|     3 | Xie Yanke     |     53 |
|     4 | Ding Dian     |     32 |
|     5 | Yu Yutong     |     26 |
|     6 | Shi Qing      |     46 |
|     7 | Xi Ren        |     19 |
|     8 | Lin Daiyu     |     17 |
|     9 | Ren Yingying  |     20 |
|    10 | Yue Lingshan  |     19 |

```

### 1.1.4 查询大于等于20的studi

```
MariaDB [hellodb]> select * from students where stuid >=20;
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|    20 | Diao Chan     |  19 | F      |       7 |      NULL |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
|    22 | Xiao Qiao     |  20 | F      |       1 |      NULL |
|    23 | Ma Chao       |  23 | M      |       4 |      NULL |
|    24 | Xu Xian       |  27 | M      |    NULL |      NULL |
|    25 | Sun Dasheng   | 100 | M      |    NULL |      NULL |
+-------+---------------+-----+--------+---------+-----------+
6 rows in set (0.00 sec)

```

### 1.1.5 查询stuid等于20的

```
MariaDB [hellodb]> select * from students where stuid=20;
+-------+-----------+-----+--------+---------+-----------+
| StuID | Name      | Age | Gender | ClassID | TeacherID |
+-------+-----------+-----+--------+---------+-----------+
|    20 | Diao Chan |  19 | F      |       7 |      NULL |
+-------+-----------+-----+--------+---------+-----------+
1 row in set (0.00 sec)

```

### 1.1.6 查询stuid不等于

```
MariaDB [hellodb]> select * from students where stuid<>20;
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu   |  22 | M      |       2 |         3 |
|     2 | Shi Potian    |  22 | M      |       1 |         7 |
|     3 | Xie Yanke     |  53 | M      |       2 |        16 |
|     4 | Ding Dian     |  32 | M      |       4 |         4 |
|     5 | Yu Yutong     |  26 | M      |       3 |         1 |
|     6 | Shi Qing      |  46 | M      |       5 |      NULL |
|     7 | Xi Ren        |  19 | F      |       3 |      NULL |
|     8 | Lin Daiyu     |  17 | F      |       7 |      NULL |
|     9 | Ren Yingying  |  20 | F      |       6 |      NULL |
|    10 | Yue Lingshan  |  19 | F      |       3 |      NULL |
|    11 | Yuan Chengzhi |  23 | M      |       6 |      NULL |
|    12 | Wen Qingqing  |  19 | F      |       1 |      NULL |
|    13 | Tian Boguang  |  33 | M      |       2 |      NULL |
|    14 | Lu Wushuang   |  17 | F      |       3 |      NULL |
|    15 | Duan Yu       |  19 | M      |       4 |      NULL |
|    16 | Xu Zhu        |  21 | M      |       1 |      NULL |
|    17 | Lin Chong     |  25 | M      |       4 |      NULL |
|    18 | Hua Rong      |  23 | M      |       7 |      NULL |
|    19 | Xue Baochai   |  18 | F      |       6 |      NULL |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
|    22 | Xiao Qiao     |  20 | F      |       1 |      NULL |
|    23 | Ma Chao       |  23 | M      |       4 |      NULL |
|    24 | Xu Xian       |  27 | M      |    NULL |      NULL |
|    25 | Sun Dasheng   | 100 | M      |    NULL |      NULL |
+-------+---------------+-----+--------+---------+-----------+
24 rows in set (0.00 sec)


```

```

MariaDB [hellodb]> select * from students where stuid!=20;
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu   |  22 | M      |       2 |         3 |
|     2 | Shi Potian    |  22 | M      |       1 |         7 |
|     3 | Xie Yanke     |  53 | M      |       2 |        16 |
|     4 | Ding Dian     |  32 | M      |       4 |         4 |
|     5 | Yu Yutong     |  26 | M      |       3 |         1 |
|     6 | Shi Qing      |  46 | M      |       5 |      NULL |
|     7 | Xi Ren        |  19 | F      |       3 |      NULL |
|     8 | Lin Daiyu     |  17 | F      |       7 |      NULL |
|     9 | Ren Yingying  |  20 | F      |       6 |      NULL |
|    10 | Yue Lingshan  |  19 | F      |       3 |      NULL |
|    11 | Yuan Chengzhi |  23 | M      |       6 |      NULL |
|    12 | Wen Qingqing  |  19 | F      |       1 |      NULL |
|    13 | Tian Boguang  |  33 | M      |       2 |      NULL |
|    14 | Lu Wushuang   |  17 | F      |       3 |      NULL |
|    15 | Duan Yu       |  19 | M      |       4 |      NULL |
|    16 | Xu Zhu        |  21 | M      |       1 |      NULL |
|    17 | Lin Chong     |  25 | M      |       4 |      NULL |
|    18 | Hua Rong      |  23 | M      |       7 |      NULL |
|    19 | Xue Baochai   |  18 | F      |       6 |      NULL |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
|    22 | Xiao Qiao     |  20 | F      |       1 |      NULL |
|    23 | Ma Chao       |  23 | M      |       4 |      NULL |
|    24 | Xu Xian       |  27 | M      |    NULL |      NULL |
|    25 | Sun Dasheng   | 100 | M      |    NULL |      NULL |
+-------+---------------+-----+--------+---------+-----------+
24 rows in set (0.00 sec)

```

### 1.1.7 查询Xuz Zhu

```
MariaDB [hellodb]> select * from students where name='Xu Zhu';
+-------+--------+-----+--------+---------+-----------+
| StuID | Name   | Age | Gender | ClassID | TeacherID |
+-------+--------+-----+--------+---------+-----------+
|    16 | Xu Zhu |  21 | M      |       1 |      NULL |
+-------+--------+-----+--------+---------+-----------+
1 row in set (0.00 sec)

```

### 1.1.8 查询年龄大于20并且女性

```
MariaDB [hellodb]> select * from students where age >20 and gender='F';
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
+-------+---------------+-----+--------+---------+-----------+
1 row in set (0.00 sec)


```

## 2.1.1 创建一个新表

```
MariaDB [hellodb]> create table user(id int,username char(30),password char(30));
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> insert user values (1,'admin','magedu');
Query OK, 1 row affected (0.00 sec)

MariaDB [hellodb]> insert user values (2,'mage','magedu');
Query OK, 1 row affected (0.00 sec)

MariaDB [hellodb]> insert user values (3,'wang','centos');
Query OK, 1 row affected (0.00 sec)


MariaDB [hellodb]> select * from user;
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    1 | admin    | magedu   |
|    2 | mage     | magedu   |
|    3 | wang     | centos   |
+------+----------+----------+
3 rows in set (0.00 sec)


```

### 2.1.2 SQL 注入

```
MariaDB [hellodb]> select * from user where username='admin' and password='magedu';
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    1 | admin    | magedu   |
+------+----------+----------+
1 row in set (0.00 sec)

MariaDB [hellodb]> select * from user where username='admin' and password='centos';
Empty set (0.00 sec)

MariaDB [hellodb]> select * from user where username='admin' and password='' or '1'='1';
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    1 | admin    | magedu   |
|    2 | mage     | magedu   |
|    3 | wang     | centos   |
+------+----------+----------+
3 rows in set (0.00 sec)

MariaDB [hellodb]> select * from user where username='admin'--' and password=''';
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    1 | admin    | magedu   |
|    2 | mage     | magedu   |
|    3 | wang     | centos   |
+------+----------+----------+
3 rows in set, 5 warnings (0.00 sec)



```

### 2.1.3 查找20和30的之间的年龄

```
MariaDB [hellodb]> select * from students where age >=20 and age <=30;
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu   |  22 | M      |       2 |         3 |
|     2 | Shi Potian    |  22 | M      |       1 |         7 |
|     5 | Yu Yutong     |  26 | M      |       3 |         1 |
|     9 | Ren Yingying  |  20 | F      |       6 |      NULL |
|    11 | Yuan Chengzhi |  23 | M      |       6 |      NULL |
|    16 | Xu Zhu        |  21 | M      |       1 |      NULL |
|    17 | Lin Chong     |  25 | M      |       4 |      NULL |
|    18 | Hua Rong      |  23 | M      |       7 |      NULL |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
|    22 | Xiao Qiao     |  20 | F      |       1 |      NULL |
|    23 | Ma Chao       |  23 | M      |       4 |      NULL |
|    24 | Xu Xian       |  27 | M      |    NULL |      NULL |
+-------+---------------+-----+--------+---------+-----------+
12 rows in set (0.00 sec)

MariaDB [hellodb]> select * from students where age between 20 and 30;
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu   |  22 | M      |       2 |         3 |
|     2 | Shi Potian    |  22 | M      |       1 |         7 |
|     5 | Yu Yutong     |  26 | M      |       3 |         1 |
|     9 | Ren Yingying  |  20 | F      |       6 |      NULL |
|    11 | Yuan Chengzhi |  23 | M      |       6 |      NULL |
|    16 | Xu Zhu        |  21 | M      |       1 |      NULL |
|    17 | Lin Chong     |  25 | M      |       4 |      NULL |
|    18 | Hua Rong      |  23 | M      |       7 |      NULL |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
|    22 | Xiao Qiao     |  20 | F      |       1 |      NULL |
|    23 | Ma Chao       |  23 | M      |       4 |      NULL |
|    24 | Xu Xian       |  27 | M      |    NULL |      NULL |
+-------+---------------+-----+--------+---------+-----------+
12 rows in set (0.00 sec)

```

### 2.1.4 查询以S开头的用户

```
MariaDB [hellodb]> select * from students where name like 's%';
+-------+-------------+-----+--------+---------+-----------+
| StuID | Name        | Age | Gender | ClassID | TeacherID |
+-------+-------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu |  22 | M      |       2 |         3 |
|     2 | Shi Potian  |  22 | M      |       1 |         7 |
|     6 | Shi Qing    |  46 | M      |       5 |      NULL |
|    25 | Sun Dasheng | 100 | M      |    NULL |      NULL |
+-------+-------------+-----+--------+---------+-----------+
4 rows in set (0.00 sec)


```

### 2.1.5 查询包含yu的用户（不推荐使用，生成环境）

```
MariaDB [hellodb]> select * from students where name like '%yu%';
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu   |  22 | M      |       2 |         3 |
|     5 | Yu Yutong     |  26 | M      |       3 |         1 |
|     8 | Lin Daiyu     |  17 | F      |       7 |      NULL |
|    10 | Yue Lingshan  |  19 | F      |       3 |      NULL |
|    11 | Yuan Chengzhi |  23 | M      |       6 |      NULL |
|    15 | Duan Yu       |  19 | M      |       4 |      NULL |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
+-------+---------------+-----+--------+---------+-----------+
7 rows in set (0.00 sec)

```

### 2.1.6 正则表达式查询 （不推荐）

```
MariaDB [hellodb]> select * from students where name rlike '^s';
+-------+-------------+-----+--------+---------+-----------+
| StuID | Name        | Age | Gender | ClassID | TeacherID |
+-------+-------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu |  22 | M      |       2 |         3 |
|     2 | Shi Potian  |  22 | M      |       1 |         7 |
|     6 | Shi Qing    |  46 | M      |       5 |      NULL |
|    25 | Sun Dasheng | 100 | M      |    NULL |      NULL |
+-------+-------------+-----+--------+---------+-----------+
4 rows in set (0.00 sec)

```

### 2.1.7 查询年龄

```
MariaDB [hellodb]> select age from students;
+-----+
| age |
+-----+
|  22 |
|  22 |
|  53 |
|  32 |
|  26 |
|  46 |
|  19 |
|  17 |
|  20 |
|  19 |
|  23 |
|  19 |
|  33 |
|  17 |
|  19 |
|  21 |
|  25 |
|  23 |
|  18 |
|  19 |
|  22 |
|  20 |
|  23 |
|  27 |
| 100 |
+-----+
25 rows in set (0.00 sec)


```

### 2.1.8 去除重复的年龄

```
MariaDB [hellodb]> select distinct age  from students;
+-----+
| age |
+-----+
|  22 |
|  53 |
|  32 |
|  26 |
|  46 |
|  19 |
|  17 |
|  20 |
|  23 |
|  33 |
|  21 |
|  25 |
|  18 |
|  27 |
| 100 |
+-----+
15 rows in set (0.00 sec)


```

### 2.1.9 查询空值

```
MariaDB [hellodb]> select * from students where classid is null;
+-------+-------------+-----+--------+---------+-----------+
| StuID | Name        | Age | Gender | ClassID | TeacherID |
+-------+-------------+-----+--------+---------+-----------+
|    24 | Xu Xian     |  27 | M      |    NULL |      NULL |
|    25 | Sun Dasheng | 100 | M      |    NULL |      NULL |
+-------+-------------+-----+--------+---------+-----------+
2 rows in set (0.00 sec)

```

### 2.2.1 不为空

```
MariaDB [hellodb]> select * from students where classid is not  null;
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu   |  22 | M      |       2 |         3 |
|     2 | Shi Potian    |  22 | M      |       1 |         7 |
|     3 | Xie Yanke     |  53 | M      |       2 |        16 |
|     4 | Ding Dian     |  32 | M      |       4 |         4 |
|     5 | Yu Yutong     |  26 | M      |       3 |         1 |
|     6 | Shi Qing      |  46 | M      |       5 |      NULL |
|     7 | Xi Ren        |  19 | F      |       3 |      NULL |
|     8 | Lin Daiyu     |  17 | F      |       7 |      NULL |
|     9 | Ren Yingying  |  20 | F      |       6 |      NULL |
|    10 | Yue Lingshan  |  19 | F      |       3 |      NULL |
|    11 | Yuan Chengzhi |  23 | M      |       6 |      NULL |
|    12 | Wen Qingqing  |  19 | F      |       1 |      NULL |
|    13 | Tian Boguang  |  33 | M      |       2 |      NULL |
|    14 | Lu Wushuang   |  17 | F      |       3 |      NULL |
|    15 | Duan Yu       |  19 | M      |       4 |      NULL |
|    16 | Xu Zhu        |  21 | M      |       1 |      NULL |
|    17 | Lin Chong     |  25 | M      |       4 |      NULL |
|    18 | Hua Rong      |  23 | M      |       7 |      NULL |
|    19 | Xue Baochai   |  18 | F      |       6 |      NULL |
|    20 | Diao Chan     |  19 | F      |       7 |      NULL |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
|    22 | Xiao Qiao     |  20 | F      |       1 |      NULL |
|    23 | Ma Chao       |  23 | M      |       4 |      NULL |
+-------+---------------+-----+--------+---------+-----------+
23 rows in set (0.00 sec)

```

### 2.2.2 count聚合函数

```
MariaDB [hellodb]> select count(stuid) as 记录数 from students;
+-----------+
| 记录数    |
+-----------+
|        25 |
+-----------+
1 row in set (0.00 sec)

MariaDB [hellodb]> select count(classid) as 记录数 from students;
+-----------+
| 记录数    |
+-----------+
|        23 |
+-----------+
1 row in set (0.00 sec)

MariaDB [hellodb]> 

```

### 2.2.3 max  min  avg

```
MariaDB [hellodb]> select max(age) as 记录数 from students;
+-----------+
| 记录数    |
+-----------+
|       100 |
+-----------+
1 row in set (0.00 sec)

MariaDB [hellodb]> select min(age) as 记录数 from students;
+-----------+
| 记录数    |
+-----------+
|        17 |
+-----------+
1 row in set (0.00 sec)

MariaDB [hellodb]> select avg(age) as 记录数 from students;
+-----------+
| 记录数    |
+-----------+
|   27.4000 |
+-----------+
1 row in set (0.00 sec)

```

### 2.2.4 分组计算男女平均年龄，select后跟字段名字或者函数

```
MariaDB [hellodb]> select gender,avg(age) as 记录数 from students group by gender;
+--------+-----------+
| gender | 记录数    |
+--------+-----------+
| F      |   19.0000 |
| M      |   33.0000 |
+--------+-----------+
2 rows in set (0.00 sec)

```

### 2.2.5 班级平均年龄

```
MariaDB [hellodb]> select classid,avg(age) from students group by classid;
+---------+----------+
| classid | avg(age) |
+---------+----------+
|    NULL |  63.5000 |
|       1 |  20.5000 |
|       2 |  36.0000 |
|       3 |  20.2500 |
|       4 |  24.7500 |
|       5 |  46.0000 |
|       6 |  20.7500 |
|       7 |  19.6667 |
+---------+----------+
8 rows in set (0.00 sec)


```

### 2.2.6  大于3班的平均年龄

```
MariaDB [hellodb]> select classid,avg(age) from students group by classid having classid > 3;
+---------+----------+
| classid | avg(age) |
+---------+----------+
|       4 |  24.7500 |
|       5 |  46.0000 |
|       6 |  20.7500 |
|       7 |  19.6667 |
+---------+----------+
4 rows in set (0.00 sec)


MariaDB [hellodb]> select classid,avg(age) from students where classid >3  group by classid; 
+---------+----------+
| classid | avg(age) |
+---------+----------+
|       4 |  24.7500 |
|       5 |  46.0000 |
|       6 |  20.7500 |
|       7 |  19.6667 |
+---------+----------+
4 rows in set (0.00 sec)

```

### 2.2.7 大于3班且年龄大于30的

```
MariaDB [hellodb]> select classid,avg(age) as avg from students where classid >3 group by classid having avg > 30;
+---------+---------+
| classid | avg     |
+---------+---------+
|       5 | 46.0000 |
+---------+---------+
1 row in set (0.00 sec)


```

### 2.2.8  各班的男女平均年龄

```
MariaDB [hellodb]> select classid,gender,avg(age) from students group by classid,gender;
+---------+--------+----------+
| classid | gender | avg(age) |
+---------+--------+----------+
|    NULL | M      |  63.5000 |
|       1 | F      |  19.5000 |
|       1 | M      |  21.5000 |
|       2 | M      |  36.0000 |
|       3 | F      |  18.3333 |
|       3 | M      |  26.0000 |
|       4 | M      |  24.7500 |
|       5 | M      |  46.0000 |
|       6 | F      |  20.0000 |
|       6 | M      |  23.0000 |
|       7 | F      |  18.0000 |
|       7 | M      |  23.0000 |
+---------+--------+----------+
12 rows in set (0.00 sec)

```

### 2.2.9 对年龄排序

```
MariaDB [hellodb]> select * from students order by age;
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|     8 | Lin Daiyu     |  17 | F      |       7 |      NULL |
|    14 | Lu Wushuang   |  17 | F      |       3 |      NULL |
|    19 | Xue Baochai   |  18 | F      |       6 |      NULL |
|    12 | Wen Qingqing  |  19 | F      |       1 |      NULL |
|    10 | Yue Lingshan  |  19 | F      |       3 |      NULL |
|     7 | Xi Ren        |  19 | F      |       3 |      NULL |
|    15 | Duan Yu       |  19 | M      |       4 |      NULL |
|    20 | Diao Chan     |  19 | F      |       7 |      NULL |
|     9 | Ren Yingying  |  20 | F      |       6 |      NULL |
|    22 | Xiao Qiao     |  20 | F      |       1 |      NULL |
|    16 | Xu Zhu        |  21 | M      |       1 |      NULL |
|     1 | Shi Zhongyu   |  22 | M      |       2 |         3 |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
|     2 | Shi Potian    |  22 | M      |       1 |         7 |
|    23 | Ma Chao       |  23 | M      |       4 |      NULL |
|    18 | Hua Rong      |  23 | M      |       7 |      NULL |
|    11 | Yuan Chengzhi |  23 | M      |       6 |      NULL |
|    17 | Lin Chong     |  25 | M      |       4 |      NULL |
|     5 | Yu Yutong     |  26 | M      |       3 |         1 |
|    24 | Xu Xian       |  27 | M      |    NULL |      NULL |
|     4 | Ding Dian     |  32 | M      |       4 |         4 |
|    13 | Tian Boguang  |  33 | M      |       2 |      NULL |
|     6 | Shi Qing      |  46 | M      |       5 |      NULL |
|     3 | Xie Yanke     |  53 | M      |       2 |        16 |
|    25 | Sun Dasheng   | 100 | M      |    NULL |      NULL |
+-------+---------------+-----+--------+---------+-----------+
25 rows in set (0.00 sec)

```

### 2.3.1 对年龄降序

```
MariaDB [hellodb]> select * from students order by age desc;
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|    25 | Sun Dasheng   | 100 | M      |    NULL |      NULL |
|     3 | Xie Yanke     |  53 | M      |       2 |        16 |
|     6 | Shi Qing      |  46 | M      |       5 |      NULL |
|    13 | Tian Boguang  |  33 | M      |       2 |      NULL |
|     4 | Ding Dian     |  32 | M      |       4 |         4 |
|    24 | Xu Xian       |  27 | M      |    NULL |      NULL |
|     5 | Yu Yutong     |  26 | M      |       3 |         1 |
|    17 | Lin Chong     |  25 | M      |       4 |      NULL |
|    23 | Ma Chao       |  23 | M      |       4 |      NULL |
|    18 | Hua Rong      |  23 | M      |       7 |      NULL |
|    11 | Yuan Chengzhi |  23 | M      |       6 |      NULL |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
|     1 | Shi Zhongyu   |  22 | M      |       2 |         3 |
|     2 | Shi Potian    |  22 | M      |       1 |         7 |
|    16 | Xu Zhu        |  21 | M      |       1 |      NULL |
|    22 | Xiao Qiao     |  20 | F      |       1 |      NULL |
|     9 | Ren Yingying  |  20 | F      |       6 |      NULL |
|    15 | Duan Yu       |  19 | M      |       4 |      NULL |
|     7 | Xi Ren        |  19 | F      |       3 |      NULL |
|    20 | Diao Chan     |  19 | F      |       7 |      NULL |
|    10 | Yue Lingshan  |  19 | F      |       3 |      NULL |
|    12 | Wen Qingqing  |  19 | F      |       1 |      NULL |
|    19 | Xue Baochai   |  18 | F      |       6 |      NULL |
|     8 | Lin Daiyu     |  17 | F      |       7 |      NULL |
|    14 | Lu Wushuang   |  17 | F      |       3 |      NULL |
+-------+---------------+-----+--------+---------+-----------+
25 rows in set (0.00 sec)


```

### 2.3.2 针对数字为空的排序

```
MariaDB [hellodb]> select * from students order by -classid desc;
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|     2 | Shi Potian    |  22 | M      |       1 |         7 |
|    22 | Xiao Qiao     |  20 | F      |       1 |      NULL |
|    16 | Xu Zhu        |  21 | M      |       1 |      NULL |
|    12 | Wen Qingqing  |  19 | F      |       1 |      NULL |
|     1 | Shi Zhongyu   |  22 | M      |       2 |         3 |
|    13 | Tian Boguang  |  33 | M      |       2 |      NULL |
|     3 | Xie Yanke     |  53 | M      |       2 |        16 |
|     7 | Xi Ren        |  19 | F      |       3 |      NULL |
|    10 | Yue Lingshan  |  19 | F      |       3 |      NULL |
|     5 | Yu Yutong     |  26 | M      |       3 |         1 |
|    14 | Lu Wushuang   |  17 | F      |       3 |      NULL |
|    23 | Ma Chao       |  23 | M      |       4 |      NULL |
|    17 | Lin Chong     |  25 | M      |       4 |      NULL |
|     4 | Ding Dian     |  32 | M      |       4 |         4 |
|    15 | Duan Yu       |  19 | M      |       4 |      NULL |
|     6 | Shi Qing      |  46 | M      |       5 |      NULL |
|     9 | Ren Yingying  |  20 | F      |       6 |      NULL |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
|    19 | Xue Baochai   |  18 | F      |       6 |      NULL |
|    11 | Yuan Chengzhi |  23 | M      |       6 |      NULL |
|    20 | Diao Chan     |  19 | F      |       7 |      NULL |
|    18 | Hua Rong      |  23 | M      |       7 |      NULL |
|     8 | Lin Daiyu     |  17 | F      |       7 |      NULL |
|    24 | Xu Xian       |  27 | M      |    NULL |      NULL |
|    25 | Sun Dasheng   | 100 | M      |    NULL |      NULL |
+-------+---------------+-----+--------+---------+-----------+
25 rows in set (0.00 sec)


```

### 2.3.3 对班级年龄总和排序

```
MariaDB [hellodb]> select classid,sum(age) from students group by classid order by classid;
+---------+----------+
| classid | sum(age) |
+---------+----------+
|    NULL |      127 |
|       1 |       82 |
|       2 |      108 |
|       3 |       81 |
|       4 |       99 |
|       5 |       46 |
|       6 |       83 |
|       7 |       59 |
+---------+----------+
8 rows in set (0.00 sec)

```

### 2.3.4 去除班级为空的

```
MariaDB [hellodb]> select classid,sum(age) from students group by classid having classid is not null order by classid ;
+---------+----------+
| classid | sum(age) |
+---------+----------+
|       1 |       82 |
|       2 |      108 |
|       3 |       81 |
|       4 |       99 |
|       5 |       46 |
|       6 |       83 |
|       7 |       59 |
+---------+----------+
7 rows in set (0.00 sec)


```

```
MariaDB [hellodb]> select classid,sum(age) from students where classid is not null group by classid order by classid;
+---------+----------+
| classid | sum(age) |
+---------+----------+
|       1 |       82 |
|       2 |      108 |
|       3 |       81 |
|       4 |       99 |
|       5 |       46 |
|       6 |       83 |
|       7 |       59 |
+---------+----------+
7 rows in set (0.00 sec)

```

看着3行

```
MariaDB [hellodb]> select classid,sum(age) from students where classid is not null group by classid order by classid limit 3;
+---------+----------+
| classid | sum(age) |
+---------+----------+
|       1 |       82 |
|       2 |      108 |
|       3 |       81 |
+---------+----------+
3 rows in set (0.01 sec)


```

跳过2，取3行

```
MariaDB [hellodb]> select classid,sum(age) from students where classid is not null group by classid order by classid limit 3;
+---------+----------+
| classid | sum(age) |
+---------+----------+
|       1 |       82 |
|       2 |      108 |
|       3 |       81 |
+---------+----------+
3 rows in set (0.01 sec)


```

### 2.3.5 查询1.3.5班的

```
MariaDB [hellodb]> select * from students where classid=1 or classid=3 or classid=5;
+-------+--------------+-----+--------+---------+-----------+
| StuID | Name         | Age | Gender | ClassID | TeacherID |
+-------+--------------+-----+--------+---------+-----------+
|     2 | Shi Potian   |  22 | M      |       1 |         7 |
|     5 | Yu Yutong    |  26 | M      |       3 |         1 |
|     6 | Shi Qing     |  46 | M      |       5 |      NULL |
|     7 | Xi Ren       |  19 | F      |       3 |      NULL |
|    10 | Yue Lingshan |  19 | F      |       3 |      NULL |
|    12 | Wen Qingqing |  19 | F      |       1 |      NULL |
|    14 | Lu Wushuang  |  17 | F      |       3 |      NULL |
|    16 | Xu Zhu       |  21 | M      |       1 |      NULL |
|    22 | Xiao Qiao    |  20 | F      |       1 |      NULL |
+-------+--------------+-----+--------+---------+-----------+
9 rows in set (0.00 sec)


```

```
MariaDB [hellodb]> select * from students where classid in (1,3,5);
+-------+--------------+-----+--------+---------+-----------+
| StuID | Name         | Age | Gender | ClassID | TeacherID |
+-------+--------------+-----+--------+---------+-----------+
|     2 | Shi Potian   |  22 | M      |       1 |         7 |
|     5 | Yu Yutong    |  26 | M      |       3 |         1 |
|     6 | Shi Qing     |  46 | M      |       5 |      NULL |
|     7 | Xi Ren       |  19 | F      |       3 |      NULL |
|    10 | Yue Lingshan |  19 | F      |       3 |      NULL |
|    12 | Wen Qingqing |  19 | F      |       1 |      NULL |
|    14 | Lu Wushuang  |  17 | F      |       3 |      NULL |
|    16 | Xu Zhu       |  21 | M      |       1 |      NULL |
|    22 | Xiao Qiao    |  20 | F      |       1 |      NULL |
+-------+--------------+-----+--------+---------+-----------+
9 rows in set (0.00 sec)

```

## 3.1.1 多张表纵向合并

```
MariaDB [hellodb]> select stuid,name,age,gender from students union select * from teachers;
+-------+---------------+-----+--------+
| stuid | name          | age | gender |
+-------+---------------+-----+--------+
|     1 | Shi Zhongyu   |  22 | M      |
|     2 | Shi Potian    |  22 | M      |
|     3 | Xie Yanke     |  53 | M      |
|     4 | Ding Dian     |  32 | M      |
|     5 | Yu Yutong     |  26 | M      |
|     6 | Shi Qing      |  46 | M      |
|     7 | Xi Ren        |  19 | F      |
|     8 | Lin Daiyu     |  17 | F      |
|     9 | Ren Yingying  |  20 | F      |
|    10 | Yue Lingshan  |  19 | F      |
|    11 | Yuan Chengzhi |  23 | M      |
|    12 | Wen Qingqing  |  19 | F      |
|    13 | Tian Boguang  |  33 | M      |
|    14 | Lu Wushuang   |  17 | F      |
|    15 | Duan Yu       |  19 | M      |
|    16 | Xu Zhu        |  21 | M      |
|    17 | Lin Chong     |  25 | M      |
|    18 | Hua Rong      |  23 | M      |
|    19 | Xue Baochai   |  18 | F      |
|    20 | Diao Chan     |  19 | F      |
|    21 | Huang Yueying |  22 | F      |
|    22 | Xiao Qiao     |  20 | F      |
|    23 | Ma Chao       |  23 | M      |
|    24 | Xu Xian       |  27 | M      |
|    25 | Sun Dasheng   | 100 | M      |
|     1 | Song Jiang    |  45 | M      |
|     2 | Zhang Sanfeng |  94 | M      |
|     3 | Miejue Shitai |  77 | F      |
|     4 | Lin Chaoying  |  93 | F      |
+-------+---------------+-----+--------+
29 rows in set (0.00 sec)

```

```
MariaDB [hellodb]> select * from teachers union select * from teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
+-----+---------------+-----+--------+
4 rows in set (0.00 sec)

MariaDB [hellodb]> select * from teachers union all select * from teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
+-----+---------------+-----+--------+
8 rows in set (0.00 sec)


```

### 3.1.2  横向合并 cross join

```
MariaDB [hellodb]> select * from students crsoss join teachers;

```

### 3.1.3 取出两张表的交际（内连接）

```
MariaDB [hellodb]> select * from students inner join teachers on students.teacherid=teachers.tid;
+-------+-------------+-----+--------+---------+-----------+-----+---------------+-----+--------+
| StuID | Name        | Age | Gender | ClassID | TeacherID | TID | Name          | Age | Gender |
+-------+-------------+-----+--------+---------+-----------+-----+---------------+-----+--------+
|     5 | Yu Yutong   |  26 | M      |       3 |         1 |   1 | Song Jiang    |  45 | M      |
|     1 | Shi Zhongyu |  22 | M      |       2 |         3 |   3 | Miejue Shitai |  77 | F      |
|     4 | Ding Dian   |  32 | M      |       4 |         4 |   4 | Lin Chaoying  |  93 | F      |
+-------+-------------+-----+--------+---------+-----------+-----+---------------+-----+--------+
3 rows in set (0.001 sec)

```

```
MariaDB [hellodb]> select s.stuid,s.name,s.age,t.tid,t.name,t.age from students as s inner join teachers as t on teacherid=tid;
+-------+-------------+-----+-----+---------------+-----+
| stuid | name        | age | tid | name          | age |
+-------+-------------+-----+-----+---------------+-----+
|     5 | Yu Yutong   |  26 |   1 | Song Jiang    |  45 |
|     1 | Shi Zhongyu |  22 |   3 | Miejue Shitai |  77 |
|     4 | Ding Dian   |  32 |   4 | Lin Chaoying  |  93 |
+-------+-------------+-----+-----+---------------+-----+
3 rows in set (0.001 sec)

```

```
MariaDB [hellodb]> select s.stuid,s.name,s.age,t.tid,t.name,t.age from students s,teachers t where s.teacherid=t.tid;
+-------+-------------+-----+-----+---------------+-----+
| stuid | name        | age | tid | name          | age |
+-------+-------------+-----+-----+---------------+-----+
|     5 | Yu Yutong   |  26 |   1 | Song Jiang    |  45 |
|     1 | Shi Zhongyu |  22 |   3 | Miejue Shitai |  77 |
|     4 | Ding Dian   |  32 |   4 | Lin Chaoying  |  93 |
+-------+-------------+-----+-----+---------------+-----+
3 rows in set (0.001 sec)

```

### 3.1.4  left outer 左外连接

```
MariaDB [hellodb]> select s.stuid,s.name,s.age,t.tid,t.name,t.age from students as s left outer join teachers as t on teacherid=tid;
+-------+---------------+-----+------+---------------+------+
| stuid | name          | age | tid  | name          | age  |
+-------+---------------+-----+------+---------------+------+
|     1 | Shi Zhongyu   |  22 |    3 | Miejue Shitai |   77 |
|     2 | Shi Potian    |  22 | NULL | NULL          | NULL |
|     3 | Xie Yanke     |  53 | NULL | NULL          | NULL |
|     4 | Ding Dian     |  32 |    4 | Lin Chaoying  |   93 |
|     5 | Yu Yutong     |  26 |    1 | Song Jiang    |   45 |
|     6 | Shi Qing      |  46 | NULL | NULL          | NULL |
|     7 | Xi Ren        |  19 | NULL | NULL          | NULL |
|     8 | Lin Daiyu     |  17 | NULL | NULL          | NULL |
|     9 | Ren Yingying  |  20 | NULL | NULL          | NULL |
|    10 | Yue Lingshan  |  19 | NULL | NULL          | NULL |
|    11 | Yuan Chengzhi |  23 | NULL | NULL          | NULL |
|    12 | Wen Qingqing  |  19 | NULL | NULL          | NULL |
|    13 | Tian Boguang  |  33 | NULL | NULL          | NULL |
|    14 | Lu Wushuang   |  17 | NULL | NULL          | NULL |
|    15 | Duan Yu       |  19 | NULL | NULL          | NULL |
|    16 | Xu Zhu        |  21 | NULL | NULL          | NULL |
|    17 | Lin Chong     |  25 | NULL | NULL          | NULL |
|    18 | Hua Rong      |  23 | NULL | NULL          | NULL |
|    19 | Xue Baochai   |  18 | NULL | NULL          | NULL |
|    20 | Diao Chan     |  19 | NULL | NULL          | NULL |
|    21 | Huang Yueying |  22 | NULL | NULL          | NULL |
|    22 | Xiao Qiao     |  20 | NULL | NULL          | NULL |
|    23 | Ma Chao       |  23 | NULL | NULL          | NULL |
|    24 | Xu Xian       |  27 | NULL | NULL          | NULL |
|    25 | Sun Dasheng   | 100 | NULL | NULL          | NULL |
+-------+---------------+-----+------+---------------+------+
25 rows in set (0.001 sec)


```

### 3.1.5 right outer   右外连接

```
MariaDB [hellodb]> select s.stuid,s.name,s.age,t.tid,t.name,t.age from students as s right outer join teachers as t on teacherid=tid;
+-------+-------------+------+-----+---------------+-----+
| stuid | name        | age  | tid | name          | age |
+-------+-------------+------+-----+---------------+-----+
|     1 | Shi Zhongyu |   22 |   3 | Miejue Shitai |  77 |
|     4 | Ding Dian   |   32 |   4 | Lin Chaoying  |  93 |
|     5 | Yu Yutong   |   26 |   1 | Song Jiang    |  45 |
|  NULL | NULL        | NULL |   2 | Zhang Sanfeng |  94 |
+-------+-------------+------+-----+---------------+-----+
4 rows in set (0.000 sec)

MariaDB [hellodb]> 

```

### 3.1.6  内连接且年龄大于30 的

```
MariaDB [hellodb]> select s.stuid,s.name,s.age,t.tid,t.name,t.age from students as s inner join teachers as t on teacherid=tid and s.age >30;
+-------+-----------+-----+-----+--------------+-----+
| stuid | name      | age | tid | name         | age |
+-------+-----------+-----+-----+--------------+-----+
|     4 | Ding Dian |  32 |   4 | Lin Chaoying |  93 |
+-------+-----------+-----+-----+--------------+-----+
1 row in set (0.000 sec)
 
```

### 3.1.7  左外连接后，过滤

```
MariaDB [hellodb]> select s.stuid,s.name,s.age,t.tid,t.name,t.age from students as s left outer join teachers as t on teacherid=tid where t.tid is null;
+-------+---------------+-----+------+------+------+
| stuid | name          | age | tid  | name | age  |
+-------+---------------+-----+------+------+------+
|     2 | Shi Potian    |  22 | NULL | NULL | NULL |
|     3 | Xie Yanke     |  53 | NULL | NULL | NULL |
|     6 | Shi Qing      |  46 | NULL | NULL | NULL |
|     7 | Xi Ren        |  19 | NULL | NULL | NULL |
|     8 | Lin Daiyu     |  17 | NULL | NULL | NULL |
|     9 | Ren Yingying  |  20 | NULL | NULL | NULL |
|    10 | Yue Lingshan  |  19 | NULL | NULL | NULL |
|    11 | Yuan Chengzhi |  23 | NULL | NULL | NULL |
|    12 | Wen Qingqing  |  19 | NULL | NULL | NULL |
|    13 | Tian Boguang  |  33 | NULL | NULL | NULL |
|    14 | Lu Wushuang   |  17 | NULL | NULL | NULL |
|    15 | Duan Yu       |  19 | NULL | NULL | NULL |
|    16 | Xu Zhu        |  21 | NULL | NULL | NULL |
|    17 | Lin Chong     |  25 | NULL | NULL | NULL |
|    18 | Hua Rong      |  23 | NULL | NULL | NULL |
|    19 | Xue Baochai   |  18 | NULL | NULL | NULL |
|    20 | Diao Chan     |  19 | NULL | NULL | NULL |
|    21 | Huang Yueying |  22 | NULL | NULL | NULL |
|    22 | Xiao Qiao     |  20 | NULL | NULL | NULL |
|    23 | Ma Chao       |  23 | NULL | NULL | NULL |
|    24 | Xu Xian       |  27 | NULL | NULL | NULL |
|    25 | Sun Dasheng   | 100 | NULL | NULL | NULL |
+-------+---------------+-----+------+------+------+
22 rows in set (0.000 sec)
 
```

###  3.1.8  完全外连接

```
MariaDB [hellodb]> select s.stuid,s.name,s.age,t.tid,t.name,t.age from students as s left outer join teachers as t on teacherid=tid
    -> union
    -> select s.stuid,s.name,s.age,t.tid,t.name,t.age from students as s right outer join teachers as t on teacherid=tid;
+-------+---------------+------+------+---------------+------+
| stuid | name          | age  | tid  | name          | age  |
+-------+---------------+------+------+---------------+------+
|     1 | Shi Zhongyu   |   22 |    3 | Miejue Shitai |   77 |
|     2 | Shi Potian    |   22 | NULL | NULL          | NULL |
|     3 | Xie Yanke     |   53 | NULL | NULL          | NULL |
|     4 | Ding Dian     |   32 |    4 | Lin Chaoying  |   93 |
|     5 | Yu Yutong     |   26 |    1 | Song Jiang    |   45 |
|     6 | Shi Qing      |   46 | NULL | NULL          | NULL |
|     7 | Xi Ren        |   19 | NULL | NULL          | NULL |
|     8 | Lin Daiyu     |   17 | NULL | NULL          | NULL |
|     9 | Ren Yingying  |   20 | NULL | NULL          | NULL |
|    10 | Yue Lingshan  |   19 | NULL | NULL          | NULL |
|    11 | Yuan Chengzhi |   23 | NULL | NULL          | NULL |
|    12 | Wen Qingqing  |   19 | NULL | NULL          | NULL |
|    13 | Tian Boguang  |   33 | NULL | NULL          | NULL |
|    14 | Lu Wushuang   |   17 | NULL | NULL          | NULL |
|    15 | Duan Yu       |   19 | NULL | NULL          | NULL |
|    16 | Xu Zhu        |   21 | NULL | NULL          | NULL |
|    17 | Lin Chong     |   25 | NULL | NULL          | NULL |
|    18 | Hua Rong      |   23 | NULL | NULL          | NULL |
|    19 | Xue Baochai   |   18 | NULL | NULL          | NULL |
|    20 | Diao Chan     |   19 | NULL | NULL          | NULL |
|    21 | Huang Yueying |   22 | NULL | NULL          | NULL |
|    22 | Xiao Qiao     |   20 | NULL | NULL          | NULL |
|    23 | Ma Chao       |   23 | NULL | NULL          | NULL |
|    24 | Xu Xian       |   27 | NULL | NULL          | NULL |
|    25 | Sun Dasheng   |  100 | NULL | NULL          | NULL |
|  NULL | NULL          | NULL |    2 | Zhang Sanfeng |   94 |
+-------+---------------+------+------+---------------+------+
26 rows in set (0.001 sec)


```

### 3.1.9 子查询

```
MariaDB [hellodb]> select * from students where age >(select avg(age) from students);
+-------+--------------+-----+--------+---------+-----------+
| StuID | Name         | Age | Gender | ClassID | TeacherID |
+-------+--------------+-----+--------+---------+-----------+
|     3 | Xie Yanke    |  53 | M      |       2 |        16 |
|     4 | Ding Dian    |  32 | M      |       4 |         4 |
|     6 | Shi Qing     |  46 | M      |       5 |      NULL |
|    13 | Tian Boguang |  33 | M      |       2 |      NULL |
|    25 | Sun Dasheng  | 100 | M      |    NULL |      NULL |
+-------+--------------+-----+--------+---------+-----------+
5 rows in set (0.004 sec)

```

```
MariaDB [hellodb]> update students set age=(select avg(age) from teachers) where stuid=25;
  
```

### 3.2.1 创建一个表

```
MariaDB [hellodb]> create table emp (id int,name char(20),leaderid int)；
MariaDB [hellodb]> insert emp value (1,'mage',null);
Query OK, 1 row affected (0.002 sec)

MariaDB [hellodb]> insert emp value (2,'zhangsir',1);
Query OK, 1 row affected (0.004 sec)

MariaDB [hellodb]> insert emp value (3,'wang',2);
Query OK, 1 row affected (0.001 sec)

MariaDB [hellodb]> insert emp value (4,'zhang',3);
Query OK, 1 row affected (0.001 sec)

MariaDB [hellodb]> select * from emp;
+------+----------+----------+
| id   | name     | leaderid |
+------+----------+----------+
|    1 | mage     |     NULL |
|    2 | zhangsir |        1 |
|    3 | wang     |        2 |
|    4 | zhang    |        3 |
+------+----------+----------+
4 rows in set (0.000 sec)


```

### 3.2.2 自联结

```
MariaDB [hellodb]> select e.name,l.name from emp as e left join emp as l on e.leaderid=l.id;
+----------+----------+
| name     | name     |
+----------+----------+
| zhangsir | mage     |
| wang     | zhangsir |
| zhang    | wang     |
| mage     | NULL     |
+----------+----------+
4 rows in set (0.001 sec)

```

### 3.2.3 三张表连接

```
MariaDB [hellodb]> select st.name,co.course,sc.score from students st inner join scores sc on st.stuid=sc.stuid inner join courses co on sc.courseid=co.courseid;
+-------------+----------------+-------+
| name        | course         | score |
+-------------+----------------+-------+
| Shi Zhongyu | Kuihua Baodian |    77 |
| Shi Zhongyu | Weituo Zhang   |    93 |
| Shi Potian  | Kuihua Baodian |    47 |
| Shi Potian  | Daiyu Zanghua  |    97 |
| Xie Yanke   | Kuihua Baodian |    88 |
| Xie Yanke   | Weituo Zhang   |    75 |
| Ding Dian   | Daiyu Zanghua  |    71 |
| Ding Dian   | Kuihua Baodian |    89 |
| Yu Yutong   | Hamo Gong      |    39 |
| Yu Yutong   | Dagou Bangfa   |    63 |
| Shi Qing    | Hamo Gong      |    96 |
| Xi Ren      | Hamo Gong      |    86 |
| Xi Ren      | Dagou Bangfa   |    83 |
| Lin Daiyu   | Taiji Quan     |    57 |
| Lin Daiyu   | Jinshe Jianfa  |    93 |
+-------------+----------------+-------+
15 rows in set (0.000 sec)


```



### 3.2.4 定义视图

```
MariaDB [hellodb]> create view view_test as  select st.name,co.course,sc.score from students st inner join scores sc on st.stuid=sc.stuid inner join courses co on sc.courseid=co.courseid;
MariaDB [hellodb]> show tables;
+-------------------+
| Tables_in_hellodb |
+-------------------+
| classes           |
| coc               |
| courses           |
| emp               |
| scores            |
| students          |
| teachers          |
| toc               |
| user              |
| view_test         |
+-------------------+
10 rows in set (0.001 sec)

MariaDB [hellodb]> select * from view_test;
+-------------+----------------+-------+
| name        | course         | score |
+-------------+----------------+-------+
| Shi Zhongyu | Kuihua Baodian |    77 |
| Shi Zhongyu | Weituo Zhang   |    93 |
| Shi Potian  | Kuihua Baodian |    47 |
| Shi Potian  | Daiyu Zanghua  |    97 |
| Xie Yanke   | Kuihua Baodian |    88 |
| Xie Yanke   | Weituo Zhang   |    75 |
| Ding Dian   | Daiyu Zanghua  |    71 |
| Ding Dian   | Kuihua Baodian |    89 |
| Yu Yutong   | Hamo Gong      |    39 |
| Yu Yutong   | Dagou Bangfa   |    63 |
| Shi Qing    | Hamo Gong      |    96 |
| Xi Ren      | Hamo Gong      |    86 |
| Xi Ren      | Dagou Bangfa   |    83 |
| Lin Daiyu   | Taiji Quan     |    57 |
| Lin Daiyu   | Jinshe Jianfa  |    93 |
+-------------+----------------+-------+
15 rows in set (0.001 sec)


```



### 3.2.5 判断是否为视图

```
MariaDB [hellodb]> show table status like 'view_test'\G
*************************** 1. row ***************************
            Name: view_test
          Engine: NULL
         Version: NULL
      Row_format: NULL
            Rows: NULL
  Avg_row_length: NULL
     Data_length: NULL
 Max_data_length: NULL
    Index_length: NULL
       Data_free: NULL
  Auto_increment: NULL
     Create_time: NULL
     Update_time: NULL
      Check_time: NULL
       Collation: NULL
        Checksum: NULL
  Create_options: NULL
         Comment: VIEW
Max_index_length: NULL
       Temporary: NULL
1 row in set (0.001 sec)


```

### 3.2.6 创建一个老学员的视图

```
MariaDB [hellodb]> create view view_oldstudents as  select * from students where age > 30;
Query OK, 0 rows affected (0.001 sec)

MariaDB [hellodb]> 
MariaDB [hellodb]> 
MariaDB [hellodb]> select * from view_oldstudents;
+-------+--------------+-----+--------+---------+-----------+
| StuID | Name         | Age | Gender | ClassID | TeacherID |
+-------+--------------+-----+--------+---------+-----------+
|     3 | Xie Yanke    |  53 | M      |       2 |        16 |
|     4 | Ding Dian    |  32 | M      |       4 |         4 |
|     6 | Shi Qing     |  46 | M      |       5 |      NULL |
|    13 | Tian Boguang |  33 | M      |       2 |      NULL |
|    25 | Sun Dasheng  |  77 | M      |    NULL |      NULL |
+-------+--------------+-----+--------+---------+-----------+
5 rows in set (0.001 sec)

MariaDB [hellodb]> 

```

```
MariaDB [hellodb]> insert view_oldstudents(name,age) values('tom',50);
Query OK, 1 row affected (0.004 sec)

MariaDB [hellodb]> insert view_oldstudents(name,age) values('alice',20);
Query OK, 1 row affected (0.001 sec)

MariaDB [hellodb]> select * from view_oldstudents;
+-------+--------------+-----+--------+---------+-----------+
| StuID | Name         | Age | Gender | ClassID | TeacherID |
+-------+--------------+-----+--------+---------+-----------+
|     3 | Xie Yanke    |  53 | M      |       2 |        16 |
|     4 | Ding Dian    |  32 | M      |       4 |         4 |
|     6 | Shi Qing     |  46 | M      |       5 |      NULL |
|    13 | Tian Boguang |  33 | M      |       2 |      NULL |
|    25 | Sun Dasheng  |  77 | M      |    NULL |      NULL |
|    26 | tom          |  50 | F      |    NULL |      NULL |
+-------+--------------+-----+--------+---------+-----------+
6 rows in set (0.001 sec)


root@shanghai:~# mysql -e "use hellodb;select * from students;" |grep alice
27	alice	20	F	NULL	NULL
root@shanghai:~# 

```

### 3.2.7 删除视图

```
MariaDB [hellodb]> drop view view_test;
Query OK, 0 rows affected (0.000 sec)
MariaDB [hellodb]> drop view view_oldstudents;
Query OK, 0 rows affected (0.000 sec)

```

### 3.2.8 创建函数

```
MariaDB [hellodb]> create function simplefun() returns varchar(20) return "hello world!";
Query OK, 0 rows affected (0.001 sec)

MariaDB [hellodb]> select simplefun();
+--------------+
| simplefun()  |
+--------------+
| hello world! |
+--------------+
1 row in set (0.000 sec)


```

### 3.2.9 查看函数列表

```
MariaDB [hellodb]> show function status\G
*************************** 1. row ***************************
                  Db: hellodb
                Name: simplefun
                Type: FUNCTION
             Definer: root@localhost
            Modified: 2020-11-16 09:20:49
             Created: 2020-11-16 09:20:49
       Security_type: DEFINER
             Comment: 
character_set_client: utf8mb4
collation_connection: utf8mb4_general_ci
  Database Collation: utf8_general_ci
1 row in set (0.001 sec)

```

### 3.3.1 查看函数定义

```
MariaDB [hellodb]> show create function simplefun\G
*************************** 1. row ***************************
            Function: simplefun
            sql_mode: STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
     Create Function: CREATE DEFINER=`root`@`localhost` FUNCTION `simplefun`() RETURNS varchar(20) CHARSET utf8
return "hello world!"
character_set_client: utf8mb4
collation_connection: utf8mb4_general_ci
  Database Collation: utf8_general_ci
1 row in set (0.000 sec)


```

### 3.3.2 自己定义函数保存的位置

```
MariaDB [hellodb]> select * from mysql.proc\G

```

### 3.3.3 删除函数

```
MariaDB [hellodb]> drop function simplefun;

```

### 3.3.4 自定义函数

```
DELIMITER //
CREATE FUNCTION deleteById(uid SMALLINT UNSIGNED) RETURNS
VARCHAR(20)
BEGIN
DELETE FROM students WHERE stuid = uid;
RETURN (SELECT COUNT(stuid) FROM students);
END//
DELIMITER ;
```

### 3.3.5 删除id为10

```
select deleteByid(10);

```

### 3.3.6 定义局部变量

```
DELIMITER //
CREATE FUNCTION addTwoNumber(x SMALLINT UNSIGNED, y SMALLINT
UNSIGNED)
RETURNS SMALLINT
BEGIN
DECLARE a, b SMALLINT UNSIGNED;
SET a = x, b = y;
RETURN a+b;
END//
DELIMITER ;
```

### 3.3.7 变量之和

```
 select addTwoNumber(10,20);
```

### 3.3.8 创建存储过程

```
delimiter //
CREATE PROCEDURE showTime()
BEGIN
SELECT now();
END//
delimiter ;
CALL showTime;
```

### 3.3.9  存储过程带有参数

```
delimiter //
CREATE PROCEDURE selectById(IN uid SMALLINT UNSIGNED)
BEGIN
SELECT * FROM students WHERE stuid = uid;
END//
delimiter ;
call selectById(2);
```

### 3.4.1 1+100存储过程

```
delimiter //
CREATE PROCEDURE dorepeat(n INT)
BEGIN
SET @i = 0;
SET @sum = 0;
REPEAT SET @sum = @sum+@i; SET @i = @i + 1;
UNTIL @i > n END REPEAT;
END//
delimiter ;
CALL dorepeat(100);
SELECT @sum;
```

### 3.4.2 查看函数中更改的条数

```
MariaDB [hellodb]> update students set classid=10 where classid is null;
Query OK, 3 rows affected (0.002 sec)
Rows matched: 3  Changed: 3  Warnings: 0

MariaDB [hellodb]> select row_count();
+-------------+
| row_count() |
+-------------+
|           3 |
+-------------+
1 row in set (0.000 sec)


```

### 3.4.3 删除Uid大于2的

```
delimiter //
CREATE PROCEDURE deleteById(IN uid SMALLINT UNSIGNED, OUT num
SMALLINT UNSIGNED)
BEGIN
DELETE FROM students WHERE stuid >= uid;
SELECT row_count() into num;
END//
delimiter ;
call deleteById(2,@Line);
SELECT @Line;
```

### 3.4.4 创建两张表

```
CREATE TABLE student_info (
stu_id INT(11) NOT NULL AUTO_INCREMENT,
stu_name VARCHAR(255) DEFAULT NULL,
PRIMARY KEY (stu_id)
);
CREATE TABLE student_count (
student_count INT(11) DEFAULT 0
);
INSERT INTO student_count VALUES(0);
```

### 3.4.5 创建触发器

```
CREATE TRIGGER trigger_student_count_insert
AFTER INSERT
ON student_info FOR EACH ROW
UPDATE student_count SET student_count=student_count+1;
CREATE TRIGGER trigger_student_count_delete
AFTER DELETE
ON student_info FOR EACH ROW
UPDATE student_count SET student_count=student_count-1;
```

### 3.4.6 插入数据

```
MariaDB [hellodb]> insert student_info values(1,'a');
MariaDB [hellodb]> insert student_info values(2,'b'),(3,'c');
Query OK, 2 rows affected (0.001 sec)
Records: 2  Duplicates: 0  Warnings: 0

MariaDB [hellodb]> select * from student_count;
+---------------+
| student_count |
+---------------+
|             3 |
+---------------+
1 row in set (0.000 sec)


MariaDB [hellodb]> delete from student_info where stu_id=3;
Query OK, 1 row affected (0.001 sec)

MariaDB [hellodb]> select * from student_count;
+---------------+
| student_count |
+---------------+
|             2 |
+---------------+
1 row in set (0.000 sec)



```

### 3.4.6 触发器存放位置

```
MariaDB [hellodb]> select * from information_schema.triggers\G

```

### 3.4.7 创建用户

```
MariaDB [mysql]> create user test@'192.168.0.%' identified by 'centos';
Query OK, 0 rows affected (0.000 sec)

MariaDB [mysql]> select user,host,password,authentication_string from user;
+------+-------------+-------------------------------------------+-----------------------+
| user | host        | password                                  | authentication_string |
+------+-------------+-------------------------------------------+-----------------------+
| root | localhost   |                                           |                       |
| test | 192.168.0.% | *128977E278358FF80A246B5046F51043A2B1FCED |                       |
+------+-------------+-------------------------------------------+-----------------------+
2 rows in set (0.000 sec)

```

```
[root@node1 ~]# mysql -utest -h192.168.0.111 -pcentos
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 6
Server version: 5.5.68-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> status;
--------------
mysql  Ver 15.1 Distrib 5.5.68-MariaDB, for Linux (x86_64) using readline 5.1

Connection id:		6
Current database:	
Current user:		test@master.magedu.com
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server:			MariaDB
Server version:		5.5.68-MariaDB MariaDB Server
Protocol version:	10
Connection:		192.168.0.111 via TCP/IP
Server characterset:	latin1
Db     characterset:	latin1
Client characterset:	utf8
Conn.  characterset:	utf8
TCP port:		3306
Uptime:			2 min 30 sec

Threads: 1  Questions: 15  Slow queries: 0  Opens: 0  Flush tables: 2  Open tables: 26  Queries per second avg: 0.100
--------------

MariaDB [(none)]> 

```

### 3.4.8 删除匿名账号

```
MariaDB [(none)]> select user,host,password from mysql.user;
+------+-------------+-------------------------------------------+
| user | host        | password                                  |
+------+-------------+-------------------------------------------+
| root | localhost   |                                           |
| root | node1       |                                           |
| root | 127.0.0.1   |                                           |
| root | ::1         |                                           |
|      | localhost   |                                           |
|      | node1       |                                           |
| test | 192.168.0.% | *128977E278358FF80A246B5046F51043A2B1FCED |
+------+-------------+-------------------------------------------+
7 rows in set (0.00 sec)

MariaDB [(none)]> drop user ''@'localhost';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> drop user ''@'node1';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> select user,host,password from mysql.user;
+------+-------------+-------------------------------------------+
| user | host        | password                                  |
+------+-------------+-------------------------------------------+
| root | localhost   |                                           |
| root | node1       |                                           |
| root | 127.0.0.1   |                                           |
| root | ::1         |                                           |
| test | 192.168.0.% | *128977E278358FF80A246B5046F51043A2B1FCED |
+------+-------------+-------------------------------------------+
5 rows in set (0.00 sec)

MariaDB [(none)]> 

```

### 3.4.9 修改密码

```
MariaDB [(none)]> set password for root@'localhost'=password('centos');
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> select user,host,password from mysql.user;
+------+-------------+-------------------------------------------+
| user | host        | password                                  |
+------+-------------+-------------------------------------------+
| root | localhost   | *128977E278358FF80A246B5046F51043A2B1FCED |
| root | node1       |                                           |
| root | 127.0.0.1   |                                           |
| root | ::1         |                                           |
| test | 192.168.0.% | *128977E278358FF80A246B5046F51043A2B1FCED |
+------+-------------+-------------------------------------------+
5 rows in set (0.00 sec)

127.0.0.1 和localhost 不是同一个账号，127.0.0.1会解析成localhost，配置文件加入
skip_name_resolve=on
```

```
MariaDB [mysql]> update user set password=password('centos') where host='node1';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [mysql]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [mysql]> select user,host,password from mysql.user;
+------+-------------+-------------------------------------------+
| user | host        | password                                  |
+------+-------------+-------------------------------------------+
| root | localhost   | *128977E278358FF80A246B5046F51043A2B1FCED |
| root | node1       | *128977E278358FF80A246B5046F51043A2B1FCED |
| root | 127.0.0.1   | *6B8CCC83799A26CD19D7AD9AEEADBCD30D8A8664 |
| root | ::1         |                                           |
| test | 192.168.0.% | *128977E278358FF80A246B5046F51043A2B1FCED |
+------+-------------+-------------------------------------------+
5 rows in set (0.00 sec)

```

### 3.5.0 跳过授权表修改密码

```
skip-grant-tables 配置文件中加入此项
skip_networking

MariaDB [(none)]> update mysql.user set password=password('wange') where user ='root';

```

### 3.5.1 授权一个远程登录的用户

```
MariaDB [hellodb]> grant all on hellodb.* to test@'192.168.0.%' identified by 'centos';
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> show grants for test@'192.168.0.%';
+---------------------------------------------------------------------------------------------------------------+
| Grants for test@192.168.0.%                                                                                   |
+---------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'test'@'192.168.0.%' IDENTIFIED BY PASSWORD '*128977E278358FF80A246B5046F51043A2B1FCED' |
| GRANT ALL PRIVILEGES ON `hellodb`.* TO 'test'@'192.168.0.%'                                                   |
+---------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)


```

### 3.5.2 取消权限

```
MariaDB [hellodb]> revoke drop  on hellodb.* from test@'192.168.0.%';
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> flush privileges;

```

### 3.5.3 查看自己当前的权限

```
MariaDB [(none)]> show grants for current_user();
+----------------------------------------------------------------------------------------------------------------------------------------+
| Grants for root@localhost                                                                                                              |
+----------------------------------------------------------------------------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY PASSWORD '*534B40DD6A3DF0B11F46536C928F2B3DD6DDDF97' WITH GRANT OPTION |
| GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION                                                                           |
+----------------------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

```

### 3.5.4 每个表单独使用一个表空间存储表的数据和索引

```
innodb_file_per_table

```

### 3.5.5 查看mysql支持的存储引擎

```

MariaDB [(none)]> show engines;

```

### 3.5.6 查看当前的默认存储引擎

```
MariaDB [(none)]> show variables like '%storage_engine%';

```

### 3.5.7 修改当前的默认存储引擎

```
default_storage_engine=MyISAM

```

### 3.5.8 查看mysql数据中表的存储引擎

```
MariaDB [(none)]> show table status from mysql\G

```

### 3.5.9 查看库中指定表的存储引擎

```
MariaDB [hellodb]> use hellodb;show table status like 'students'\G
MariaDB [hellodb]> use hellodb;show create table students\G

```

### 3.6.1 设置表的存储引擎

```
CREATE TABLE tb_name(... ) ENGINE=InnoDB;
ALTER TABLE tb_name ENGINE=InnoDB;
```

### 3.6.2 查看数据库变量

```
MariaDB [hellodb]> select @@skip_networking;
+-------------------+
| @@skip_networking |
+-------------------+
|                 0 |
+-------------------+
1 row in set (0.00 sec)

```

### 3.6.3 修改全局变量

```
MariaDB [hellodb]> set global innodb_file_per_table=off;
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> select @@innodb_file_per_table;
+-------------------------+
| @@innodb_file_per_table |
+-------------------------+
|                       0 |
+-------------------------+
1 row in set (0.00 sec)


```

### 3.6.4 列出所有的变量

```
MariaDB [hellodb]> show variables\G
MariaDB [hellodb]> show variables like 'version_compile_os';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| version_compile_os | Linux |
+--------------------+-------+
1 row in set (0.00 sec)

MariaDB [hellodb]> show variables like '%os';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| version_compile_os | Linux |
+--------------------+-------+
1 row in set (0.00 sec)


```

### 3.6.5 系统状态变量

```
MariaDB [hellodb]> show status;
MariaDB [hellodb]> show status like '%threads%';
+-------------------------+-------+
| Variable_name           | Value |
+-------------------------+-------+
| Delayed_insert_threads  | 0     |
| Slow_launch_threads     | 0     |
| Threadpool_idle_threads | 0     |
| Threadpool_threads      | 0     |
| Threads_cached          | 0     |
| Threads_connected       | 1     |
| Threads_created         | 2     |
| Threads_running         | 1     |
+-------------------------+-------+
8 rows in set (0.00 sec)

```

### 3.6.6 系统默认最大并发连接数

```
MariaDB [hellodb]> show variables like '%connection%';
+--------------------------+-----------------+
| Variable_name            | Value           |
+--------------------------+-----------------+
| character_set_connection | utf8            |
| collation_connection     | utf8_general_ci |
| extra_max_connections    | 1               |
| max_connections          | 151             |
| max_user_connections     | 0               |
+--------------------------+-----------------+
5 rows in set (0.00 sec)

```

### 3.6.7 修改最大连接数

```
MariaDB [hellodb]> set global max_connections=3000;
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> show variables like '%connection%';
+--------------------------+-----------------+
| Variable_name            | Value           |
+--------------------------+-----------------+
| character_set_connection | utf8            |
| collation_connection     | utf8_general_ci |
| extra_max_connections    | 1               |
| max_connections          | 3000            |
| max_user_connections     | 0               |
+--------------------------+-----------------+
5 rows in set (0.00 sec)


```

### 3.6. 8 查看mysqld 默认设置

```
[root@node1 ~]# /usr/libexec/mysqld --print-defaults
/usr/libexec/mysqld would have been started with the following arguments:
--skip_name_resolve --max-connections=3000 --innodb_file_per_table --datadir=/var/lib/mysql --socket=/var/lib/mysql/mysql.sock --symbolic-links=0 
[root@node1 ~]# 

```

### 3.6.9 创建一个表

```

MariaDB [hellodb]> create table t1 (id int ,name char(3));
MariaDB [hellodb]> insert t1 values(3,'abcde');
Query OK, 1 row affected, 1 warning (0.00 sec)

MariaDB [hellodb]> show warnings;
+---------+------+-------------------------------------------+
| Level   | Code | Message                                   |
+---------+------+-------------------------------------------+
| Warning | 1265 | Data truncated for column 'name' at row 1 |
+---------+------+-------------------------------------------+
1 row in set (0.00 sec)

MariaDB [hellodb]> select * from t1;
+------+------+
| id   | name |
+------+------+
|    1 | a    |
|    2 | abc  |
|    3 | abc  |
+------+------+
3 rows in set (0.00 sec)

```

### 3.7.1 设置sql_mode

```
MariaDB [hellodb]> set sql_mode='traditional';
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> show variables like 'sql_mode';
+---------------+------------------------------------------------------------------------------------------------------------------------------------------------------+
| Variable_name | Value                                                                                                                                                |
+---------------+------------------------------------------------------------------------------------------------------------------------------------------------------+
| sql_mode      | STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION |
+---------------+------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

MariaDB [hellodb]> insert t1 values(4,'abcde');
ERROR 1406 (22001): Data too long for column 'name' at row 1
MariaDB [hellodb]> 

```

### 3.7.2 设置缓存大小100M

```
query-cache-size=100M
MariaDB [(none)]> show variables like 'query%';
+------------------------------+-----------+
| Variable_name                | Value     |
+------------------------------+-----------+
| query_alloc_block_size       | 8192      |
| query_cache_limit            | 1048576   |
| query_cache_min_res_unit     | 4096      |
| query_cache_size             | 104857600 |
| query_cache_strip_comments   | OFF       |
| query_cache_type             | ON        |
| query_cache_wlock_invalidate | OFF       |
| query_prealloc_size          | 8192      |
+------------------------------+-----------+
8 rows in set (0.00 sec)


```

### 3.7.3 状态的缓存变量

```
MariaDB [(none)]> show global status like 'Qcache%';
+-------------------------+-----------+
| Variable_name           | Value     |
+-------------------------+-----------+
| Qcache_free_blocks      | 1         |
| Qcache_free_memory      | 104839808 |
| Qcache_hits             | 0         |
| Qcache_inserts          | 0         |
| Qcache_lowmem_prunes    | 0         |
| Qcache_not_cached       | 1         |
| Qcache_queries_in_cache | 0         |
| Qcache_total_blocks     | 1         |
+-------------------------+-----------+
8 rows in set (0.00 sec)

```

### 3.7.4 查看命中率

```
MariaDB [hellodb]> select * from students where stuid=10;
+-------+--------------+-----+--------+---------+-----------+
| StuID | Name         | Age | Gender | ClassID | TeacherID |
+-------+--------------+-----+--------+---------+-----------+
|    10 | Yue Lingshan |  19 | F      |       3 |      NULL |
+-------+--------------+-----+--------+---------+-----------+
1 row in set (0.00 sec)

MariaDB [hellodb]> show global status like 'Qcache%';
+-------------------------+-----------+
| Variable_name           | Value     |
+-------------------------+-----------+
| Qcache_free_blocks      | 1         |
| Qcache_free_memory      | 104838272 |
| Qcache_hits             | 2         |
| Qcache_inserts          | 1         |
| Qcache_lowmem_prunes    | 0         |
| Qcache_not_cached       | 3         |
| Qcache_queries_in_cache | 1         |
| Qcache_total_blocks     | 4         |
+-------------------------+-----------+
8 rows in set (0.00 sec)

```

