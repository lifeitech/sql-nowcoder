# SQL23 统计每个学校各难度的用户平均刷题数

中等  通过率：49.97%  时间限制：1秒  空间限制：256M

## 描述

题目：运营想要计算一些**参加了答题**的不同学校、不同难度的用户平均答题量，请你写SQL取出相应数据

用户信息表：user_profile

| id  | device_id | gender | age  | university | gpa | active_days_within_30 | question_cnt | answer_cnt |
| --- | --------- | ------ | ---- | ---------- | --- | --------------------- | ------------ | ---------- |
| 1   | 2138      | male   | 21   | 北京大学       | 3.4 | 7                     | 2            | 12         |
| 2   | 3214      | male   | NULL | 复旦大学       | 4   | 15                    | 5            | 25         |
| 3   | 6543      | female | 20   | 北京大学       | 3.2 | 12                    | 3            | 30         |
| 4   | 2315      | female | 23   | 浙江大学       | 3.6 | 5                     | 1            | 2          |
| 5   | 5432      | male   | 25   | 山东大学       | 3.8 | 20                    | 15           | 70         |
| 6   | 2131      | male   | 28   | 山东大学       | 3.3 | 15                    | 7            | 13         |
| 7   | 4321      | male   | 28   | 复旦大学       | 3.6 | 9                     | 6            | 52         |

第一行表示:id为1的用户的常用信息为使用的设备id为2138，性别为男，年龄21岁，北京大学，gpa为3.4，在过去的30天里面活跃了7天，发帖数量为2，回答数量为12

最后一行表示:id为7的用户的常用信息为使用的设备id为4321，性别为男，年龄28岁，复旦大学，gpa为3.6，在过去的30天里面活跃了9天，发帖数量为6，回答数量为52  

题库练习明细表：question_practice_detail

| id     | device_id | question_id | result    |
| ------ | --------- | ----------- | --------- |
| 1      | 2138      | 111         | wrong     |
| 2<br>  | 3214      | 112         | wrong<br> |
| 3      | 3214      | 113         | wrong<br> |
| 4      | 6534      | 111         | right     |
| 5      | 2315      | 115         | right<br> |
| 6      | 2315      | 116         | right<br> |
| 7      | 2315<br>  | 117         | wrong<br> |
| 8      | 5432      | 117         | wrong<br> |
| 9      | 5432      | 112         | wrong<br> |
| 10     | 2131      | 113         | right     |
| 11<br> | 5432      | 113         | wrong<br> |
| 12     | 2315      | 115         | right     |
| 13     | 2315      | 116         | right     |
| 14     | 2315<br>  | 117         | wrong<br> |
| 15     | 5432<br>  | 117         | wrong<br> |
| 16<br> | 5432      | 112         | wrong<br> |
| 17<br> | 2131      | 113         | right     |
| 18<br> | 5432      | 113         | wrong<br> |
| 19     | 2315      | 117         | wrong<br> |
| 20<br> | 5432      | 117         | wrong<br> |
| 21     | 5432      | 112         | wrong<br> |
| 22     | 2131      | 113         | right     |
| 23<br> | 5432      | 113         | wrong<br> |

第一行表示:id为1的用户的常用信息为使用的设备id为2138，在question_id为111的题目上，回答错误  

......

最后一行表示:id为23的用户的常用信息为使用的设备id为5432，在question_id为113的题目上，回答错误

表：question_detail

| id  | question_id | difficult_level |
| --- | ----------- | --------------- |
| 1   | 111         | hard            |
| 2   | 112         | medium          |
| 3   | 113         | easy            |
| 4   | 115         | easy            |
| 5   | 116         | medium          |
| 6   | 117         | easy            |

第一行表示: 题目id为111的难度为hard

....

第一行表示: 题目id为117的难度为easy

请你写一个SQL查询，计算不同学校、不同难度的用户平均答题量，根据示例，你的查询应返回以下结果(结果在小数点位数保留4位，4位之后四舍五入)：

| university | difficult_level | avg_answer_cnt |
| ---------- | --------------- | -------------- |
| 北京大学       | hard            | 1.0000         |
| 复旦大学       | easy            | 1.0000         |
| 复旦大学       | medium<br>      | 1.0000         |
| 山东大学       | easy            | 4.5000         |
| 山东大学       | medium          | 3.0000         |
| 浙江大学       | easy            | 5.0000         |
| 浙江大学       | medium          | 2.0000         |

解释：

第一行：北京大学有设备id为2138，6543这2个用户，这2个用户在question_practice_detail表下都只有一条答题记录，且答题题目是111，从question_detail可以知道这个题目是hard，故 北京大学的用户答题为hard的题目平均答题为2/2=1.0000

第二行，第三行：复旦大学有设备id为3214，4321这2个用户，但是在question_practice_detail表只有1个用户(device_id=3214有答题，device_id=4321没有答题，不计入后续计算)有2条答题记录，且答题题目是112，113各1个，从question_detail可以知道题目难度分别是medium和easy，故 复旦大学的用户答题为easy, medium的题目平均答题量都为1(easy=1或medium=1) /1 (device_id=3214)=1.0000

第四行，第五行：山东大学有设备id为5432和2131这2个用户，这2个用户总共在question_practice_detail表下有12条答题记录，且答题题目是112，113，117，且数目分别为3，6，3，从question_detail可以知道题目难度分别为medium,easy,easy，所以，easy共有9个，故easy的题目平均答题量= 9(easy=9)/2 (device_id=3214 or device_id=5432) =4.5000，medium共有3个，medium的答题只有device_id=5432的用户，故medium的题目平均答题量= 3(medium=9)/1 ( device_id=5432) =3.0000

.....

## 答案

```sql
select university, difficult_level, count(qanswer.question_id)/ count(distinct qanswer.device_id) as avg_answer_cnt
from question_practice_detail as qanswer
left join user_profile as usr
on usr.device_id = qanswer.device_id
left join question_detail as qtype
on qanswer.question_id = qtype.question_id
group by university, difficult_level
order by university asc
```

## 思路

同上一题，小表 append 到大表上面，然后再group by。

这次我们有三个 table，所以要 left join 两次。


