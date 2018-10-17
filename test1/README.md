**Oracle实验一：分析SQL执行计划，执行SQL语句的优化指导**  
========
                               ———— 2018年10月17日 16软3刘宇坤/201610414314  
一、第一个查询语句如下  
-------

SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
from hr.departments d，hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT'，'Sales')
GROUP BY department_name;

#### 结果如下图1.1、图1.2与图1.3所示   
![image](https://github.com/201610414314/Oracle/blob/master/test1/2.png)  
   ###### 图1.1  
                    
![image](https://github.com/201610414314/Oracle/blob/master/test1/3.png)  
   ###### 图1.2  
                    
![image](https://github.com/201610414314/Oracle/blob/master/test1/4.png)  
   ###### 图1.3  

二、第二个查询语句如下  
---------

SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
FROM hr.departments d，hr.employees e
WHERE d.department_id = e.department_id
GROUP BY department_name
HAVING d.department_name in ('IT'，'Sales');  

### 查询结果如下图2.1、图2.2与图2.3所示   

![image](https://github.com/201610414314/Oracle/blob/master/test1/2.png)  
   ###### 图2.1  
                    
![image](https://github.com/201610414314/Oracle/blob/master/test1/6.png)  
   ###### 图2.2  

![image](https://github.com/201610414314/Oracle/blob/master/test1/7.png)
   ###### 图2.3    
                    
三、对上面两个查询语句进行比较分析  
---------

   首先从上面的两个查询语句所对应的结果图可以看出两个查询语句对应的查询结果相同但
是第一个查询语句用时较多，并且统计信息中第一个的consistent gets为16，第二个统计信
息的consistent gets为8，所以第二个SQL语句为最优的。

四、将最优的SQL语句（查询2）通过sqldeveloper的优化指导工具进行优化指导，结果如下图4.1所示  
--------

![image](https://github.com/201610414314/Oracle/blob/master/test1/8.jpg)  
   ###### 图4.1  
                    
  通过上图可以看出经过Sqldeveloper优化工具后，发现已经是最佳优化了。


