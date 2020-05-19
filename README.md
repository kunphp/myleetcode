# myleetcode
## mysql

**1、编写一个 SQL 查询来实现分数排名。如果两个分数相同，则两个分数排名（Rank）相同**

| Id | Score |
|----|-------|
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |

```mysql
  select a.Score, (
     select count(distinct b.Score) from Scores as b 
     where b.Score>=a.Score 
  ) as 'Rank' 
  from Scores as a
  group by a.id
  order by a.Score desc
  // 思路：分数比大小查询出不重复的分数的数量作为排名
```
**2、编写一个 SQL 查询，获取 Employee 表中第 n 高的Salary。**

| Id | Salary |
|----|--------|
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
```mysql
  N = N-1；
  select distinct IFNULL(Salary,NULL) from Employee order by Salary DESC limit N,1
  // 思路：先 order by 然后 limit （注意limit下标从0开始）
```
**3、编写一个 SQL 查询，查找所有至少连续出现三次的数字。**

| Id | Num |
|----|-----|
| 1  |  1  |
| 2  |  1  |
| 3  |  1  |
| 4  |  2  |
| 5  |  1  |
| 6  |  2  |
| 7  |  2  |

```mysql
  select distinct a.Num  from Logs as a,Logs as b, Logs as c 
  where a.Id = b.Id+1 and b.Id = c.Id+1 and a.Num=b.Num and b.Num=c.Num
  //思路：利用多表查出至少相邻三行数据其字段相等并且去重
```

**4、编写一个 SQL 查询，查找各个部门工资最高的员工（有可能多个最高）。**

`Employee表`

| Id | Name  | Salary | DepartmentId |
|----|-------|--------|--------------|
| 1  | Joe   | 70000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |

```mysql
//不考虑存在多个最高
select Name,max(Salary) from Employee GROUP BY DepartmentId;

//存在同个部门多个最高的
select DepartmentId,Name,Salary from Employee 
where 
(DepartmentId,Salary) in 
(
    select DepartmentId,max(Salary) from Employee GROUP BY DepartmentId
)
//思路：in方法多个字段的使用
```
**5、编写一个 SQL 查询，找出每个部门获得前三高工资的所有员工**

`Employee表`

| Id | Name  | Salary | DepartmentId |
|----|-------|--------|--------------|
| 1  | Joe   | 85000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
| 5  | Janet | 69000  | 1            |
| 6  | Randy | 85000  | 1            |
| 7  | Will  | 70000  | 1            |

`Department` 表包含公司所有部门的信息。

| Id | Name     |
|----|----------|
| 1  | IT       |
| 2  | Sales    |

结果例如，根据上述给定的表，查询结果应返回：

| Department | Employee | Salary |
|------------|----------|--------|
| IT         | Max      | 90000  |
| IT         | Randy    | 85000  |
| IT         | Joe      | 85000  |
| IT         | Will     | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |


```sql
SELECT P2.Name AS Department,P3.Name AS Employee,P3.Salary AS Salary
FROM Employee AS P3
INNER JOIN Department AS P2
ON P2.Id = P3.DepartmentId 
WHERE 
(
    SELECT COUNT(DISTINCT Salary)
    FROM Employee AS P4
    WHERE P3.DepartmentId = P4.DepartmentId
    AND P4.Salary >= P3.Salary
) <= 3  //核心：查出相同部门而且Salary比我高的人数不超过3个
ORDER BY DepartmentId,Salary DESC
```

**6、编写一个 SQL 查询，将学生信息和成绩汇总为一行**

`Scores表`

| Id | item  | Score | StudentId |
|----|-------|--------|--------------|
| 1  | 语文   | 100  | 1            |
| 2  | 数学 | 98  | 2            |
| 3  | 英语   | 72  | 2            |
| 4  | 语文   | 95  | 3            |
| 5  | 数学 | 93  | 1            |
| 6  | 语文 | 89  | 2            |
| 7  | 英语  | 84  | 1            |
| 8  | 数学  | 82  | 3            |
| 9  | 英语  | 99  | 3            |

`Student` 表包含公司所有部门的信息。

| Id | Name   |
|----|--------|
| 1  |ZhangSan|
| 2  |LiSi    |
| 3  |WangWu  |

根据上述给定的表，查询结果应返回：

| 姓名 | 语文 | 数学 | 英语 |
|------|-----|------|-----|
| ZhangSan| 100  | 93 | 94 |
| LiSi   | 89| 98 | 72 |
| WangWu  | 95 | 82  |99|


```mysql
select 
     tmp.Name '姓名',
	 max(case item when '语文' then Score end) '语文',
	 max(case item when '数学' then Score end) '数学',
	 max(case item when '英语' then Score end) '英语'
from 
(
	select s.`item`,s.`Score`,s.StudentId,t.Name from Scores s LEFT join  Student t on t.Id = s.StudentId 
) as tmp
GROUP BY StudentId
```

## 算法
**1、一个整数数组 nums ，请你找出数组中乘积最大的连续子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积**
```php
class Solution {

    /**
     * @param Integer[] $nums
     * @return Integer
     */
    function maxProduct($nums) {
        $n = count($nums);
        $maxs[0] = $nums[0];
        $mins[0] = $nums[0];
        $max = $nums[0];
        for($i = 1; $i < $n; $i++){
            $maxs[$i] = max($maxs[$i-1]*$nums[$i],max($mins[$i-1]*$nums[$i],$nums[$i]));
            $mins[$i] = min($mins[$i-1]*$nums[$i],min($maxs[$i-1]*$nums[$i],$nums[$i]));
            $max = max($max, $maxs[$i]);
        }
        return $max;
    }
}
```
