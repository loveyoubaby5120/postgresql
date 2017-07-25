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
|:---:|:---:|:---:|:---|:---:|
|->| int |获取JSON数组元素（索引从0开始）|<pre>`select '[{"a":"foo"},{"b":"bar"},{"c":"baz"}]'::json->2;`</pre>|<pre>`{"c":"baz"}`</pre>|
|->| text |通过键获取值|<pre>`select '{"a": {"b":"foo"}}'::json->'a';`</pre>|<pre>`{"b":"foo"}`</pre>|
|->>| int |获取JSON数组元素为 text|<pre>`select '[1,2,3]'::json->>2;`</pre>|<pre>`3`</pre>|
|->>| text |通过键获取值为text|<pre>`select '{"a":1,"b":2}'::json->>'b';`</pre>|<pre>`2`</pre>|
|#>| text[] |在指定的路径获取JSON对象|<pre>`select '{"a": {"b":{"c": "foo"}}}'::json#>'{a,b}';`</pre>|<pre>`{"c": "foo"}`</pre>|
|#>>| text[] |在指定的路径获取JSON对象为 text|<pre>`select '{"a":[1,2,3],"b":[4,5,6]}'::json#>>'{a,2}';`</pre>|<pre>`3`</pre>|


<table class="table table-bordered table-striped table-condensed">  
    <thead>
        <tr>
            <th width="10%">操作符</th>
            <th width="15%">右操作符类型</th>
            <th width="25%">描述</th>
            <th width="30%">示例</th>
            <th width="20%">结果</th>
        </tr>
    </thead>
    <tbody>
        <tr>  
            <td align=center>-></td>  
            <td align=center>int</td>  
            <td align=center>获取JSON数组元素（索引从0开始）</td>
            <td align=left>
                <pre><code>select '[{"a":"foo"},{"b":"bar"},{"c":"baz"}]'::json->2;</code></pre>
            </td>  
            <td align=center><pre><code>{"c":"baz"}</code></pre></td>  
        </tr>
        <tr>  
            <td align=center>-></td>  
            <td align=center>text</td>  
            <td align=center>通过键获取值</td>
            <td align=left>
                <pre><code>select '{"a": {"b":"foo"}}'::json->'a';</code></pre>
            </td>  
            <td align=center><pre><code>{"b":"foo"}`````````````````````````````````</code></pre></td>  
        </tr>
    </tbody>
</table>  
