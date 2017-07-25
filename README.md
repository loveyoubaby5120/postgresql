* SYNTAX
    * [SQL](#SQL)
    * [json和jsonb的操作符](#OPERATOR)
    * [jsonb额外操作符](#JSONB)
    * [json创建函数](#JSONCREATE)
    * [json处理函数](#JSONFUN)

# <a name="SQL">SQL</a>
<br/>

```sql
# 查询jsonb 某个字段 并排序
# https://www.cnblogs.com/alianbog/p/5658156.html
SELECT "id", "subscribe":: json->0->'str' FROM product ORDER BY (subscribe->0->'str') desc;
SELECT "id", "subscribe":: json->0->'str' FROM product ORDER BY (subscribe->>'str') desc;

# 修改数组类型json
update product set subscribe = '[{"str": 1.1, "end": 2.2, "year_earnings_rate": 4.3, "commission": 4.4}]' where id='111';

```


<br/>
# <a name="OPERATOR">json和jsonb的操作符</a>
<br/>

|操作符|右操作符类型|描述|示例|结果|
|:---:|:---:|:---:|:---|:---:|
|->| int |获取JSON数组元素（索引从0开始）|<pre>`select '[{"a":"foo"},{"b":"bar"},{"c":"baz"}]'::json->2;`</pre>|<pre>`{"c":"baz"}`</pre>|
|->| text |通过键获取值|<pre>`select '{"a": {"b":"foo"}}'::json->'a';`</pre>|<pre>`{"b":"foo"}`</pre>|
|->>| int |获取JSON数组元素为 text|<pre>`select '[1,2,3]'::json->>2;`</pre>|<pre>`3`</pre>|
|->>| text |通过键获取值为text|<pre>`select '{"a":1,"b":2}'::json->>'b';`</pre>|<pre>`2`</pre>|
|#>| text[] |在指定的路径获取JSON对象|<pre>`select '{"a": {"b":{"c": "foo"}}}'::json#>'{a,b}';`</pre>|<pre>`{"c": "foo"}`</pre>|
|#>>| text[] |在指定的路径获取JSON对象为 text|<pre>`select '{"a":[1,2,3],"b":[4,5,6]}'::json#>>'{a,2}';`</pre>|<pre>`3`</pre>|


<br/>
# <a name="JSONB">jsonb额外操作符</a>
<br/>

|操作符|右操作符类型|描述|示例|结果|
|:---:|:---:|:---:|:---|:---:|
|@>| jsonb |左侧json最上层的值是否包含右边json对象|<pre>`select '{"a":{"b":2}}'::jsonb @> '{"b":2}'::jsonb;`<br/>`select '{"a":1, "b":2}'::jsonb @> '{"b":2}'::jsonb;`</pre>|<pre>`f`<br/>`t`</pre>|
|<@| jsonb |左侧json对象是否包含于右侧json最上层的值内|<pre>`select '{"b":2}'::jsonb <@ '{"a":1, "b":2}'::jsonb;`</pre>|<pre>`t`</pre>|
|?| text |text是否作为左侧Json对象最上层的键|<pre>`select '{"a":1, "b":2}'::jsonb ? 'b';`</pre>|<pre>`t`</pre>|
|?\|| text[] |text[]中的任一元素是否作为左侧Json对象最上层的键|<pre>`select '{"a":1, "b":2, "c":3}'::jsonb ?| array['b', 'c'];`</pre>|<pre>`t`</pre>|
|?&| text[] |text[]中的任一元素是否作为左侧Json对象最上层的键|<pre>`select '["a", "b"]'::jsonb ?& array['a', 'b'];`</pre>|<pre>`t`</pre>|
|\|\|| jsonb |连接两个json对象，组成一个新的json对象|<pre>`select '["a", "b"]'::jsonb || '["c", "d"]'::jsonb;`</pre>|<pre>`["a", "b",`<br/>` "c", "d"]`</pre>|
|-| text |删除左侧json对象中键为text的键值对|<pre>`select '{"a": "b"}'::jsonb - 'a';`</pre>|<pre>`{}`</pre>|
|-| integer |	删除数组指定索引处的元素，如果索引值为负数，则从右边计算索引值。<br/>如果最上层容器内不是数组，则抛出错误。|<pre>`select '["a", "b"]'::jsonb - 1;`</pre>|<pre>`["a"]`</pre>|
|#-| integer |删除指定路径下的域或元素（如果是json数组，且整数值是负的，则索引值从右边算起）|<pre>`select '["a", {"b":1}]'::jsonb #- '{1,b}';`</pre>|<pre>`["a", {}]`</pre>|


<br/>
# <a name="JSONCREATE">json创建函数</a>
<br/>

|函数|描述|示例|结果|
|:---:|:---:|:---|:---:|
|<pre>`to_json(anyelement)`<br/><br/>`to_jsonb(anyelement)`</pre>|返回json或jsonb类型的值。数组和复合被转换（递归）成数组和对象。另外除数字、布尔、NULL值（直接使用NULL抛出错误）外，其他标量必须有类型转换。（此处请参考原文）|<pre>`select to_json('3'::int);`</pre>|<pre>`3`</pre>|
|<pre>`array_to_json`<br/>`(anyarray[, `<br/>`pretty_bool])`</pre>|以JSON数组返回该数组。PostgreSQL多维数组变成JSON数组中的数组。如果pretty_bool 为真，则在维度1元素之间添加换行。|<pre>`select array_to_json('{{1,5},{99,100}}'::int[],true);`</pre>|<pre>`[[1,5], +`<br/>`[99,100]]`</pre>|
|<pre>`row_to_json(`<br/>`record [, `<br/>`pretty_bool])`</pre>|以JSON对象返回行。如果pretty_bool 为真，则在级别1元素之间添加换行。|<pre>`select row_to_json(row(1,'foo'),true);`</pre>|<pre>`{"f1":1, + `<br/>`"f2":"foo"}`</pre>|
|<pre>`json_build_array(`<br/>`VARIADIC "any")`<br/><br/>`jsonb_build_array(`<br/>`VARIADIC "any")`</pre>|建立一个由可变参数列表组成的不同类型的JSON数组|<pre>`select json_build_array(1,2,'3',4,5);`</pre>|<pre>`[1, 2, "3"`<br/>`, 4, 5]`</pre>|
|<pre>`json_build_object(`<br/>`VARIADIC "any")`<br/><br/>`jsonb_build_object(`<br/>`VARIADIC "any")`</pre>|建立一个由可变参数列表组成的JSON对象。参数列表参数交替转换为键和值。|<pre>`select json_build_object('foo',1,'bar',2);`</pre>|<pre>`{"foo" : 1, `<br/>`"bar" : 2}`</pre>|
|<pre>`json_object(text[])`<br/><br/>`jsonb_object(text[])`</pre>|根据text[]数组建立一个json对象，如果是一维数组，则必须有偶数个元素，元素交替组成键和值。如果是二维数组，则每个元素必须有2个元素，可以组成键值对。|<pre>`select json_object('{a, 1, b, "def", c, 3.5}');`<br/><br/>`select json_object('{{a, 1},{b, "def"},{c, 3.5}}');`</pre>|<pre>`{"a" : "1", `<br/>`"b" : "def", `<br/>`"c" : "3.5"}`</pre>|
|<pre>`json_object(`<br/>`keys text[], `<br/>`values text[])`<br/><br/>`jsonb_object(`<br/>`keys text[], `<br/>`values text[])`</pre>|分别从两组text[]中获取键和值，与一维数组类似。|<pre>`select json_object('{a, b}', '{1,2}');`</pre>|<pre>`{"a" : "1",`<br/>` "b" : "2"}`</pre>|


<br/>
# <a name="JSONFUN">json处理函数</a>
<br/>

|函数|返回类型|描述|示例|结果|
|:---:|:---:|:---:|:---|:---:|
|<pre>`json_array_length(json)`<br/><br/>`jsonb_array_length(jsonb)`</pre>|int|返回Json数组最外层元素个数|<pre>`select json_array_length(`<br/>`'[1,2,3,{"f1":1,"f2":[5,6]},4]');`</pre>|<pre>`5`</pre>|
|<pre>`json_each(json)`<br/><br/>`jsonb_each(jsonb)`</pre>|setof key text, value json<br/><br/>setof key text, value jsonb|将最外层Json对象转换为键值对集合|<pre>`select json_each('{"a":"foo", "b":"bar"}');`</pre>|<pre>`(a,"""foo""")`<br/><br/>`(b,"""bar""")`</pre>|
|<pre>`json_each_text(json)`<br/><br/>`jsonb_each_text(jsonb)`</pre>|setof key text, value text|将最外层Json对象转换为键值对集合，且value为text类型|<pre>`select json_each_text('{"a":"foo", "b":"bar"}');`</pre>|<pre>`(a,foo)`<br/><br/>`(b,bar)`</pre>|
|<pre>`json_extract_path(`<br/>`from_json json,VARIADIC path_elems text[])`<br/><br/>`jsonb_extract_path(from_json jsonb,VARIADIC path_elems text[])`</pre>|json<br/><br/>jsonb|返回path_elems指向的value，同操作符#>|<pre>`select json_extract_path('{"f2":{"f3":1},"f4":{"f5":99,"f6":"foo"}}','f4');`</pre>|<pre>`{"f5":99,"f6":"foo"}`</pre>|



















