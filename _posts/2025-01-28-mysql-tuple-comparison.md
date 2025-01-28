---
title: " 📘 [en] MySQL Tuple Comparison (Row Constructor): Use with Caution "

categories:
  - MySQL
  - Database
layout: single
classes: wide
last_modified_at: now
---

**Using tuple comparison-based queries in MySQL can cause unexpected performance issues. (very slow)**

<div style="text-align: center;">
    <img src="https://github.com/user-attachments/assets/d4b2d5cf-ecbf-4a22-9c2e-505ac5d08129" alt="image" width="300">
</div>

#### Row Constructor

- In MySQL, you can group multiple columns into a single tuple-like value using the ROW(col1, col2, col3, ...) or (col1, col2, ...) syntax for comparison.

```sql
SELECT *
FROM your_table_name
WHERE (col1, col2) = (x, y);
```

This method can make your queries easier to read, especially when handling with complex conditions involving multiple columns.

However, such queries can become very slow.

- [Related report on MySQL](https://bugs.mysql.com/bug.php?id=111952) 
  - An issue where using tuple comparison causes indexes to be ignored, leading to a Full Table Scan.

Let's look an example.

- Table Schema

```sql
create table comment
(
  comment_id        bigint        not null
    primary key,
  content           varchar(3000) not null,
  article_id        bigint        not null,
  parent_comment_id bigint        not null,
  writer_id         bigint        not null,
  is_deleted        tinyint(1)    not null,
  created_at        datetime      not null
);

create index idx_article_id_parent_comment_id_comment_id
  on comment (article_id, parent_comment_id, comment_id);
```

- mysql base image: mysql:8.0.38
- test dataset: ~= 8 million rows

#### Case 1. tuple comparison (slow case)

```sql
explain analyze
select comment.comment_id,
       comment.parent_comment_id,
       comment.article_id,
       comment.writer_id,
       comment.content,
       comment.is_deleted,
       comment.created_at
from comment
where article_id = 1
  and (parent_comment_id, comment_id) > (142539921307124354, 142539921307124350)
order by parent_comment_id, comment_id
limit 30;
```


- Result
```sql
-> Limit: 30 row(s)  (cost=542979 rows=30) (actual time=8620..8620 rows=30 loops=1)
-> Filter: ((`comment`.comment_id,`comment`.parent_comment_id) > (142539921307124354,142539921307124350))  (cost=542979
rows=4.01e+6) (actual time=8620..8620 rows=30 loops=1)
-> Index lookup on comment using idx_article_id_parent_comment_id_comment_id (article_id=1)  (cost=542979
rows=4.01e+6) (actual time=1.83..8251 rows=8e+6 loops=1)
```

As shown, an index full scan occurs. It scans all 8 million rows and takes about 8 seconds to run the query.

The tuple comparison (a, b) > (x, y) can be decomposed into (a > x) OR (a = x AND b > y).


### Case 2. Decomposed Conditions (fast case)

```sql
explain analyze
select comment.comment_id,
       comment.parent_comment_id,
       comment.article_id,
       comment.writer_id,
       comment.content,
       comment.is_deleted,
       comment.created_at
from comment
where article_id = 1
  and (
  parent_comment_id > 142539921307124354
    or
  (parent_comment_id = 142539921307124354 and comment_id > 142539921307124350)
  )
order by parent_comment_id, comment_id
limit 30;
```


- Result
```sql
-> Limit: 30 row(s)  (cost=416 rows=30) (actual time=0.252..0.727 rows=30 loops=1)
-> Index range scan on comment using idx_article_id_parent_comment_id_comment_id over (article_id = 1 AND
parent_comment_id = 142539921307124354 AND 142539921307124350 < comment_id) OR (article_id = 1 AND 142539921307124354 <
parent_comment_id), with index condition: ((`comment`.article_id = 1) and ((`comment`.parent_comment_id >
142539921307124354) or ((`comment`.parent_comment_id = 142539921307124354) and (`comment`.comment_id >
142539921307124350))))  (cost=416 rows=358) (actual time=0.232..0.705 rows=30 loops=1)
```

MySQL performs an Index Range Scan, making the query run fast in under 0.7 seconds.


#### Conclusion

- Use Explicit Conditions
  - Avoid using Row Constructors for tuple comparisons with inequalities. Instead, decompose the conditions. This helps MySQL’s optimizer use indexes effectively.
- Always use EXPLAIN to understand query plan
  - Before using any query, use EXPLAIN ANALYZE to see the query plan.
  - Ensure that indexes are being used as expected.

By following these tips, I hope you can prevent performance issues related to tuple comparisons in MySQL.
