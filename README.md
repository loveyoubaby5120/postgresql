* SYNTAX
    * [JSON](#JSON)

# <a name="JSON">JSON</a>

```sql
# 查询jsonb 某个字段 并排序
# https://www.cnblogs.com/alianbog/p/5658156.html
SELECT "id", "subscribe":: json->0->'str' FROM product ORDER BY (subscribe->0->'str') desc;
SELECT "id", "subscribe":: json->0->'str' FROM product ORDER BY (subscribe->>'str') desc;

# 修改数组类型json
update product set subscribe = '[{"str": 1.1, "end": 2.2, "year_earnings_rate": 4.3, "commission": 4.4}]' where id='111';

```
