**Oracle实验三：创建分区表**  
========
                            ———— 2018年10月31日 16软3刘宇坤/201610414314  
一、第1步：在主表orders和从表order_details之间建立引用分区 在new_lyk用户中创建两个表：orders（订单表）和order_details（订单详表），  
两个表通过列order_id建立主外键关联。orders表按范围分区进行存储，order_details使用引用分区进行存储：
-------
##### 首先登录自己的账号new_lyk：  
[oracle@deep02 ~]$ sqlplus new_lyk/123@pdborcl  
SQL*Plus: Release 12.1.0.2.0 Production on 星期三 10月 31 08:51:15 2018  
Copyright (c) 1982, 2014, Oracle.  All rights reserved.  
上次成功登录时间: 星期三 10月 24 2018 09:42:16 +08:00  

连接到:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL>    

##### 其次，进行orders表的创建，语句及结果如下：  
    CREATE TABLE orders (  
    2   order_id NUMBER(10, 0) NOT NULL  
    , customer_name VARCHAR2(40 BYTE) NOT NULL  
    , order_date DATE NOT NULL  
    , customer_tel VARCHAR2(40 BYTE) NOT NULL  
    , order_date DATE NOT NULL  
    6   , employee_id NUMBER(6, 0) NOT NULL  
    , discount NUMBER(8, 2) DEFAULT 0  
    PARTITION BY RANGE (order_date)  
    (  
    PARTITION PARTITION_BEFORE_2016 VALUES LESS THAN (  
    TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS',  
    'NLS_CALENDAR=GREGORIAN'))  
    , trade_receivable NUMBER(8, 2) DEFAULT 0  
    9  )  
    PCTFREE 10  
    TABLESPACE USERS  
    PCTFREE 10 INITRANS 1  
    STORAGE (   BUFFER_POOL DEFAULT )  
    NOCOMPRESS NOPARALLEL  
    BUFFER_POOL DEFAULT  
    PARTITION BY RANGE (order_date)  
    (  
    NOLOGGING    
    TABLESPACE USERS02  
    PARTITION PARTITION_BEFORE_2016 VALUES LESS THAN (  
    (  
    INITIAL 8388608  
    TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS',  
    BUFFER_POOL DEFAULT  
    'NLS_CALENDAR=GREGORIAN'))  
    NOCOMPRESS NO INMEMORY   
    , PARTITION PARTITION_BEFORE_2018 VALUES LESS THAN (  
    NOLOGGING  
    TABLESPACE USERS  
    STORAGE  
    PCTFREE 10  
    MINEXTENTS 1  
    INITRANS 1  
    STORAGE  
    (  
    INITIAL 8388608  
    NEXT 1048576  
    MINEXTENTS 1  
    MAXEXTENTS UNLIMITED  
    BUFFER_POOL DEFAULT  
    )  
    NOCOMPRESS NO INMEMORY  
    , PARTITION PARTITION_BEFORE_2017 VALUES LESS THAN (  
    TO_DATE(' 2017-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS',  
    'NLS_CALENDAR=GREGORIAN'))  
    NOLOGGING  
    TABLESPACE USERS02  
    PCTFREE 10  
    INITRANS 1  
    STORAGE  
    (  
    INITIAL 8388608  
    NEXT 1048576  
    MINEXTENTS 1  
    MAXEXTENTS UNLIMITED  
    BUFFER_POOL DEFAULT  
    )  
    NOCOMPRESS NO INMEMORY   
    , PARTITION PARTITION_BEFORE_2018 VALUES LESS THAN (  
    TO_DATE(' 2018-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS',  
    'NLS_CALENDAR=GREGORIAN'))  
    NOLOGGING  
    TABLESPACE USERS03  
    PCTFREE 10  
    INITRANS 1  
    STORAGE  
    (  
    INITIAL 8388608  
    NEXT 1048576  
    MINEXTENTS 1  
    MAXEXTENTS UNLIMITED  
    BUFFER_POOL DEFAULT  
    )  
     63  );   

    表已创建。

##### 然后，进行order_details表的创建,并通过order_id进行主外键关联，语句及结果如下:  
    CREATE TABLE order_details   
    (  
    id NUMBER(10, 0) NOT NULL   
    , order_id NUMBER(10, 0) NOT NULL  
    , product_id VARCHAR2(40 BYTE) NOT NULL   
    , product_num NUMBER(8, 2) NOT NULL   
    , product_price NUMBER(8, 2) NOT NULL   
    , CONSTRAINT order_details_fk1 FOREIGN KEY  (order_id)  
    REFERENCES orders(order_id)  
    ENABLE   
    )   
    TABLESPACE USERS   
    PCTFREE 10 INITRANS 1    
    STORAGE (   BUFFER_POOL DEFAULT )   
    NOCOMPRESS NOPARALLEL  
    PARTITION BY REFERENCE (order_details_fk1)  
    (  
    PARTITION PARTITION_BEFORE_2016   
    NOLOGGING   
    TABLESPACE USERS  
    PCTFREE 10   
    INITRANS 1   
    STORAGE   
    (   
    INITIAL 8388608   
    NEXT 1048576   
    MINEXTENTS 1   
    MAXEXTENTS UNLIMITED   
    BUFFER_POOL DEFAULT   
    )   
    NOCOMPRESS NO INMEMORY,   
    PARTITION PARTITION_BEFORE_2017   
    NOLOGGING   
    TABLESPACE USERS02  
    PCTFREE 10   
    INITRANS 1   
    STORAGE    
    (   
    INITIAL 8388608   
    NEXT 1048576   
    MINEXTENTS 1   
    MAXEXTENTS UNLIMITED   
    BUFFER_POOL DEFAULT   
    )   
    NOCOMPRESS NO INMEMORY,  
    PARTITION PARTITION_BEFORE_2018   
    NOLOGGING   
    TABLESPACE USERS03  
    PCTFREE 10   
    INITRANS 1   
    STORAGE   
    (   
    INITIAL 8388608   
    NEXT 1048576   
    MINEXTENTS 1   
    MAXEXTENTS UNLIMITED   
    BUFFER_POOL DEFAULT   
    )   
    );  

    Table ORDER_DETAILS 已创建。

二、第2步：给用户进行分配可查询的权限，如下图2.1所示： 
---------
![image](https://github.com/201610414314/Oracle/blob/master/test3/1.jpg)  
   ###### 图2.1  

三、第3步：插入一万条数据语句及联合查询语句：  
--------
##### 向两个表中各自插入10000条数据：
    begin
    for i in 1..3000
    loop
    insert into ORDERS(ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT) VALUES(i,'李四',12345,to_date('2017-02-14','yyyy-mm-dd'),i,i);
    end loop;
    commit;
    
    for i in 3001..6000
    loop
     insert into ORDERS(ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT) VALUES(i,'李四',12345,to_date('2015-02-14','yyyy-mm-dd'),i,i);
    end loop;
    commit;
    
    for i in 6001..10000
    loop
     insert into ORDERS(ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT) VALUES(i,'王五',12345,to_date('2016-02-14','yyyy-mm-dd'),i,i);
    end loop;
    commit;
    
    for j in 1..3000
    loop
    insert into order_details(ID,ORDER_ID,PRODUCT_ID,PRODUCT_NUM,PRODUCT_PRICE) VALUES(j,j,'j',20,100);
    end loop;
    commit;
    
    for j in 3001..6000
    loop
    insert into order_details(ID,ORDER_ID,PRODUCT_ID,PRODUCT_NUM,PRODUCT_PRICE) VALUES(j,j,'j',30,200);
    end loop;
    commit;
    
        for j in 6001..10000
    loop
    insert into order_details(ID,ORDER_ID,PRODUCT_ID,PRODUCT_NUM,PRODUCT_PRICE) VALUES(j,j,'j',40,300);
    end loop;
    commit;
    end;  
    
 ##### 联合查询语句：
    SELECT
    *
    FROM orders,order_details  Where orders.order_id = order_details.order_id AND
    orders.order_date>=to_date('2016-02-14','yyyy-mm-dd')

四、第4步：对查询语句，进行分析语句的执行计划：  
--------
##### 联合查询语句执行计划如下图4.1所示：
![image](https://github.com/201610414314/Oracle/blob/master/test3/2.jpg)  
   ###### 图4.1      

五、总结分析
--------
- 数据库pdborcl中包含了每个人创建的的角色和用户。  
- 所有人的用户都使用表空间users存储表的数据。  
- 表空间中存储了很多相同名称的表mytable和视图myview，但分别属性于不同的用户，不会引起混淆。  
- 随着用户往表中插入数据，表空间的磁盘使用量会增加。  
- 这次实验，我觉得较上一次来说还是比较轻松地，不仅学会了用ssh来对Oracle数据库的用户方面进行管理操作，  
  还学会了用sqlDeveloper来对Oracle数据库的用户方面进行管理操作。
