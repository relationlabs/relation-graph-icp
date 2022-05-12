# SPARQL 语法说明

我们列出常用的语法示例：

## Insert

### 单条数据

1. 基础语法  
   ~~~sql
    INSERT DATA 
        { 
            :P2 :name "GraphDB" ;
                :friends :P1 . 
        } 
   ~~~
    + 声明为插入数据语句
      + INSERT DATA  
    + {} 内列出要保存的实例和属性
    + 三元组格式
      + :P2 :name "GraphDB" ; 
        + :P2   实例的唯一标识
        + :name     实例的属性名
        + "GraphDB"     实例的属性值
    + 分号结尾
      + 说明后面还有其他属性描述
    + 点号结尾
      + 表示该实例描述结束

### 批量数据

1. 基础语法
   ```sql
    INSERT DATA 
         { 
            :P1 :name "GraphDB" ;
                    :friends :P2 . 
            :P2 :name "Relation Graph" ;
                    :friends :P1 .
        } 
   ```


## Query

### 查询全量数据

1. 基础语法  
   ~~~sql
	SELECT  ?s ?p ?o 
		WHERE {
			?s ?p ?o.
		} 
   ~~~
    + 声明为查询语句
      + SELECT  
    +  {} 内列出要查询的实例和属性
    +  三元组格式
       +   ?s  ?p ?o  表示查询所有实例以及所有属性数据

### 查询指定属性

1. 基础语法  
   ~~~sql
	SELECT   * 
		WHERE {
			?s :name ?name;
				:age ?age ;
				:gender ?gender ;
				:birthdate ?birthdate.
		} 
   ~~~
    + 声明为查询语句
      + SELECT  
    +  要展示的属性，有两种写法
       +  “*” ： 指代展示{}内查到的所有属性
       +  ?name ?age 展示指定属性
    +  {} 内列出要查询的实例和属性
    +  三元组格式
       +   ?s  未指明具体实例
       +   ：name  属性名
       +   ?name  实例的属性值

### 去重

1. 基础语法
   
   ```sql
	SELECT  DISTINCT * 
		WHERE {
			?s :name ?name;
				:age ?age ;
				:gender ?gender ;
				:birthdate ?birthdate.
		} 

   ```
    + DISTINCT 对结果进行去重 

### 排序
   
1. 基础语法
   ```sql
   SELECT  DISTINCT * 
		WHERE {
			?s :name ?name;
				:age ?age ;
				:gender ?gender ;
				:birthdate ?birthdate.
		} 
        ORDER BY  DESC(?age)
        OFFSET 5
        limit 10
   ```
    +  ORDER BY  DESC(?age)  按照age进行排序，注意括号内是问号形式 

### 分页
    
1. 基础语法
   ```sql
   SELECT  DISTINCT * 
		WHERE {
			?s :name ?name;
				:age ?age ;
				:gender ?gender ;
				:birthdate ?birthdate.
		} 
        ORDER BY  DESC(?age)
        OFFSET 5
        LIMIT 10
   ```
    
    +  OFFSET   分页的offset
    +  LIMIT   分页 limit 

### 约束匹配
   
1. 基础语法  
   ```sql
	SELECT  DISTINCT * 
		WHERE {
			?s :name ?name;
				:age ?age ;
				:gender ?gender ;
				:birthdate ?birthdate.
                FILTER regex(?name, \"i\") .
                FILTER (?age >= 40) .
		} 
        ORDER BY  DESC(?age)
        OFFSET 5
        limit 10
   ```
   + **FILTER** 进行条件匹配,可以自定义正则实现模糊查询

### 关联查询
   
1. 基础语法 

   ```sql
    SELECT DISTINCT ?name ?age ?gender ?birthdate
        WHERE {
            :P1001 :friends ?friends1.
                            ?friends1  :name ?name;
                                        :age ?age ;
                                        :gender ?gender ;
                                        :birthdate ?birthdate .
            }
            LIMIT 10
   ```
   + 通过查询属性的属性数据,实现关联查询

## Delete
  
### 删除单个属性

1. 基础语法

   ~~~sql
    DELETE WHERE
        {
        :P1001 :age ?age.
        }

   ~~~
   + 声明为删除语句
     + DELETE  
   + {} 内列出要删除的实例和属性
   + 三元组格式
     + :P1001 :age ?age. 删除实例的age属性

### 删除实例

1. 基础语法
   ~~~sql  
    DELETE WHERE
        {
        :P1003 ?p ?o .
        }
   ~~~
   + 声明为删除语句
     + DELETE  
   + {} 内列出要删除的实例和属性
   + 三元组格式
     + :P1003 ?p ?o . 删除实例所有属性
       + :P1003 删除唯一标识为P1003的实体
       + ?p  属性名，使用问号，表示未指明具体属性，则删除实例所有属性
       + ?o  属性值

## Update

1. 基础语法
   ~~~sql
        DELETE
        { :P1001 :age ?o }
        INSERT
        { :P1001 :age 38 }
        WHERE
        { :P1001 :age ?o }
   ~~~
   + 在SPARQL中，Update是通过Delete和Insert语句组合完成。
