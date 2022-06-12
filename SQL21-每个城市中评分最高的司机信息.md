# SQL21 每个城市中评分最高的司机信息

中等  通过率：30.79%  时间限制：1秒  空间限制：256M

## 描述

用户打车记录表tb_get_car_record

| id  | uid | city | event_time          | end_time            | order_id |
| --- | --- | ---- | ------------------- | ------------------- | -------- |
| 1   | 101 | 北京   | 2021-10-01 07:00:00 | 2021-10-01 07:02:00 | NULL     |
| 2   | 102 | 北京   | 2021-10-01 09:00:30 | 2021-10-01 09:01:00 | 9001     |
| 3   | 101 | 北京   | 2021-10-01 08:28:10 | 2021-10-01 08:30:00 | 9002     |
| 4   | 103 | 北京   | 2021-10-02 07:59:00 | 2021-10-02 08:01:00 | 9003     |
| 5   | 104 | 北京   | 2021-10-03 07:59:20 | 2021-10-03 08:01:00 | 9004     |
| 6   | 105 | 北京   | 2021-10-01 08:00:00 | 2021-10-01 08:02:10 | 9005     |
| 7   | 106 | 北京   | 2021-10-01 17:58:00 | 2021-10-01 18:01:00 | 9006     |
| 8   | 107 | 北京   | 2021-10-02 11:00:00 | 2021-10-02 11:01:00 | 9007     |
| 9   | 108 | 北京   | 2021-10-02 21:00:00 | 2021-10-02 21:01:00 | 9008     |
| 10  | 109 | 北京   | 2021-10-08 18:00:00 | 2021-10-08 18:01:00 | 9009     |

（uid-用户ID, city-城市, event_time-打车时间, end_time-打车结束时间, order_id-订单号）

打车订单表tb_get_car_order

| id  | order_id | uid | driver_id | order_time          | start_time          | finish_time         | mileage | fare | grade |
| --- | -------- | --- | --------- | ------------------- | ------------------- | ------------------- | ------- | ---- | ----- |
| 1   | 9002     | 101 | 202       | 2021-10-01 08:30:00 | NULL                | 2021-10-01 08:31:00 | NULL    | NULL | NULL  |
| 2   | 9001     | 102 | 202       | 2021-10-01 09:01:00 | 2021-10-01 09:06:00 | 2021-10-01 09:31:00 | 10      | 41.5 | 5     |
| 3   | 9003     | 103 | 202       | 2021-10-02 08:01:00 | 2021-10-02 08:15:00 | 2021-10-02 08:31:00 | 11      | 41.5 | 4     |
| 4   | 9004     | 104 | 202       | 2021-10-03 08:01:00 | 2021-10-03 08:13:00 | 2021-10-03 08:31:00 | 7.5     | 22   | 4     |
| 5   | 9005     | 105 | 203       | 2021-10-01 08:02:10 | NULL                | 2021-10-01 08:31:00 | NULL    | NULL | NULL  |
| 6   | 9006     | 106 | 203       | 2021-10-01 18:01:00 | 2021-10-01 18:09:00 | 2021-10-01 18:31:00 | 8       | 25.5 | 5     |
| 7   | 9007     | 107 | 203       | 2021-10-02 11:01:00 | 2021-10-02 11:07:00 | 2021-10-02 11:31:00 | 9.9     | 30   | 5     |
| 8   | 9008     | 108 | 203       | 2021-10-02 21:01:00 | 2021-10-02 21:10:00 | 2021-10-02 21:31:00 | 13.2    | 38   | 4     |
| 9   | 9009     | 109 | 203       | 2021-10-08 18:01:00 | 2021-10-08 18:11:50 | 2021-10-08 18:51:00 | 13      | 40   | 5     |

（order_id-订单号, uid-用户ID, driver_id-司机ID, order_time-接单时间, start_time-开始计费的上车时间,  finish_time-订单完成时间, mileage-行驶里程数, fare-费用, grade-评分）

**场景逻辑说明**：

- 用户提交打车请求后，在用户打车记录表生成一条打车记录，**order_id-订单号**设为null；

- 当有司机接单时，在打车订单表生成一条订单，填充**order_time-接单时间**及其左边的字段，**start_time-开始计费的上车时间**及其右边的字段全部为null，并把**order_id-订单号**和**order_time-接单时间**（**end_time-打车结束时间**）写入打车记录表；若一直无司机接单，超时或中途用户主动取消打车，则记录**end_time-打车结束时间**。

- 若乘客上车前，乘客或司机点击取消订单，会将打车订单表对应订单的f**inish_time-订单完成时间**填充为取消时间，其余字段设为**null**。

- 当司机接上乘客时，填充订单表中该**start_time-开始计费的上车时间**。

- 当订单完成时填充订单完成时间、里程数、费用；评分设为**null**，在用户给司机打1~5星评价后填充。

**问题**：请统计每个城市中评分最高的司机平均评分、日均接单量和日均行驶里程数。

**注**：有多个司机评分并列最高时，都输出。

平均评分和日均接单量保留1位小数，

日均行驶里程数保留3位小数，按日均接单数升序排序。

示例数据的输出结果如下

| city | driver_id | avg_grade | avg_order_num | avg_mileage |
| ---- | --------- | --------- | ------------- | ----------- |
| 北京   | 203       | 4.8       | 1.7           | 14.700      |

解释：

示例数据中，在北京市，共有2个司机接单，202的平均评分为4.3，203的平均评分为4.8，因此北京的最高评分的司机为203；203的共在3天里接单过，一共接单5次（包含1次接单后未完成），因此日均接单数为1.7；总行驶里程数为44.1，因此日均行驶里程数为14.700。

## 答案

```sql
select city, driver_id, avg_grade, avg_order_num, avg_mileage
from (
select city, driver_id, round(avg_grade, 1) as avg_grade,
round(num_orders / num_days, 1) as avg_order_num,
round(total_mileage / num_days, 3) as avg_mileage,
rank() over(partition by city order by avg_grade desc) as rk
from ( 
## joined table
select driver_id, city, avg(grade) as avg_grade,
count(order_id) as num_orders,
sum(mileage) as total_mileage,
count(distinct date(order_time)) as num_days
from tb_get_car_record join tb_get_car_order using(order_id)
group by city, driver_id) as x 
## joined table
) as y
where rk = 1
order by avg_order_num
```

## 思路

这个问题并不复杂。

1️⃣ 首先明确，我们需要按司机的维度计算，所以主要计算的表是第二张表`tb_get_car_order`，因为里面有`driver_id`，以及司机每单的评分等其他信息。但是城市信息在第一张表里，所以要根据`order_id`来`join`一下两张表。我们只用到第一张表的城市这一个column，除此之外第一张表没有任何其他的作用。

2️⃣ `count(order_id)`得到总订单数，`sum(mileage)`得到总里程数，`count(distinct date(order_time))`得到接单天数，用`distinct`去重一天接多个订单的情况。最后再按`city`和`driver_id`进行group by即可。

```sql
(select driver_id, city, avg(grade) as avg_grade,
count(order_id) as num_orders,
sum(mileage) as total_mileage,
count(distinct date(order_time)) as num_days
from tb_get_car_record join tb_get_car_order using(order_id)
group by city, driver_id
) as x 
```

3️⃣ 下一步，用`rank()`窗口函数对`avg_grade`按城市维度进行排序。然后按题目要求计算各项指标即可。

```sql
(select city, driver_id, round(avg_grade, 1) as avg_grade,
round(num_orders / num_days, 1) as avg_order_num,
round(total_mileage / num_days, 3) as avg_mileage,
rank() over(partition by city order by avg_grade desc) as rk
from x
) as y
```

4️⃣ 最后选出每个城市中rank为1的row即可。

```sql
select city, driver_id, avg_grade, avg_order_num, avg_mileage
from y
where rk = 1
order by avg_order_num
```
