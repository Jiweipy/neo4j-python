# Neo4j

## 配置

- 初始账户：neo4j
- 初始密码：neo4j
- 修改密码：199***

## neo4j操作

### 统计

统计根节点数

> match (n) where not ()-->(n) return count(distinct n)

统计叶节点数

> MATCH (n) RETURN count(*)

### 添加

创建节点,并添加节点属性

> $ create (n:华山{name:"令狐冲", skill:"易筋经", master:"岳不群"})
>
> $ create (n:华山{name:"岳不群", skill:"易筋经", master:"岳是群"})

添加关系

> $ match (a:华山), (b:华山) where a.name = "岳不群" and b.name = "令狐冲"  create (a)-[r:师徒]->(b);

创建节点时添加关系

> $ create (n:地球{name:"草地"}), (m:天空{name:"太阳"})  create (n) - [r:拥抱] -> (m)

### 查询

查询节点(添加限制)

> $ match (e:华山) where e.name = "令狐冲" return e

查询整个节点信息

> $ match (e:华山) return e

按照关系查询

> $ match R = (p1:华山) - [r:师徒] ->(p2) return R

按照关系查询(根据限制)

> $ match R = (p1:华山) - [r:师徒] ->(p2) where p1.name="岳不群" and p2.name="令狐冲" return R;

### 更新

更新(添加)节点属性

> $ match (e:华山) set e.skill = "吸星大法"

更改关系

> $  MATCH (n:华山 {name:"令狐冲"})-[r:师父]->(m:华山 {name:"岳不群"}) CREATE (n)-[r2:朋友]->(m) 
> ​	SET r2 =r
> ​	WITH r
> ​	DELETE r

### 删除

删除关系（先删除关系，再删除节点）

> $ match (n:华山) - [r:朋友] -> (m:华山) delete r

删除单个节点

> $ match (n:自然) where n.name = "太阳" delete n

删除整个标签

> $ match (n:自然) delete n

删除属性

> $ match (n:华山{name:"岳不群"}) remove n.skill;

<u>**删除所有数据与关系**</u>

> $ match (n) detach delete n

## neo4j-python操作

### 添加

添加节点和属性

```python
def create_node(tx, name, skill):
    tx.run("CREATE (n:华山 {name: $name, skill: $skill}) ", name=name, skill=skill)
with driver.session() as session:
    session.write_transaction(create_node, "令狐冲", "易筋经")
```

添加关系

```python
# 创建关系
def create_relation(tx, name1, name2):
    tx.run(" match (a:华山), (b:华山) where a.name = $name1 and b.name = $name2  create 			(a)-[r:师父]->(b); ", 
           	name1=name1, 
           	name2=name2)
with driver.session() as session:
    session.write_transaction(create_relation, "令狐冲", "岳不群")
```

添加节点时添加关系

```python
def create_node_relation(tx, name1, name2):
    tx.run("create (n:自然{name:$name1}), (m:自然{name:$name2})  create (n) - [r:拥抱] -> 			(m)", 
           name1=name1, 
           name2=name2)
with driver.session() as session:
    session.write_transaction(create_node_relation, "草地", "太阳")
```

### 查询

查询节点(添加限制)

```python
def query_node_by_limit(tx, name):
    result = tx.run("match (e:华山) where e.name = $name  return e", name=name)
#     print(result.single()[0].values())
    for i in result:
        print(i["e"])
with driver.session() as session:
    session.write_transaction(query_node_by_limit, "令狐冲")
```

查询整个节点信息

```python
def query_node(tx):
    result = tx.run("match (e:华山) return e")
    for i in result:
        print(i["e"])
with driver.session() as session:
    session.write_transaction(query_node)
```

按照关系查询

```python
def query_node_by_relation(tx, name1, name2):
    result = tx.run("match R = (p1:华山) - [r:师父] ->(p2:华山) where p1.name=$name1 and 						p2.name=$name2 return R;", 
                    name1=name1, name2=name2)
    for i in result:
        print(i["R"])
with driver.session() as session:
    session.write_transaction(query_node_by_relation, "令狐冲", "岳不群")
```

### 更新

更新属性

```python
def update_attr(tx, name, skill):
    tx.run("match (e:华山 {name: $name}) set e.skill = $skill", name=name, skill=skill)
with driver.session() as session:
    session.write_transaction(update_attr, "令狐冲", "吸星大法")
```

更新关系

```python
def update_relation(tx, name1, name2):
    tx.run("MATCH (n:华山 {name:$name1})-[r:朋友]->(m:华山 {name:$name2}) CREATE (n)-[r2:师父]->(m) SET r2 = r WITH r DELETE r", name1=name1, name2=name2)
with driver.session() as session:
    session.write_transaction(update_relation, "令狐冲", "岳不群")
```

更新节点

```python
def update_node(tx, name, new_name):
    tx.run("match (e:华山 {name: $name}) set e.name = $new_name", name=name, new_name=new_name)
with driver.session() as session:
    session.write_transaction(update_node, "令狐冲", "LHC")
```

### 删除

删除属性

```python
def del_attr(tx, name):
    message = tx.run("match (n:华山 {name:$name}) remove n.skill", name=name)
    print(message)
with driver.session() as session:
    session.write_transaction(del_attr, "LHC")
```

删除关系

```python
def del_relation(tx, name1, name2):
    message = tx.run(" match (n:华山 {name:$name1}) - [r:师父] -> (m:华山 {name:$name2}) delete r", name1=name1, name2=name2)
with driver.session() as session:
    session.write_transaction(del_relation, "LHC", "岳不群")
```

删除节点

```python
def del_node(tx, name):
    message = tx.run("match (n:华山 {name:$name}) delete n", name=name)
with driver.session() as session:
    session.write_transaction(del_node, "LHC")
```

删除标签

```python
def del_label(tx):
    message = tx.run("match (n:华山) delete n")
with driver.session() as session:
    session.write_transaction(del_label)
```


