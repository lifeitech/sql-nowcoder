# SQL36 某乎问答高质量的回答中用户属于各级别的数量

中等  通过率：53.71%  时间限制：1秒  空间限制：256M

## 描述

现有某乎问答创作者信息表author_tb如下(其中author_id表示创作者编号、author_level表示创作者级别，共1-6六个级别、sex表示创作者性别)：

| author_id | author_level | sex |
| --------- | ------------ | --- |
| 101       | 6            | m   |
| 102       | 1            | f   |
| 103       | 1            | m   |
| 104       | 3            | m   |
| 105       | 4            | f   |
| 106       | 2            | f   |
| 107       | 2            | m   |
| 108       | 5            | f   |
| 109       | 6            | f   |
| 110       | 5            | m   |

创作者回答情况表answer_tb如下（其中answer_date表示创作日期、author_id指创作者编号、issue_id指问题编号、char_len表示回答字数）：  

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

回答字数大于等于100字的认为是高质量回答，请你统计某乎问答高质量的回答中用户属于1-2级、3-4级、5-6级的数量分别是多少，按数量降序排列，以上例子的输出结果如下：

| level_cut | num |
| --------- | --- |
| 5-6级      | 12  |
| 3-4级      | 2   |
| 1-2级      | 1   |

## 答案

```sql
select case 
when author_level in (5,6) then '5-6级'
when author_level in (3,4) then '3-4级'
when author_level in (1,2) then '1-2级'
end as level_cut, count(auth.author_level) as num
from (select author_id, issue_id, char_len from answer_tb where char_len>=100) as ans
left join author_tb as auth 
on auth.author_id = ans.author_id
group by level_cut
order by num desc
```

## 思路

考察`case when`的运用。`case when` 适合用来categorize数据。

`answer_tb`是我们要进行计算的流水表。我们给每一行贴上`level_cut` （5-6级，3-4级，1-2级）和`author_level` （1, 2, 3, 4, 5, 6）的标签，count `author_level`，然后按照`level_cut`进行group by即可。














