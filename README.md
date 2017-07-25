* SYNTAX
    * [JSON](#JSON)
    * [json和jsonb的操作符](#OPERATOR)

# <a name="JSON">JSON</a>

```sql
# 查询jsonb 某个字段 并排序
# https://www.cnblogs.com/alianbog/p/5658156.html
SELECT "id", "subscribe":: json->0->'str' FROM product ORDER BY (subscribe->0->'str') desc;
SELECT "id", "subscribe":: json->0->'str' FROM product ORDER BY (subscribe->>'str') desc;

# 修改数组类型json
update product set subscribe = '[{"str": 1.1, "end": 2.2, "year_earnings_rate": 4.3, "commission": 4.4}]' where id='111';

```

# <a name="OPERATOR">json和jsonb的操作符</a>

|操作符|右操作符类型|描述|示例|结果|
|:---:|:---:|:---:|:---:|:---:|
|->|int|获取JSON数组元素（索引从0开始）|<pre>`select '[{"a":"foo"},{"b":"bar"},{"c":"baz"}]'::json->2;`</pre>|<pre>`{"c":"baz"}`</pre>|

<pre class="sql"><code class="sql">select '[{"a":"foo"},{"b":"bar"},{"c":"baz"}]'::json->2;</code></pre>

<table class="table table-bordered table-striped table-condensed">  
    <thead>
        <tr>
            <th>操作符</th>
            <th>右操作符类型</th>
            <th>描述</th>
            <th>示例</th>
            <th>结果</th>
        </tr>
    </thead>
    <tr>  
        <td>-></td>  
        <td>int</td>  
        <td>获取JSON数组元素（索引从0开始）</td>
        <td>
            <pre><code>select '[{"a":"foo"},{"b":"bar"},{"c":"baz"}]'::json->2;</code></pre>
        </td>  
        <td><pre><code>{"c":"baz"}</code></pre></td>  
    </tr>
</table>  
