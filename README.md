# Leetcode-SQL-
The Leetcode questions and answers

### No.1 Select the customer which is not concluded in the order table ( where/ not in)
### 183. Customers Who Never Order
```SQL
SELECT Name AS Customers
FROM Customers
WHERE Id NOT IN
(SELECT CustomerId  FROM Orders);
```

### No.2 Select the people who has the highest salary in every department.
### 184. Department Highest Salary
Tip is joining the two tables and filter the salary with a subquery using MAX() and Group by(). Notice that we should define the salary in a specific department, so we need to select department as well.
```SQL
SELECT d.Name AS Department,e.Name AS Employee,Salary
FROM Employee e
JOIN Department d
ON e.DepartmentId=d.Id
WHERE (DepartmentId, Salary) IN
(select DepartmentId, max(Salary)
 FROM Employee
 GROUP BY DepartmentId
)      # here we need to select the employee's name, so we cannot use groupby() directly.
```

### No.3 Query the top n things
### 185. Department Top Three Salaries
When we want to query the top n things, we need to join the same table once, and use 'where n > select count(distinct *) from table where table1.* >table2.* ' and in where they ranked the top. This is the key.
``` SQL
SELECT d.Name AS Department, e1.Name AS Employee, Salary
FROM Employee e1
LEFT JOIN Department d
ON e1.DepartmentId=d.Id
WHERE 3 >
(SELECT COUNT(DISTINCT e2.Salary)   # distinct here!
FROM Employee e2
WHERE e2.Salary > e1.Salary
AND e1.DepartmentId=e2.DepartmentId)
```

### No.4 Select the colunmn with more than N things 
### NO.4 Classes More Than 5 Students
``` SQL
SELECT class 
FROM  
(SELECT class, COUNT(DISTINCT student) AS num
 FROM courses
GROUP BY class) a
WHERE num >=5;
```

### No.5 Delete the replicates which have larger row number/ id
### 196. Delete Duplicate Emails
Here we use `delete` to drop out the replicates. And we delte from two same tables so that we can use where to filter the id problem
``` SQL
DELETE p1
FROM Person p1,
    Person p2
WHERE
    p1.Email = p2.Email AND p1.Id > p2.Id
;
```
### No.6 Find out the the id which has rising tempreture compared with its former month
### 197. Rising Temperature
  * The idea could be the same with the problems above. So here we can just use the table twice and where clause to realize it.
``` SQL
select w1.Id
from Weather w1, Weather w2
where w1.RecordDate = w2.RecordDate+1 AND w1.Temperature > w2.Temperature;
```
  * Or we can use `join on` the same table to filter the data
  ``` SQL
SELECT
    weather.id AS 'Id'
FROM
    weather
        JOIN
    weather w ON DATEDIFF(weather.date, w.date) = 1
        AND weather.Temperature > w.Temperature
;
```
### NO.7 Qeury the nth one in the database
### 176. Second Highest Salary
Here, we can use `order by` and `limit offset` to filter the data. <br>
`limit` means we read how many lines from the top, `offset` means we skip how many rows to read
``` SQL
SELECT Salary AS SecondHighestSalary
FROM 
(SELECT DISTINCT Salary FROM 
 Employee
 ORDER BY Salary DESC
 limit 1,1) sub ;
```
``` SQL
SELECT
    (SELECT DISTINCT
            Salary
        FROM
            Employee
        ORDER BY Salary DESC
        LIMIT 1 OFFSET 1) AS SecondHighestSalary
;
```

### NO.8 Find out the duplicate emails
### 182. Duplicate Emails
here we use `Group by` to handle this problem, remember use `having` after the `where` clause
``` SQL
select Email
from Person
group by Email
having count(Email) > 1;
``` 

### NO.9  For this kind of ranking question
### 178. Rank Scores
we just need to compare the score using where to filter the data. And what we should keep in mind is that we need to use distinct for same scores will go into same ranking. 
```SQL
SELECT s.Score,
(SELECT COUNT(DISTINCT Score) FROM Scores WHERE Score >= s.Score) Rank
FROM Scores s
ORDER BY s.Score DESC
```

### NO.10 Exchange the value with adjacent row
### 626. Exchange Seats
This problem examined mod() and case when. If we want to exchange the adjacent seats, we need to figure out the odd and even number and +1,-1 respectively. Notice we need to know how many records it have, when the last number is odd, we should take the last one out and keep its original id. 
```SQL
SELECT CASE WHEN mod(id,2) !=0 AND id = counts THEN id 
        WHEN mod(id,2) !=0 THEN id+1 
        ELSE id-1 END AS id ,student
FROM seat,(
SELECT COUNT(id) AS counts
    FROM seat) AS seat_count
order by id
```


# HakerRank Problem

### No.1  query the city with maximal or minimal length of name. 
Here we need to use union to link them. And we use limit 1 to choose the first one. And when it comes to maximal or minimal, we can first think about using order by. Because when we want to use `Max` or `Min`, we need to use `Group by`
```SQL
(SELECT CITY, LENGTH(CITY) 
FROM STATION 
ORDER BY LENGTH(CITY), CITY LIMIT 1) 
UNION 
(SELECT CITY, LENGTH(CITY) FROM STATION 
ORDER BY LENGTH(CITY) DESC, CITY ASC LIMIT 1);
```

### No.2 Fuzzy or accurate search
here we need to understand the difference between `Like` and `Regexp`
  * `Regexp` search data having one of the letter in the beginning of the value like `^[wk]` 
``` SQL
SELECT DISTINCT CITY
FROM STATION 
WHERE LOWER(CITY) REGEXP '^[aeiou]';
```
  * `Regexp` search data having the condition in the end of the value like `[ww]$`
```SQL
SELECT DISTINCT CITY
FROM STATION
WHERE LOWER(CITY) REGEXP '[aeiou]$'
```
  * `Regexp` search data having the condition in the both beginning and end of the value like `^[ek].*[12]$`
```SQL
SELECT DISTINCT city FROM station WHERE city RLIKE '^[aeiou].*[aeiou]$'
SELECT DISTINCT City
FROM Station
WHERE City RLIKE '^[^AEIOU].*[^aeiou]$';
```
  * Or we can query data like this
```SQL
select distinct city from station 
where left(city,1) in ('a','e','i','o','u') 
and right(city, 1) in ('a','e','i','o','u')
```
  * not start with aeiou 
```SQL
SELECT DISTINCT CITY FROM STATION
WHERE CITY REGEXP '^[^aeiou]'
```
```SQL
select DISTINCT city from STATION where city NOT REGEXP '[aeiou]$'
```

### No.3 How to write a function in MYSQL
```
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
    set N = N-1;
    RETURN (SELECT DISTINCT(Salary) FROM Employee ORDER BY Salary DESC LIMIT 1 offset N);
    END
```

### Practice
Practice: 3 Tables <br>
          S: s_no, s_name <br>
          C: c_no, t_name <br>
          SC: s_no, c_no, grade <br>

Q1 : find out the students who didn't choose Li's course
```SQL
select s_name from S
where s_no not in 
(
select sno from SC where c_no in
(select c_no from C 
where t_name='lining')
)
```SQL

Q2 : find out the students who have more than 2 courses less than 60
```SQL
select name
from
(
select s_no 
from SC
where grade <60
group by s_no
having count(s_no) >=2
)a
join 
(
select s_no, name 
from S
)b
on a.s_no=b.s_no
```SQL

Q3 : find out students who choose course1 and course2
```SQL
select name 
from S
where s_no in 
(
select * 
from 
(
select s_no 
from SC 
where c_no=1
)a
join
(
select s_no
from SC
where c_no=2
)b
on a.s_no = b.s_no
)
