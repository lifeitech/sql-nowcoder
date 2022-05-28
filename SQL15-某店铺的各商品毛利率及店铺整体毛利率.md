# SQL15 某店铺的各商品毛利率及店铺整体毛利率

中等  通过率：24.96%  时间限制：1秒  空间限制：256M

## 描述

商品信息表tb_product_info

| id  | product_id | shop_id | tag  | in_price | quantity | release_time        |
| --- | ---------- | ------- | ---- | -------- | -------- | ------------------- |
| 1   | 8001       | 901     | 家电   | 6000     | 100      | 2020-01-01 10:00:00 |
| 2   | 8002       | 902     | 家电   | 12000    | 50       | 2020-01-01 10:00:00 |
| 3   | 8003       | 901     | 3C数码 | 12000    | 50       | 2020-01-01 10:00:00 |

（product_id-商品ID, shop_id-店铺ID, tag-商品类别标签, in_price-进货价格, quantity-进货数量, release_time-上架时间）  

订单总表tb_order_overall

| id  | order_id | uid | event_time          | total_amount | total_cnt | status |
| --- | -------- | --- | ------------------- | ------------ | --------- | ------ |
| 1   | 301001   | 101 | 2021-10-01 10:00:00 | 30000        | 3         | 1      |
| 2   | 301002   | 102 | 2021-10-01 11:00:00 | 23900        | 2         | 1      |
| 3   | 301003   | 103 | 2021-10-02 10:00:00 | 31000        | 2         | 1      |

（order_id-订单号, uid-用户ID, event_time-下单时间, total_amount-订单总金额, total_cnt-订单商品总件数, status-订单状态）

订单明细表tb_order_detail

| id  | order_id | product_id | price | cnt |
| --- | -------- | ---------- | ----- | --- |
| 1   | 301001   | 8001       | 8500  | 2   |
| 2   | 301001   | 8002       | 15000 | 1   |
| 3   | 301002   | 8001       | 8500  | 1   |
| 4   | 301002   | 8002       | 16000 | 1   |
| 5   | 301003   | 8002       | 14000 | 1   |
| 6   | 301003   | 8003       | 18000 | 1   |

（order_id-订单号, product_id-商品ID, price-商品单价, cnt-下单数量）

**场景逻辑说明**：

- 用户将购物车中多件商品一起下单时，订单总表会生成一个订单（但此时未付款，**status-订单状态**为**0**表示待付款），在订单明细表生成该订单中每个商品的信息；

- 当用户支付完成时，在订单总表修改对应订单记录的**status-订单状态**为**1**表示已付款；

- 若用户退货退款，在订单总表生成一条交易总金额为负值的记录（表示退款金额，订单号为退款单号，**status-订单状态**为2表示已退款）。

**问题**：请计算2021年10月以来店铺901中商品毛利率大于24.9%的商品信息及店铺整体毛利率。

**注**：商品毛利率=(1-进价/平均单件售价)*100%；

店铺毛利率=(1-总进价成本/总销售收入)*100%。

结果先输出店铺毛利率，再按商品ID升序输出各商品毛利率，均保留1位小数。

**输出示例**：

示例数据的输出结果如下：

| product_id | profit_rate |
| ---------- | ----------- |
| 店铺汇总       | 31.0%       |
| 8001       | 29.4%       |
| 8003       | 33.3%       |

解释：

店铺901有两件商品8001和8003；8001售出了3件，销售总额为25500，进价总额为18000，毛利率为1-18000/25500=29.4%，8003售出了1件，售价为18000，进价为12000，毛利率为33.3%；

店铺卖出的这4件商品总销售额为43500，总进价为30000，毛利率为1-30000/43500=31.0%

[题目链接](https://www.nowcoder.com/practice/65de67f666414c0e8f9a34c08d4a8ba6)

## 答案

```sql
with y as
(select p2.product_id, p2.in_price, x2.Q, x2.gr_rev from
(select x.product_id, sum(x.cnt) as Q, sum(x.price * x.cnt) as gr_rev
from
(select p.shop_id, p.product_id, detail.price, detail.cnt, t.order_id, t.event_time, t.status 
from
tb_product_info as p
left join
tb_order_detail as detail
on p.product_id = detail.product_id
right join tb_order_overall as t
on detail.order_id = t.order_id
where p.shop_id = 901 and t.event_time >= '2021-10-01' and t.status = 1) as x
group by x.product_id) as x2
join (select product_id, in_price from tb_product_info) as p2
on x2.product_id = p2.product_id
)
### end of the y table
select '店铺汇总' as product_id, concat(round((1 - sum(y.in_price * y.Q) / sum(y.gr_rev)) * 100, 1), '%') as profit_rate
from y
union all
(select y.product_id, concat(round((1 - y.in_price * y.Q / y.gr_rev) * 100, 1), '%') as profit_rate
from y having cast(profit_rate as float) > 24.9 order by y.product_id)
```

## 思路

这道题通过率非常低，因为难度极大。我的答案有二十多行，是我目前写过的最复杂的query。

其实主体部分难度并不算大，但有两个要求让这道题变得尤其难以通过：

1. 计算的各店铺的毛利率表格前面还要加一行”店铺汇总“，这个就陡然增加了不少难度。我的答案中就不得不用 with statement，最后用union all把两个表格合起来；

2. 牛客网后台默认设置的是 `sql_mode=only_full_group_by`，所以group by的时候选择商品信息表的进价`in_price`会报错，只能group by之后再select一次，也大幅增加了调试代码的时长。自己平时工作中应该完全没有必要这样，况且本题中每件商品只有一个进价，不存在ambiguity。所以实际中用
   
   ```sql
   set global sql_mode=(select replace(@@sql_mode,'ONLY_FULL_GROUP_BY',''));
   ```
   
   命令把strict mode关掉就好。

除了这两点要求很变态，主体部分还好，难度中等，思路跟

[SQL35 浙大不同难度题目的正确率](/SQL35-浙大不同难度题目的正确率.md)

这道题是相似的。接下来我们一步一步来走。

1️⃣ 我们想要计算的是每件商品的毛利率。商品的基本信息在`tb_product_info`里面。所以我们要以这张表的`product_id`为单位进行group by。

2️⃣ `tb_order_detail`是流水，有售价和销售数量的数据，是我们要进行计算的主要表格。

3️⃣第三张表`tb_order_overall`主要是用来filter题目中的各种条件，没别的用。

4️⃣ 综上，答案的骨架如下。题目其他一些各种condition比较annoying，没什么卵用，就省略去了。

```sql
select x.product_id, 1- x.in_price * sum(x.cnt) / sum(x.price * x.cnt) as profit_rate
from
(select p.shop_id, p.product_id, p.in_price, detail.price, detail.cnt, t.order_id, t.event_time, t.status 
from
tb_product_info as p
left join
tb_order_detail as detail
on p.product_id = detail.product_id
right join tb_order_overall as t
on detail.order_id = t.order_id
where p.shop_id = 901 and t.event_time >= '2021-10-01' and t.status = 1) as x
group by x.product_id
```

这个query就可以得到每件商品的毛利率了。毛利率根据题目给定的方法计算即可，即

```sql
1- x.in_price * sum(x.cnt) / sum(x.price * x.cnt)
```

其实写到这里，我觉得就可以了，目的就基本达到了，可以不必再往下继续深究以达到提交通过的要求。

我们可以看到，这道题思路跟上面浙大不同题目正确率的题目是完全一样的。都是有个要进行计算的主要的流水表格，以及其他两个有数据label的表格。我们把三个表格根据每个表格的性质join到一起，得到需要的全部的column，再group by即可。

### 进阶挑战 👑

如果要按题目要求，还要加入一行店铺总体的毛利率，那么我们就需要对上面这个derived table计算两次。这样就要用到 with statement

```sql
with tb as (...)
select col1, col2... from tb
union all
select col1, col2... from tb
...
```

另外，总体的毛利率等于$1-$总成本/总销售额，所以要计算总体的，我们要改一下derived table返回的column，先不直接计算每种商品的毛利率，而是先得到每种商品总成本和总销售额的数据：

| product_id | gross cost | gross revenue |
| ---------- | ---------- | ------------- |
| 8001       | ...        | ...           |
| 8003       | ...        | ...           |

然后每种商品的毛利率用 $1$ 减去两个column相除即可，店铺总体的就先把两个column `sum` 起来再用$1$减即可，即

```sql
1 - sum(gross_cost) / sum(gross_revenue)
```

那么到这一步，答案就是

```sql
with y as
(select x.product_id, x.in_price * sum(x.cnt) as gr_cost, sum(x.price * x.cnt) as gr_rev
from
(select p.shop_id, p.product_id, p.in_price, detail.price, detail.cnt, t.order_id, t.event_time, t.status 
from
tb_product_info as p
left join
tb_order_detail as detail
on p.product_id = detail.product_id
right join tb_order_overall as t
on detail.order_id = t.order_id
where p.shop_id = 901 and t.event_time >= '2021-10-01' and t.status = 1) as x
group by x.product_id
)
### end of the y table
select '店铺汇总' as product_id, concat(round((1 - sum(y.gr_cost) / sum(y.gr_rev)) * 100, 1), '%') as profit_rate
from y
union all
(select y.product_id, concat(round((1 - y.gr_cost / y.gr_rev) * 100, 1), '%') as profit_rate
from y order by y.product_id)
```

### group by 问题解决 💀

将上面的答案在网站上提交会报错，原因是我们select了`x.in_price`这个column，而我们group by的时候并没有对其进行aggregate。这就是为什么我提供的答案中有`x2`和`p2`这样的表。为了circumvent这个报错，就不得不再进行一遍join和select，十分费劲！如上面提到的，真是没必要较这个真。实际中用

```sql
set global sql_mode=(select replace(@@sql_mode,'ONLY_FULL_GROUP_BY',''));
```

关掉strict mode就行。


