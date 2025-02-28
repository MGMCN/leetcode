# [2494. 合并在同一个大厅重叠的活动](https://leetcode.cn/problems/merge-overlapping-events-in-the-same-hall)

[English Version](/solution/2400-2499/2494.Merge%20Overlapping%20Events%20in%20the%20Same%20Hall/README_EN.md)

## 题目描述

<!-- 这里写题目描述 -->

<p>表: <code>HallEvents</code></p>

<pre>
+-------------+------+
| Column Name | Type |
+-------------+------+
| hall_id     | int  |
| start_day   | date |
| end_day     | date |
+-------------+------+
该表中没有主键。它可能包含重复字段。
该表的每一行表示活动的开始日期和结束日期，以及活动举行的大厅。
</pre>

<p><br />
编写一个 SQL 查询来合并在&nbsp;<strong>同一个大厅举行&nbsp;</strong>的所有重叠活动。如果两个活动&nbsp;<strong>至少有一天&nbsp;</strong>相同，那么它们就是重叠的。</p>

<p data-group="1-1">以<strong>任意顺序</strong>返回结果表。</p>

<p>查询结果格式如下所示。</p>

<p>&nbsp;</p>

<p><strong>示例 1:</strong></p>

<pre>
<strong>输入:</strong> 
HallEvents 表:
+---------+------------+------------+
| hall_id | start_day  | end_day    |
+---------+------------+------------+
| 1       | 2023-01-13 | 2023-01-14 |
| 1       | 2023-01-14 | 2023-01-17 |
| 1       | 2023-01-18 | 2023-01-25 |
| 2       | 2022-12-09 | 2022-12-23 |
| 2       | 2022-12-13 | 2022-12-17 |
| 3       | 2022-12-01 | 2023-01-30 |
+---------+------------+------------+
<strong>输出:</strong> 
+---------+------------+------------+
| hall_id | start_day  | end_day    |
+---------+------------+------------+
| 1       | 2023-01-13 | 2023-01-17 |
| 1       | 2023-01-18 | 2023-01-25 |
| 2       | 2022-12-09 | 2022-12-23 |
| 3       | 2022-12-01 | 2023-01-30 |
+---------+------------+------------+
<strong>解释:</strong> 有三个大厅。
大厅 1:
- 两个活动 ["2023-01-13", "2023-01-14"] 和 ["2023-01-14", "2023-01-17"] 重叠。我们将它们合并到一个活动中 ["2023-01-13", "2023-01-17"]。
- 活动 ["2023-01-18", "2023-01-25"] 不与任何其他活动重叠，所以我们保持原样。
大厅 2:
- 两个活动 ["2022-12-09", "2022-12-23"] 和 ["2022-12-13", "2022-12-17"] 重叠。我们将它们合并到一个活动中 ["2022-12-09", "2022-12-23"]。
大厅 3:
- 大厅只有一个活动，所以我们返回它。请注意，我们只分别考虑每个大厅的活动。</pre>

## 解法

<!-- 这里可写通用的实现逻辑 -->

<!-- tabs:start -->

### **SQL**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```sql
# Write your MySQL query statement below
WITH
    S AS (
        SELECT
            hall_id,
            start_day,
            end_day,
            max(end_day) OVER (
                PARTITION BY hall_id
                ORDER BY start_day
            ) AS cur_max_end_day
        FROM HallEvents
    ),
    T AS (
        SELECT
            *,
            if(
                start_day <= lag(cur_max_end_day) OVER (
                    PARTITION BY hall_id
                    ORDER BY start_day
                ),
                0,
                1
            ) AS start
        FROM S
    ),
    P AS (
        SELECT
            *,
            sum(start) OVER (
                PARTITION BY hall_id
                ORDER BY start_day
            ) AS gid
        FROM T
    )
SELECT hall_id, min(start_day) AS start_day, max(end_day) AS end_day
FROM P
GROUP BY hall_id, gid;
```

<!-- tabs:end -->
