# SQL29 计算用户的平均次日留存率

困难  通过率：48.59%  时间限制：1秒  空间限制：256M

## 描述

题目：现在运营想要查看用户在某天刷题后第二天还会再来刷题的平均概率。请你取出相应数据。

示例：question_practice_detail

| id  | device_id | quest_id | result    | date       |
| --- | --------- | -------- | --------- | ---------- |
| 1   | 2138      | 111      | wrong     | 2021-05-03 |
| 2   | 3214      | 112      | wrong<br> | 2021-05-09 |
| 3   | 3214      | 113      | wrong<br> | 2021-06-15 |
| 4   | 6543      | 111      | right     | 2021-08-13 |
| 5   | 2315      | 115      | right<br> | 2021-08-13 |
| 6   | 2315      | 116      | right<br> | 2021-08-14 |
| 7   | 2315      | 117      | wrong<br> | 2021-08-15 |
| ……  | <br>      | <br>     | <br>      | <br>       |

根据示例，你的查询应返回以下结果：

| avg_ret |
| ------- |
| 0.3000  |

## 答案

```sql
select sum(qx.device_id is not null::int)::float / count(q.device_id)
from
(select distinct device_id, date from question_practice_detail) as q
left join
(select distinct device_id, date + 1 as d1 from question_practice_detail) as qx
on q.device_id = qx.device_id and q.date = qx.d1
```

## 思路

考察对 date operator & function 的运用。`date + 1` 就是表示后一天，例如 `date '2021-08-14' + 1` 就是 `2018-08-15`。

我们要找出所有 `date` 和 `date + 1` 都出现在表里面的 `device_id`，例如 `2315`，然后除以所有的 `device_id` 的数目。

`select distinct device_id, date` 相当于是 `select distinct (device_id, date)`，即将unique的 `(device_id, date)` pair 筛选出来。用 `disinct` 的目的是去除用户在同一天回答的多道题目。这里我们只关注用户某一天有没有答题，至于答了1道题还是10道不同的题是无关的。

所以用 `distinct`之后，return的表中仍会有重复的 `device_id`，但是`(device_id, date)` pair是unique的。比如表的一部分是

| device_id | date       |
| --------- | ---------- |
| 2315      | 2021-08-13 |
| 2315      | 2021-08-14 |
| 2315      | 2021-08-15 |

`left join` 之后会发生什么呢？`on q.device_id = qx.device_id and q.date = qx.d1` 这个条件就可以筛选出 `date + 1` 在原表里的那些row了。因为是 `left join`，所以不管怎样，对于左边的原表，所有unique的 `(device_id, date)` pair都会在，都不会缺失。而对于右边的表来说，`d1`即 `date + 1`在左边的表没有的，就是`null`了，而有的，就是符合条件的row。

用有数据的（即不是 `null` 的）数目除以所有的`(device_id, date)` pair的数目，就是用户第二天还来答题的概率，即次日留存率。

需要注意的是，用户 `2315` 连续答了3天题，即把 “在某天刷题后第二天还会再来刷题” 这件事进行了两遍。而这里我们把这两遍都算进分子里了。所以说，我们应该以 `(device_id, date)` pair 为单位进行统计，而不是 user 或 device_id。换句话说，我们关心的是某个用户某天刷题之后第二天还会不会来刷题，而并不关心这些用户具体都是谁。


