* SYNTAX
    * [SQL](#SQL)
    * [json和jsonb的操作符](#OPERATOR)
    * [jsonb额外操作符](#JSONB)
    * [json创建函数](#JSONCREATE)
    * [json处理函数](#JSONFUN)

# <a name="SQL">SQL</a>

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


# <a name="JSONB">jsonb额外操作符</a>

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


# <a name="JSONCREATE">json创建函数</a>

|函数|描述|示例|结果|
|:---:|:---:|:---|:---:|
|<pre>`to_json(anyelement)`<br/><br/>`to_jsonb(anyelement)`</pre>|返回json或jsonb类型的值。数组和复合被转换（递归）成数组和对象。另外除数字、布尔、NULL值（直接使用NULL抛出错误）外，其他标量必须有类型转换。（此处请参考原文）|<pre>`select to_json('3'::int);`</pre>|<pre>`3`</pre>|
|<pre>`array_to_json`<br/>`(anyarray[, `<br/>`pretty_bool])`</pre>|以JSON数组返回该数组。PostgreSQL多维数组变成JSON数组中的数组。如果pretty_bool 为真，则在维度1元素之间添加换行。|<pre>`select array_to_json('{{1,5},{99,100}}'::int[],true);`</pre>|<pre>`[[1,5], +`<br/>`[99,100]]`</pre>|
|<pre>`row_to_json(`<br/>`record [, `<br/>`pretty_bool])`</pre>|以JSON对象返回行。如果pretty_bool 为真，则在级别1元素之间添加换行。|<pre>`select row_to_json(row(1,'foo'),true);`</pre>|<pre>`{"f1":1, + `<br/>`"f2":"foo"}`</pre>|
|<pre>`json_build_array(`<br/>`VARIADIC "any")`<br/><br/>`jsonb_build_array(`<br/>`VARIADIC "any")`</pre>|建立一个由可变参数列表组成的不同类型的JSON数组|<pre>`select json_build_array(1,2,'3',4,5);`</pre>|<pre>`[1, 2, "3"`<br/>`, 4, 5]`</pre>|
|<pre>`json_build_object(`<br/>`VARIADIC "any")`<br/><br/>`jsonb_build_object(`<br/>`VARIADIC "any")`</pre>|建立一个由可变参数列表组成的JSON对象。参数列表参数交替转换为键和值。|<pre>`select json_build_object('foo',1,'bar',2);`</pre>|<pre>`{"foo" : 1, `<br/>`"bar" : 2}`</pre>|
|<pre>`json_object(text[])`<br/><br/>`jsonb_object(text[])`</pre>|根据text[]数组建立一个json对象，如果是一维数组，则必须有偶数个元素，元素交替组成键和值。如果是二维数组，则每个元素必须有2个元素，可以组成键值对。|<pre>`select json_object('{a, 1, b, "def", c, 3.5}');`<br/><br/>`select json_object('{{a, 1},{b, "def"},{c, 3.5}}');`</pre>|<pre>`{"a" : "1", `<br/>`"b" : "def", `<br/>`"c" : "3.5"}`</pre>|
|<pre>`json_object(`<br/>`keys text[], `<br/>`values text[])`<br/><br/>`jsonb_object(`<br/>`keys text[], `<br/>`values text[])`</pre>|分别从两组text[]中获取键和值，与一维数组类似。|<pre>`select json_object('{a, b}', '{1,2}');`</pre>|<pre>`{"a" : "1",`<br/>` "b" : "2"}`</pre>|


# <a name="JSONFUN">json处理函数</a>

|函数|返回类型|描述|示例|结果|
|:---:|:---:|:---:|:---|:---:|
|<pre>`json_array_length(json)`<br/><br/>`jsonb_array_length(jsonb)`</pre>|int|返回Json数组最外层元素个数|<pre>`select json_array_length(`<br/>`'[1,2,3,{"f1":1,"f2":[5,6]},4]');`</pre>|<pre>`5`</pre>|
|<pre>`json_each(json)`<br/><br/>`jsonb_each(jsonb)`</pre>|setof key text, value json<br/><br/>setof key text, value jsonb|将最外层Json对象转换为键值对集合|<pre>`select json_each('{"a":"foo", "b":"bar"}');`</pre>|<pre>`(a,"""foo""")`<br/><br/>`(b,"""bar""")`</pre>|
|<pre>`json_each_text(json)`<br/><br/>`jsonb_each_text(jsonb)`</pre>|setof key text, value text|将最外层Json对象转换为键值对集合，且value为text类型|<pre>`select json_each_text('{"a":"foo", "b":"bar"}');`</pre>|<pre>`(a,foo)`<br/><br/>`(b,bar)`</pre>|
|<pre>`json_extract_path(`<br/>`from_json json,`<br/>`VARIADIC `<br/>`path_elems text[])`<br/><br/>`jsonb_extract_path(`<br/>`from_json jsonb,`<br/>`VARIADIC `<br/>`path_elems text[])`</pre>|json<br/><br/>jsonb|返回path_elems指向的value，同操作符#>|<pre>`select json_extract_path(`<br/>`'{"f2":{"f3":1},`<br/>`"f4":{"f5":99,"f6":"foo"}}',`<br/>`'f4');`</pre>|<pre>`{"f5":99,`<br/>`"f6":"foo"}`</pre>|
|<pre>`json_extract_path_text(`<br/>`from_json json,`<br/>`VARIADIC `<br/>`path_elems text[])`<br/><br/>`jsonb_extract_path_text(`<br/>`from_json jsonb,`<br/>`VARIADIC `<br/>`path_elems text[])`</pre>|text|返回path_elems指向的value，并转为text类型，同操作符#>>|<pre>`select json_extract_path_text(`<br/>`'{"f2":{"f3":1},`<br/>`"f4":{"f5":99,"f6":"foo"}}',`<br/>`'f4', 'f6');`</pre>|<pre>`foo`</pre>|
|<pre>`json_object_keys(json)`<br/><br/>`jsonb_object_keys(jsonb)`</pre>|setof text|返回json对象最外层的key|<pre>`select json_object_keys(`<br/>`'{"f1":"abc",`<br/>`"f2":{"f3":"a", "f4":"b"}}');`</pre>|<pre>`f1`<br/>`f2`</pre>|
|<pre>`json_populate_record(`<br/>`base anyelement, `<br/>`from_json json)`<br/><br/>`jsonb_populate_record(`<br/>`base anyelement, `<br/>`from_json jsonb)`</pre>|anyelement|将json对象的value以base定义的行类型返回，如果行类型字段比json对象键值少，则多出的键值将被抛弃；如果行类型字段多，则多出的字段自动填充NULL。|<pre>`表tbl_test定义：`<br/>`Table"public.tbl_test"`<br/>` Column | Type | Modifiers`<br/>` -----------+---------------------------+`<br/>`-----------`<br/>`a | bigint | b | character varying(32) | `<br/>`c | character varying(32) |`<br><br>`select * from json_populate_record(`<br/>`null::tbl_test, '{"a":1,"b":2}');`</pre>|<pre>`a |  b |  c `<br/>`---+---+------ `<br/>` 1 | 2  | NULL`</pre>|
|<pre>`json_populate_recordset(`<br/>`base anyelement, `<br/>`from_json json)`<br/><br/>`jsonb_populate_recordset(`<br/>`base anyelement, `<br/>`from_json jsonb)`</pre>| setof anyelement|将json对象最外层数组以base定义的行类型返回|<pre>`表定义同上`<br/>`select * from json_populate_recordset(`<br/>`null::tbl_test, `<br/>`'[{"a":1,"b":2},{"a":3,"b":4}]');`</pre>|<pre>`a |  b |  c `<br/>`---+---+------ `<br/>` 1 | 2  | NULL`<br/>` 3 | 4  | NULL`</pre>|
|<pre>`json_array_elements(`<br/>`json)`<br/><br/>`jsonb_array_elements(`<br/>`jsonb)`</pre>| setof json <br/><br/> setof jsonb|将json数组转换成json对象value的集合|<pre>`select json_array_elements(`<br/>`'[1,true, [2,false]]');`</pre>|<pre>`1`<br/>`true`<br/>`[2,false]`</pre>|
|<pre>`json_array_elements_text(json)`<br/><br/>`jsonb_array_elements_text(jsonb)`</pre>| setof text|将json数组转换成text的value集合|<pre>`select json_array_elements_text(`<br/>`'["foo", "bar"]');`</pre>|<pre>`foo`<br/>`bar`</pre>|
|<pre>`json_typeof(json)`<br/><br/>`jsonb_typeof(jsonb)`</pre>|text|返回json最外层value的数据类型，可能的类型有<br/>object, array, string, number, boolean, 和null.|<pre>`select json_typeof('-123.4')`</pre>|<pre>`number`</pre>|
|<pre>`json_to_record(json)`<br/><br/>`jsonb_to_record(jsonb)`</pre>| record |根据json对象创建一个record类型记录，所有的函数都返回record类型，所以必须使用as明确定义record的结构。|<pre>`select * from json_to_record(`<br/>`'{"a":1,"b":[1,2,3],"c":"bar"}') `<br/>`as x(a int, b text, d text);`</pre>|<pre>`a |    b    |   d `<br/>`---+---------+------`<br/>`1 | [1,2,3] | NULL`</pre>|
|<pre>`json_to_recordset(json)`<br/><br/>`jsonb_to_recordset(jsonb)`</pre>| setof record |根据json数组创建一个record类型记录，所有的函数都返回record类型，所以必须使用as明确定义record的结构。|<pre>`select * from json_to_recordset(`<br/>`'[{"a":1,"b":"foo"},`<br/>`{"a":"2","c":"bar"}]') `<br/>`as x(a int, b text);`</pre>|<pre>`a | b `<br/>`---+----`<br/>`1 | foo`<br/>`2 | NULL`</pre>|
|<pre>`json_strip_nulls(`<br/>`from_json json)`<br/><br/>`jsonb_strip_nulls(`<br/>`from_json jsonb)`</pre>| json <br/><br/> jsonb |返回json对象中所有非null的数据，其他的null保留。|<pre>`select json_strip_nulls(`<br/>`'[{"f1":1,"f2":null},`<br/>`2,null,3]');`</pre>|<pre>`[{"f1":1},`<br/>`2,null,3]`</pre>|
|<pre>`jsonb_set(`<br/>`target jsonb, `<br/>`path text[],`<br/>`new_value jsonb[,`<br/>`create_missing boolean])`</pre>| jsonb |如果create_missing为true，则将在target的path处追加新的jsonb；如果为false，则替换path处的value。|<pre>`select jsonb_set(`<br/>`'[{"f1":1,"f2":null},`<br/>`2,null,3]', `<br/>`'{0,f1}','[2,3,4]', false);`<br/><br/>`select jsonb_set('`<br/>`[{"f1":1,"f2":null},2]', `<br/>`'{0,f3}','[2,3,4]');`</pre>|<pre>`[{"f1": [2, 3, 4], `<br/>`"f2": null}, `<br/>`2, null, 3]`<br/><br/>`[{"f1": 1, `<br/>`"f2": null, `<br/>`"f3": [2, 3, 4]},`<br/>` 2]`</pre>|
|<pre>`jsonb_insert(`<br/>`target jsonb, path text[],`<br/>`new_value jsonb, `<br/>`[insert_after boolean])`</pre>| jsonb |如果insert_after是true，则在target的path后面插入新的value，否则在path之前插入|<pre>`select jsonb_insert(`<br/>`'{"a": [0,1,2]}', `<br/>`'{a, 1}', `<br/>`'"new_value"');`<br/><br/>`select jsonb_insert(`<br/>`'{"a": [0,1,2]}', `<br/>`'{a, 1}', '"new_value"', true);`</pre>|<pre>`{"a": [`<br/>`0,`<br/>` "new_value",`<br/>` 1, 2]}`<br/><br/>`{"a": [`<br/>`0, 1, `<br/>`"new_value",`<br/>` 2]}`</pre>|
|<pre>`jsonb_pretty(`<br/>`from_json jsonb)`</pre>| text |以缩进的格式更容易阅读的方式返回json对象|<pre>`select jsonb_pretty(`<br/>`'[{"f1":1,"f2":null},`<br/>`2,null,3]');`</pre>|<pre>`[{"f1": 1,`<br/>`"f2": null},`<br/>`2,null,3]`</pre>|


















