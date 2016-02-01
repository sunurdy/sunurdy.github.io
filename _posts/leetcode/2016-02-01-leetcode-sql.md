---
layout:     post
title:      database related
category: leetcode
description: ALL DATABASE RELATED QUESTIONS

---

## 175. [Combine Two Tables][2]
### description

Table: <code>Person</code></p>

<pre>+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
PersonId is the primary key column for this table.
</pre>

<p>
Table: <code>Address</code></p>
<pre>+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
AddressId is the primary key column for this table.
</pre>

<br>
<p>
Write a SQL query for a report that provides the following information for 
each person in the Person table, regardless if there is an address for each 
of those people:
</p>

<pre>FirstName, LastName, City, State
</pre>

### Solution

```
select FirstName, LastName, City, State 
from Person left join Address 
on Person.PersonID = Address.PersonID;
```

SQL中的相等只有一个`=`,和编程中不同。

---
## [176. Second Highest Salary][3]

### description 
<div class="question-content">
Write a SQL query to get the second highest salary from the <code>Employee</code> table.
<pre>
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
</pre>
<p>For example, given the above Employee table, the second highest salary is <code>200</code>. If there is no second highest salary, then the query should return <code>null</code>.</p></p>
</div>

### solution

```
SELECT MAX(Salary)
FROM Employee
WHERE Salary < ( SELECT MAX( Salary) FROM Employee )
```
```
SELECT IFNULL( (SELECT distinct Salary as SecondHighestSalary FROM Employee order by Salary desc limit 1,1) ,null);
```

[2]: https://leetcode.com/problems/combine-two-tables/
[3]: https://leetcode.com/problems/second-highest-salary/