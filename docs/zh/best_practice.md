#  最佳实践

利用Relation Graph，我们可以快速实现一些基础应用，比如通讯录、云笔记、

## 图数据使用Demo


我们将在本地控制台演示，如何使用 Relation Graph提供的sparql_update接口和sparql_query接口，实现用户信息管理和用户好友数据管理。

### ACL 授权

+ 给用户授权
    ```sh
    dfx canister call ic_graph acl_grant '("g4lfy-u4kk4-vlixd-jufrl-7x2ro-myhu2-ptpz3-lx4vu-u5ruj-ohwvl-hqe","user")'

    ```
    授权成功，返回 SUCCESS:
    ![](../../images/2022-03-30-10-47-52.png)

+ 查询授权列表
    ```sh
    dfx canister call ic_graph acl_show '(10)'

    ```
    查询结果如下，下面这两个用户均可以进行sparql_update的操作:
    ![](../../images/2022-03-30-10-50-35.png)



### 新增用户

+ 保存三个用户，包括用户的属性数据。 三个用户的好友关系： P1001 --> P1002 <--> P1003
    ```sh
    dfx canister  call ic_graph sparql_update '("  
        INSERT DATA
        { 
            :P1001 :name \"Alice\" ;
                :gender \"Female\" ;
                :age 35 ;
                :birthdate \"1986-10-14\"^^xsd:date ;
                :friends :P1002 .
        }
    ")'

    dfx canister  call ic_graph sparql_update '("  
        INSERT DATA
        { 
            :P1002 :name \"Bob\" ;
                :gender \"Male\" ;
                :age 38 ;
                :birthdate \"1983-10-14\"^^xsd:date ;
                :friends :P1003 .
        }
    ")'

    dfx canister  call ic_graph sparql_update '("  
        INSERT DATA
        { 
            :P1003 :name \"Chris\" ;
                :gender \"Male\" ;
                :age 40 ;
                :birthdate \"1981-10-14\"^^xsd:date ;
                :friends :P1002 .
        }
    ")'

    ```
    执行成功，将返回 "SUCCESS"提示信息

+ 查询保存的结果

    ~~~sh
    dfx canister call ic_graph sparql_query '("tsv","
        SELECT ?name  ?age ?gender ?friends ?birthdate
        WHERE {
            ?s :name ?name;
                :age ?age ;
                :gender ?gender ;
                :friends ?friends ;
                :birthdate ?birthdate .
        } 
    ")'
    ~~~
    查询当前数据库所有用户数据：
    ![](../../images/2022-03-28-19-24-28.png)

### 查询用户列表

在查询用户列表示例中，我们主要演示排序、查询条数、多条件查询：

+ 查询数据，对查询结果进行排序，且限制查询条目数:
    ```sh
    dfx canister  call ic_graph sparql_query '("tsv","
        SELECT *
        WHERE {
            ?person :name ?name;
                    :age ?age ;
                    :gender ?gender ;
                    :friends ?friends ;
                    :birthdate ?birthdate .
        } 
        ORDER BY ASC(?age)
        limit 10
    ")'
    ```
    按照age顺序展示结果：
    ![](../../images/2022-03-28-19-25-47.png)

+ 查询数据，使用多个查询条件进行过滤:
    ```sh
    dfx canister  call ic_graph sparql_query '("tsv","
        SELECT *
        WHERE {
            ?person :name ?name;
                    :age ?age ;
                    :gender ?gender ;
                    :friends ?friends ;
                    :birthdate ?birthdate .
                    FILTER (?name = \"Chris\") .
                    FILTER (?age >= 40) .
        } 
        ORDER BY DESC(?age)
        limit 10
    ")'
    ```
    仅查询name为Chris，且age>=40的用户结果：
    ![](../../images/2022-03-28-19-26-34.png)

+ 模糊查询:
    ```sh
    dfx canister   call ic_graph sparql_query '("tsv","
        SELECT *
        WHERE {
            ?person  :name ?name;
                    :age ?age ;
                    :gender ?gender ;
                    :friends ?friends ;
                    :birthdate ?birthdate .
                    FILTER regex(?name, \"i\") .
        } 
        ORDER BY DESC(?age)
        limit 10
    ")'
    ```
    name中包含字母i的用户结果：
    ![](../../images/2022-03-28-19-27-11.png)


### 查询好友列表

在查询好友列表的示例中，我们主要演示查询一度关系、查询二度关系：

+ 查询一个用户的好友列表

  ~~~sh
    dfx canister   call ic_graph sparql_query '("tsv","
        SELECT DISTINCT ?name ?age ?gender ?birthdate
        WHERE {
            :P1001 :friends ?friends1.
            ?friends1  :name ?name;
                    :age ?age ;
                    :gender ?gender ;
                    :birthdate ?birthdate .
        }
        LIMIT 10
    ")'
  ~~~

+ 查询一个用户的二度好友关系（好友的好友列表），且对结果进行去重

    ~~~sh
    dfx canister  call ic_graph sparql_query '("tsv","
        SELECT DISTINCT ?name ?age ?gender ?birthdate
        WHERE {
            :P1001 :friends ?friends1.
            ?friends1 :friends  ?friends2.
            ?friends2  :name ?name;
                    :age ?age ;
                    :gender ?gender ;
                    :birthdate ?birthdate .
        }
        LIMIT 10
    ")'
    ~~~
    P1001的二度好友是P1003,故查询将返回P1003的数据：
    ![](../../images/2022-03-28-19-14-07.png)



### 修改用户信息

+ 修改一个用户年龄

    ```sh
    dfx canister  call ic_graph sparql_update '("  
        DELETE
        { :P1001 :age ?o }
        INSERT
        { :P1001 :age 38 }
        WHERE
        { :P1001 :age ?o }
    ")'
    ```

+ 检查修改的结果

    ~~~sh
    dfx canister  call ic_graph sparql_query '("tsv","
        SELECT ?age
        WHERE {
            :P1001 :name ?name;
                :age ?age ;
                :gender ?gender ;
                :friends ?friends ;
                :birthdate ?birthdate .
        } 
    ")'
    ~~~

### 删除用户

+ 删除P1001用户的部分属性
    ```sh
    dfx canister  call ic_graph sparql_update '("
        DELETE WHERE
        {
        :P1001 :age ?age;
            :name ?name .
        }
    ")'
    ```

+ 删除P1003用户的所有属性
    ```sh
    dfx canister  call ic_graph sparql_update '("  
        DELETE WHERE
        {
        :P1001 ?p ?o .
        }
    ")'
    ```



+ 检查删除结果
    ```sh
    dfx canister   call ic_graph sparql_query '("tsv","
        SELECT *
        WHERE { 
            :P1001 ?p ?o .
        }
    ")'
    ```
    删除成功后，将查询不到对应数据：
    ![](../../images/2022-03-28-19-12-51.png)

