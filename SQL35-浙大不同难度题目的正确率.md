# SQL35 浙大不同难度题目的正确率

困难  通过率：37.07%  时间限制：1秒  空间限制：256M

## 描述

题目：现在运营想要了解**浙江大学的用户在不同难度题目下答题的正确率情况**，请取出相应数据，并按照准确率升序输出。

示例： user_profile

| id  | device_id | gender | age | university | gpa | active_days_within_30 | question_cnt | answer_cnt |
| --- | --------- | ------ | --- | ---------- | --- | --------------------- | ------------ | ---------- |
| 1   | 2138      | male   | 21  | 北京大学       | 3.4 | 7                     | 2            | 12         |
| 2   | 3214      | male   |     | 复旦大学       | 4   | 15                    | 5            | 25         |
| 3   | 6543      | female | 20  | 北京大学       | 3.2 | 12                    | 3            | 30         |
| 4   | 2315      | female | 23  | 浙江大学       | 3.6 | 5                     | 1            | 2          |
| 5   | 5432      | male   | 25  | 山东大学       | 3.8 | 20                    | 15           | 70         |
| 6   | 2131      | male   | 28  | 山东大学       | 3.3 | 15                    | 7            | 13         |
| 7   | 4321      | female | 26  | 复旦大学       | 3.6 | 9                     | 6            | 52         |

示例： question_practice_detail

| id  | device_id | question_id | result |
| --- | --------- | ----------- | ------ |
| 1   | 2138      | 111         | wrong  |
| 2   | 3214      | 112         | wrong  |
| 3   | 3214      | 113         | wrong  |
| 4   | 6543      | 111         | right  |
| 5   | 2315      | 115         | right  |
| 6   | 2315      | 116         | right  |
| 7   | 2315      | 117         | wrong  |

示例： question_detail

| question_id | difficult_level |
| ----------- | --------------- |
| 111         | hard            |
| 112         | medium          |
| 113         | easy            |
| 115         | easy            |
| 116         | medium          |
| 117         | easy            |

根据示例，你的查询应返回以下结果：

| difficult_level | correct_rate |
| --------------- | ------------ |
| easy            | 0.5000       |
| medium          | 1.0000       |

[题目链接](https://www.nowcoder.com/practice/d8a4f7b1ded04948b5435a45f03ead8c)

## 答案

```sql
select x.difficult_level, count(x.result) filter (where x.result='right')/count(x.result)::float as correct_rate
from
-- this table is (device_id, question_id, result(w/r), diff_level) for ZJU users
(select distinct q.device_id, q.question_id, q.result, qd.difficult_level from
(select device_id, university from user_profile where university = '浙江大学') as u
left join 
question_practice_detail as q
on q.device_id = u.device_id
right join question_detail as qd
on q.question_id = qd.question_id) x
-- end table
group by x.difficult_level
order by correct_rate asc
```

Note: 对于MySQL，`count(x.result) filter (where x.result='right')` 要替换成 `sum(x.result='right')`。

## 思路

这个题目有三个表，属于困难类别，乍一看感觉query好像很难写，但问题本身并不难，我们一步一步来，答案就会水到渠成。

首先我们明确每个表的性质。要进行计算的表主要是第二张表，`question_practice_detail` （简称`q`表），这里面有用户答题的历史数据。`user_profile` （简称`u`表）主要是用来filter浙江大学的用户，`question_detail` （简称`qd`表）用来给每个题目贴difficulty level的标签。这两个表都属于辅助性质，不是主要用于计算的表。

明确了每个表是干嘛的，就可以开始写query了。

1️⃣ 第一步，可以先把 

```sql
select device_id, university from user_profile where university = '浙江大学'
```

写上。这是最容易的。

2️⃣ 把筛选过的用户表 `u` join到`q`表上面，需要匹配的（即`on`的）column是`device_id`。用户表的`device_id`是永远准确无误的，故以用户表的`device_id`为准，即答案中的`left join`。

```sql
u left join q
on q.device_id = u.device_id
```

3️⃣ 我们需要`qd`表中的题目标签信息，所以把此表也join到`q`表上面，`on`的column是`question_id`。道理同上，以`qd`表的column为准，故答案中的`right join`。

```sql
q right join qd
on q.question_id = qd.question_id
```

4️⃣ 合并后的表要有 `(device_id, question_id, result(w/r), diff_level)` 这些信息，所以前面写上 `select` 这些column。

```sql
select distinct q.device_id, q.question_id, q.result, qd.difficult_level from x
```

5️⃣ 接下来，我们就可以group by 了。group by difficulty level即可。三个表合并后括起来，命名为`x`，然后从`x`里`select` 这两个最终的column。

```sql
select x.difficult_level, [calculation] as correct_rate from x
group by x.difficult_level
```

注意在query中，我们最好把每个column来源的表用dot写在前面，这样可以更清楚、少出错。

（完）
