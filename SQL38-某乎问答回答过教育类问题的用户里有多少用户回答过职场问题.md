# SQL38 某乎问答回答过教育类问题的用户里有多少用户回答过职场问题

中等  通过率：63.60%  时间限制：1秒  空间限制：256M

## 描述

现有某乎问答题目信息表issue_tb如下（其中issue_id代表问题编号，issue_type表示问题类型）：

| issue_id | issue_type |
| -------- | ---------- |
| E001     | Education  |
| E002     | Education  |
| E003     | Education  |
| C001     | Career     |
| C002     | Career     |
| C003     | Career     |
| C004     | Career     |
| P001     | Psychology |
| P002     | Psychology |

创作者回答情况表answer_tb如下（其中answer_date表示创作日期、author_id指创作者编号、issue_id指回答问题编号、char_len表示回答字数）：  

| answer_date | author_id | issue_id | char_len |
| ----------- | --------- | -------- | -------- |
| 2021-11-01  | 101       | E001     | 150      |
| 2021-11-01  | 101       | E002     | 200      |
| 2021-11-01  | 102       | C003     | 50       |
| 2021-11-01  | 103       | P001     | 35       |
| 2021-11-01  | 104       | C003     | 120      |
| 2021-11-01  | 105       | P001     | 125      |
| 2021-11-01  | 102       | P002     | 105      |
| 2021-11-02  | 101       | P001     | 201      |
| 2021-11-02  | 110       | C002     | 200      |
| 2021-11-02  | 110       | C001     | 225      |
| 2021-11-02  | 110       | C002     | 220      |
| 2021-11-03  | 101       | C002     | 180      |
| 2021-11-04  | 109       | E003     | 130      |
| 2021-11-04  | 109       | E001     | 123      |
| 2021-11-05  | 108       | C001     | 160      |
| 2021-11-05  | 108       | C002     | 120      |
| 2021-11-05  | 110       | P001     | 180      |
| 2021-11-05  | 106       | P002     | 45       |
| 2021-11-05  | 107       | E003     | 56       |

请你统计回答过教育类问题的用户里有多少用户回答过职场类问题，以上例子的输出结果如下：

| num |
| --- |
| 1   |

## 答案

```sql
select count(distinct author_id) as num
from answer_tb as ans
right join issue_tb as iss
on iss.issue_id = ans.issue_id
where issue_type = 'Career' and
author_id in 
(select distinct author_id
from answer_tb as ans
right join issue_tb as iss
on iss.issue_id = ans.issue_id
where issue_type = 'Education')
```

## 思路

其实intersection是最直观的解法。回答过教育类问题的用户里有哪些用户回答过职场类问题，就等同于（is equivalent to）：即回答过教育类问题、又回答过职场类问题的用户。所以把 `issue_type = 'Education'` 和 `issue_type = 'Career'` 的用户分别filter出来，再intersect就好了。可惜MySQL没有`intersect`，所以我的答案里用`author_id in`的方式解决。

本题中，尽管通过`issue_id`本身的首字母可以看出对应的`issue_type`，但对于其它问题一般而言未必成立，故没有选用相关方法。












