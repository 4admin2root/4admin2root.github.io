---
layout:     post
title:      转：15SQLite3SQLCommandsExplainedwithExamples
date:       2012-09-21
tags: [cnblogs]
---
原文[http://www.thegeekstuff.com/2012/09/sqlite-command-examples/](http://www.thegeekstuff.com/2012/09/sqlite-command-examples/)

# 15 SQLite3 SQL Commands Explained with Examples

by Ramesh Natarajan on <abbr class="published" title="2012-09-20">September 20, 2012</abbr>

While most of the commands in the SQLite are similar to SQL commands of other datbases like MySQL and ORACLE, there are some SQLite SQL commands that are different.

This article explains all the basic SQL commands that you need to know to use the SQLite database effectively.

If you dont have sqlite installed, execute yum install sqlite to install it. You can also [install SQLite database from source](http://www.thegeekstuff.com/2011/07/install-sqlite3/) to get the latest version.

### 1. Create a SQLite Database (and a Table)

First, let us understand how create a SQLite database with couple of tables, populate some data, and view those records.

The following example creates a database called employee.db. This 
also creates an employee table with 3 columns (id, name and title), and a
 department table in the company.db database. Weve purposefully missed 
the deptid column in the employee table. Well see how to add that 
later.

```
# sqlite3 company.db
sqlite> create table employee(empid integer,name varchar(20),title varchar(10));
sqlite> create table department(deptid integer,name varchar(20),location varchar(10));
sqlite> .quit
```

A SQLite database is nothing but a file that gets created under your current directory as shown below.

```
# ls -l company.db
-rw-r--r--. 1 root root 3072 Sep 19 11:21 company.db
```

### 2. Insert Records

The following example populates both employee and department table with some sample records.

You can execute all the insert statements from the sqlite command line, or you can add those commands into a file and execute the file as shown below.

First, create a insert-data.sql file as shown below.

```
# vi insert-data.sql
insert into employee values(101,'John Smith','CEO');
insert into employee values(102,'Raj Reddy','Sysadmin');
insert into employee values(103,'Jason Bourne','Developer');
insert into employee values(104,'Jane Smith','Sale Manager');
insert into employee values(105,'Rita Patel','DBA');

insert into department values(1,'Sales','Los Angeles');
insert into department values(2,'Technology','San Jose');
insert into department values(3,'Marketing','Los Angeles');
```

The following will execute all the commands from the insert-data.sql in the company.db database

```
# sqlite3 company.db < insert-data.sql
```

### 3. View Records

Once youve inserted the records, view it using select command as shown below.

```
# sqlite3 company.db
sqlite> select * from employee;
101|John Smith|CEO
102|Raj Reddy|Sysadmin
103|Jason Bourne|Developer
104|Jane Smith|Sale Manager
105|Rita Patel|DBA

sqlite> select * from department;
1|Sales|Los Angeles
2|Technology|San Jose
3|Marketing|Los Angeles
```

### 4. Rename a Table

The following example renames department table to dept using the alter table command.

```
sqlite> alter table department rename to dept;
```

### 5. Add a Column to an Existing Table

The following examples adds deptid column to the existing employee table;

```
sqlite> alter table employee add column deptid integer;
```

Update the department id for the employees using update command as shown below.

```
update employee set deptid=3 where empid=101;
update employee set deptid=2 where empid=102;
update employee set deptid=2 where empid=103;
update employee set deptid=1 where empid=104;
update employee set deptid=2 where empid=105;
```

Verify that the deptid is updated properly in the employee table.

```
sqlite> select * from employee;
101|John Smith|CEO|3
102|Raj Reddy|Sysadmin|2
103|Jason Bourne|Developer|2
104|Jane Smith|Sale Manager|1
105|Rita Patel|DBA|2
```

### 6. View all Tables in a Database

Execute the following command to view all the tables in the current database. The folowing example shows that there are two tables in the current database.

```
sqlite> .tables
dept      employee
```

### 7. Create an Index

The following example creates an unique index called empidx on the empid field of employee table.

```
sqlite> create unique index empidx on employee(empid);
```

Once an unique index is created, if you try to add another record with an empid that already exists, youll get an error as shown below.

```
sqlite> insert into employee values (101,'James Bond','Secret Agent',1);
Error: constraint failed
```

### 8. Create a Trigger

For this example, first add a date column called updatedon on employee table.

```
sqlite> alter table employee add column updatedon date;
```

```
# vi employee_update_trg.sql
create trigger employee_update_trg after update on employee
begin
  update employee set updatedon = datetime('NOW') where rowid = new.rowid;
end;
```

Create the trigger on the company.db database as shown below.

```
# sqlite3 company.db < employee_update_trg.sql
```

```
# sqlite3 company.db
sqlite> update employee set title='Sales Manager' where empid=104;

sqlite> select * from employee;
101|John Smith|CEO|3|
102|Raj Reddy|Sysadmin|2|
103|Jason Bourne|Developer|2|
104|Jane Smith|Sales Manager|1|2012-09-15 18:29:28
105|Rita Patel|DBA|2|
```

### 9. Create a View

The following example creates a view called empdept, which combines fields from both employee and dept table.

```
sqlite> create view empdept as select empid, e.name, title, d.name, location from employee e, dept d where e.deptid = d.deptid;
```

```
sqlite> select * from empdept;
101|John Smith|CEO|Marketing|Los Angeles
102|Raj Reddy|Sysadmin|Technology|San Jose
103|Jason Bourne|Developer|Technology|San Jose
104|Jane Smith|Sales Manager|Sales|Los Angeles
105|Rita Patel|DBA|Technology|San Jose
```

After creating a view, if you execute .tables, youll also see the view name along with the tables.

```
sqlite> .tables
dept      empdept   employee
```

### 10. SQLite Savepoint, Rollback, Commit

Currently dept table has the following 3 records.

```
sqlite> select * from dept;
1|Sales|Los Angeles
2|Technology|San Jose
3|Marketing|Los Angeles
```

```
sqlite> savepoint major;
sqlite> insert into dept values(4,'HR','Los Angeles');
sqlite> insert into dept values(5,'Finance','San Jose');
sqlite> delete from dept where deptid=1;
sqlite> select * from dept;
2|Technology|San Jose
3|Marketing|Los Angeles
4|HR|Los Angeles
5|Finance|San Jose
```

```
sqlite> rollback to savepoint major;
sqlite> select * from dept;
1|Sales|Los Angeles
2|Technology|San Jose
3|Marketing|Los Angeles
```

If you dont want your savepoints anymore, you can erase it using release command.

```
sqlite> release savepoint major;
```

### 11. Additional Date Functions

```
sqlite> select empid,datetime(updatedon,'localtime') from employee;
104|2012-09-15 11:29:28
```

You can also use strftime to display the date column in various output.

```
sqlite> select empid,strftime('%d-%m-%Y %w %W',updatedon) from employee;
104|19-09-2012 3 38
```

The following are the possible modifers you can use in the strftime function.

- %d day of month: 00
- %f fractional seconds: SS.SSS
- %H hour: 00-24
- %j day of year: 001-366
- %J Julian day number
- %m month: 01-12
- %M minute: 00-59
- %s seconds since 1970-01-01
- %S seconds: 00-59
- %w day of week 0-6 with Sunday==0
- %W week of year: 00-53
- %Y year: 0000-9999
- %% %

### 12. Dropping Objects

You can drop all the above created objects using the appropriate drop command as shown below.

```
# cp company.db test.db

# sqlite3 test.db
sqlite> .tables
dept      empdept   employee

sqlite> drop index empidx;
sqlite> drop trigger employee_update_trg;
sqlite> drop view empdept;
sqlite> drop table employee;
sqlite> drop table dept;
```

All the tables and views from the test.db are now deleted.

```
sqlite> .tables
sqlite>
```

### 13. Operators

The following are the possible operators you can use in SQL statements.

- ||
- * / %
- + -
- << >> & |
- < >=
- = == != <> IS IS NOT IN LIKE GLOB MATCH REGEXP
- AND OR

For example:

```
sqlite> select * from employee where empid >= 102 and empid  select * from dept where location like 'Los%';
1|Sales|Los Angeles
3|Marketing|Los Angeles
```

### 14. Explain Query Plan

Execute explain query plan, to get information about the table that is getting used in a query or view. This is very helpful when you are debugging a complex query with multiple joins on several tables.

```
sqlite> explain query plan select * from empdept;
0|0|TABLE employee AS e
1|1|TABLE dept AS d
```

For a detailed trace, just execute explain followed by the query to get more performance data on the query. This is helpful for debugging purpose when the query is slow.

```
sqlite> explain select empid,strftime('%d-%m-%Y %w %W',updatedon) from employee;
0|Trace|0|0|0||00|
1|Goto|0|12|0||00|
2|OpenRead|0|2|0|4|00|
3|Rewind|0|10|0||00|
4|Column|0|0|1||00|
5|String8|0|3|0|%d-%m-%Y %w %W|00|
6|Column|0|3|4||00|
7|Function|1|3|2|strftime(-1)|02|
8|ResultRow|1|2|0||00|
9|Next|0|4|0||01|
10|Close|0|0|0||00|
11|Halt|0|0|0||00|
12|Transaction|0|0|0||00|
13|VerifyCookie|0|19|0||00|
14|TableLock|0|2|0|employee|00|
15|Goto|0|2|0||00|
```

### 15. Attach and Detach Database

When you have multiple database, you can use attach command to execute queries across database.

For example, if you have two database that has the same table name with different data, you can create a union query across the database to view the combined records as explained below.

In this example, we have two company database (company1.db and company2.db). From the sqlite prompt, attach both these database by giving alias as c1 and c2 as shown below.

```
# sqlite3
sqlite> attach database 'company1.db' as c1;
sqlite> attach database 'company2.db' as c2;
```

Execute .database command which will display all the attached databases.

```
sqlite> .database
seq  name             file
---  ---------------  ------------------
0    main
2    c1               /root/company1.db
3    c2               /root/company2.db
```

```
sqlite> select empid, name, title from c1.employee union select empid, name, title from c2.employee;
101|John Smith|CEO
102|Raj Reddy|Sysadmin
103|Jason Bourne|Developer
104|Jane Smith|Sales Manager
105|Rita Patel|DBA
201|James Bond|Secret Agent
202|Spider Man|Action Hero
```

After attaching a database, from the current sqlite session, if you want to detach it, use detach command as shown below.

```
sqlite> detach c1;

sqlite> .database
seq  name             file
---  ---------------  -----------------
0    main
2    c2               /root/company2.db
```
