**Oracle实验二：用户管理 - 掌握管理角色、权根、用户的能力，并在用户之间共享对象**  
========
                            ———— 2018年10月24日 16软3刘宇坤/201610414314  
一、第1步：以system登录到pdborcl，创建角色con_lyk_view和用户new_lyk，并授权和分配空间，语句与结果如下：
-------

[oracle@deep02 ~]$ sqlplus system/123@pdborcl  
SQL*Plus: Release 12.1.0.2.0 Production on 星期三 10月 24 08:55:12 2018  
Copyright (c) 1982, 2014, Oracle.  All rights reserved.  
上次成功登录时间: 星期三 10月 24 2018 08:54:43 +08:00  
连接到:  
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production  
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options  
SQL> CREATE ROLE con_lyk_view;  
角色已创建。  
SQL> GRANT connect,resource,CREATE VIEW TO con_lyk_view;  
授权成功。  
SQL> CREATE USER new_lyk IDENTIFIED BY 123 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;  
用户已创建。  
SQL> ALTER USER new_lyk QUOTA 50M ON users;  
用户已更改。  
SQL> GRANT con_lyk_view TO new_lyk;  
授权成功。  
SQL> exit    

总结:语句“ALTER USER new_lyk QUOTA 50M ON users;”是指授权new_lyk用户访问users表空间，空间限额是50M。

二、第2步：新用户new_lyk连接到pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户，语句与结果如下： 
---------

[oracle@deep02 ~]$ sqlplus new_lyk/123@pdborcl  
SQL*Plus: Release 12.1.0.2.0 Production on 星期三 10月 24 09:08:19 2018  
Copyright (c) 1982, 2014, Oracle.  All rights reserved.  
连接到:  
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production  
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options  
SQL> show user;  
USER 为 "NEW_LYK"  
SQL> CREATE TABLE mytable (id number,name varchar(50));  
表已创建。  
SQL> INSERT INTO mytable(id,name)VALUES(1,'zhang');  
已创建 1 行。  
SQL> INSERT INTO mytable(id,name)VALUES (2,'wang');  
已创建 1 行。  
SQL> CREATE VIEW myview AS SELECT name FROM mytable;  
视图已创建。  
SQL> SELECT * FROM myview;  
NAME  
zhang
wang

SQL> GRANT SELECT ON myview TO hr;  
授权成功。  
SQL> exit    

总结分析:创建一个名为mytable的表，并向表中插入相关值，并将查询权限赋给hr,这样hr就可以查询表中的值了。

三、第3步：用户hr连接到pdborcl，查询new_lyk授予它的视图myview，语句与结果如下：    
---------
[oracle@deep02 ~]$ sqlplus hr/123@pdborcl

SQL*Plus: Release 12.1.0.2.0 Production on 星期三 10月 24 09:14:20 2018

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

上次成功登录时间: 星期三 10月 24 2018 09:14:06 +08:00

连接到:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL> SELECT * FROM new_user.myview;

NAME  

zhang
wang
zhang
WANG

SQL> exit

总结分析:这是登录的hr用户，如果在上面将权限赋给其他用户，将可以使用其他用户登录，并查询new_lyk插入的值。

四、查看数据库的使用情况，语句与结果如下：  
--------

[oracle@deep02 ~]$ sqlplus system/123@pdborcl  
SQL*Plus: Release 12.1.0.2.0 Production on 星期三 10月 24 09:16:27 2018  
Copyright (c) 1982, 2014, Oracle.  All rights reserved.  
上次成功登录时间: 星期三 10月 24 2018 09:14:53 +08:00  
连接到:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options  
SQL> SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';  
TABLESPACE_NAME

FILE_NAME

        MB     MAX_MB AUTOEXTEN
---------- ---------- ---------
USERS
/home/oracle/app/oracle/oradata/orcl/pdborcl/SAMPLE_SCHEMA_users01.dbf
         5 32767.9844 YES


SELECT from (SELECT tablespace_name,Sum(bytes)free
SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
 free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
 Round(( total - free )/ total,4)* 100 "使用率%"
 from (SELECT tablespace_name,Sum(bytes)free
        FROM   dba_free_space group  BY tablespace_name)a,
       (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
        group  BY tablespace_name)b
  8   where  a.tablespace_name = b.tablespace_name;

表空间名
---------- ----------------------------------- ---------------------------------
    大小MB     剩余MB     使用MB    使用率%
---------- ---------- ---------- ----------
SYSAUX
       630         39        591      93.81

USERS
         5        1.5        3.5         70

SYSTEM
       270     3.5625   266.4375      98.68


表空间名
---------- -------------------------------- ------------------------------------
    大小MB     剩余MB     使用MB    使用率%
---------- ---------- ---------- ----------
EXAMPLE
  1281.875      62.25   1219.625      95.14  
  
  
总结分析:
- autoextensible是显示表空间中的数据文件是否自动增加。  
- MAX_MB是指数据文件的最大容量。  

五、总结分析
--------
- 数据库pdborcl中包含了每个人创建的的角色和用户。  
- 所有人的用户都使用表空间users存储表的数据。  
- 表空间中存储了很多相同名称的表mytable和视图myview，但分别属性于不同的用户，不会引起混淆。  
- 随着用户往表中插入数据，表空间的磁盘使用量会增加。  
- 这次实验，我觉得较上一次来说还是比较轻松地，不仅学会了用ssh来进行对Oracle数据库的用户方面进行管理操作  
  还学会了用sqlDeveloper来对Oracle数据库的用户方面进行管理操作。




