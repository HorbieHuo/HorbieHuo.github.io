---
title: 多表连接查询统计sql示例
date: 2017-04-27 08:49:44
tags:
---

**背景：** 一个电商类似的项目，这个是后台统计使用的sql，之前使用的sqlalchemy ，但是运行的时间太长，400条记录，运行时间在1分钟左右，使用sql后运行时间在3秒左右

下面是示例的sql语句
```sql
SELECT 
table_1.id AS table_1_id, 
table_1.item_id AS table_1_item_id, 
table_1.sku_no AS table_1_sku_no, 
table_1.sku_name AS table_1_sku_name, 
table_1.MDM_code AS table_1_MDM_code,
table_1.created_at AS table_1_created_at, 
table_1.created_by AS table_1_created_by, 
table_2.id AS item_id, 
table_2.name AS item_name, 
table_2.item_no AS item_item_no, 
table_2.item_type AS item_item_type, 
table_2.price AS item_price, 
count(DISTINCT table_3.id) AS browse_times, /*用 DISTINCT 去重，统计数量*/
count(DISTINCT table_4.id) AS like_times, 
count(DISTINCT table_5.id) AS item_share_times, 
count(DISTINCT table_6.id) AS add_cart_times, 
count(DISTINCT table_7.id) AS table_7_times, 
count(DISTINCT table_8.id) AS comment_times, 
count(DISTINCT table_9.id) AS table_9_order_count,
sum(table_9.cnt)/(count(table_9.order_id)/count(distinct table_9.order_id)) AS table_9_sku_count,
sum(table_9.cnt * table_9.price)/(count(table_9.order_id)/count(distinct table_9.order_id)) AS table_9_total_price
/* sum算总和的时候除掉重复的次数，目前没有找到什么更好的办法来解决 */

FROM table_1 JOIN table_2 ON table_1.item_id = table_2.id

LEFT OUTER JOIN table_3 ON table_3.detail = table_1.item_id AND  table_3.act_time BETWEEN '1487981301.0' AND '1493165301.0' 
LEFT OUTER JOIN table_4 ON table_4.detail = table_1.item_id AND  table_4.act_time BETWEEN '1487981301.0' AND '1493165301.0' 
LEFT OUTER JOIN table_5 ON table_5.detail = table_1.item_id AND  table_5.act_time BETWEEN '1487981301.0' AND '1493165301.0' 
LEFT OUTER JOIN table_6 ON table_6.detail = table_1.item_id AND  table_6.act_time BETWEEN '1487981301.0' AND '1493165301.0' 
LEFT OUTER JOIN table_8 ON table_8.item_id = table_1.item_id AND  table_8.created_at BETWEEN '2017-02-25 08:08:21' AND '2017-04-26 08:08:21' 
LEFT OUTER JOIN table_7 ON table_7.sku_id = table_1.id AND  table_7.created_at BETWEEN '2017-02-25 08:08:21' AND '2017-04-26 08:08:21' 
LEFT OUTER JOIN table_9 ON table_9.sku_id = table_1.id AND table_9.order_id in 
    (select table_10.id from table_10 WHERE table_10.created_at BETWEEN '2017-02-25 08:08:21' AND '2017-04-26 08:08:21')

GROUP BY table_1.id;
```

