
| 约束类型 | 规范命名     | 名称说明    |
|:-------- |:------------ |:----------- |
| 主键约束 | PK_表名_列名 | Primary Key |
| 外键约束 | FK_表名_列名 | Foreign Key |
| 非空约束 | NN_表名_列名 | Not Null    |
| 唯一约束 | UK_表名_列名 | Unique Key  |
| 检查约束 | CK_表名_列名 | Check       |

2.2 约束信息查询
1. 常用视图 (权限由大到小: dba_* > all_* > user_*)
   (1) dba_constraints : 侧重约束具体信息
   (2) dba_cons_columns: 侧重约束列信息

2. 参考如下
   select *
     from dba_constraints dc
    where dc.owner = 'SCOTT'
      and dc.table_name = 'EMP';
  
   select *
     from dba_cons_columns dcc
    where dcc.owner = 'SCOTT'
      and dcc.table_name = 'EMP';

2.3 添加约束
-- 1.唯一性约束
alter table 表名 add constraint uk_* unique(列名) [not null];

-- 2.检查约束
alter table 表名 add constraint ck_* check(列名 between 1 and 100); 
alter table 表名 add constraint ck_* check(列名 in ('值1', '值n')); 

-- 3.非空约束(多个约束中，not null 位于末尾)
alter table 表名 modify(列名 constraint nk_* not null);

2.4 删除约束
alter table 表名 drop constraint 约束名;
1
2.5 重命名约束
alter table 表名 rename constraint 约束名 to new_约束名;
1
2.6 禁用启用约束
-- 1.禁用 disable
alter table 表名 disable constraint 约束名 [cascade];

-- 2.启用 enable
alter table 表名 enable constraint 约束名 [cascade];

3 约束分类
3.1 主键约束 P
create table scott.sex (
   sex_code  varchar2(2) constraint pk_sex_code primary key,
   sex_desc  varchar2(10)
);

3.2 外键约束 R
create table scott.student_info (
   sno      number(10) constraint pk_student_info_sno primary key,
   sname    varchar2(30),
   sex_code varchar2(2),
   constraint fk_student_info_sex_code foreign key(sex_code) references scott.sex(sex_code)
);
1
2
3
4
5
6
外键约束有以下 3 种情况：子表 引用 父表

1. 普通外键约束: 删除 父表 记录时，'报错'。'默认'
2. 级联外键约束: 删除 父表 记录时，同时 '删除子表记录'
3. 置空外键约束: 删除 父表 记录时，同时将 '子表记录置为 null'
————————————————
版权声明：本文为CSDN博主「鱼丸丶粗面」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_34745941/article/details/81161479