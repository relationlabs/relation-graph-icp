#  API文档

## 概述


|   模块   |    接口名称        |       操作者角色        |     接口说明  |
|   ---     |   ---             |       ---          |     ---     |
|   授权    |    acl_grant      |     admin          |   管理员给其他用户授权|
|   授权    |   acl_show        |      admin         |   管理员查看授权列表    |
|   授权    |   acl_revoke        |      admin         |   管理员删除授权对象    |
|   数据操作    |   sparql_update   |      admin / user    | 执行SPARQL语句，处理增、删、改操作 |
|   数据操作    |   sparql_query    |      all        | 执行SPARQL语句，处理查询操作。|


Dapp可以将下面的js添加到项目中，进行使用：
```js
export const idlFactory = ({ IDL }) => {
  return IDL.Service({
    'acl_grant' : IDL.Func([IDL.Text, IDL.Text], [IDL.Text], []),
    'acl_revoke' : IDL.Func([IDL.Text], [IDL.Text], []),
    'acl_show' : IDL.Func([IDL.Nat], [IDL.Text], ['query']),
    'sparql_query' : IDL.Func([IDL.Text, IDL.Text], [IDL.Text], ['query']),
    'sparql_update' : IDL.Func([IDL.Text], [IDL.Text], []),
  });
};
export const init = ({ IDL }) => { return []; };

```

## 授权

### acl_grant

Relation Graph的管理员可以给其他用户进行授权。

**备注**：我们预设了Relation Graph的部署者为初始管理员，具备所有接口的操作权限。

+ 方法did：
    ~~~rust
    service : {
        "acl_grant": (text, text) -> (text);
    }
    ~~~
+ 请求参数说明：
    - 第一个参数： 被授权用户的principal
    - 第二个参数： 被授权用户的角色，角色目前分为 管理员
+ 返回参数说明：
    - SUCCESS: 成功
    - ACCESS_DENIED ： 无权限调用
+ 请求示例：
  ~~~sh
    dfx canister call ic_graph acl_grant '("g4lfy-u4kk4-vlixd-jufrl-7x2ro-myhu2-ptpz3-lx4vu-u5ruj-ohwvl-hqe","user")'
  ~~~

### acl_show

Relation Graph的管理员可以查看当前的授权列表

+ 方法did：
    ~~~rust
    service : {
        "acl_show": (nat) -> (text) query;
    }
    ~~~
+ 请求参数说明：
    - 查询的数目
+ 返回参数说明：
    - 授权列表
+ 请求示例：
  ~~~sh
    dfx canister call ic_graph acl_show '(10)'
  ~~~

### acl_revoke

Relation Graph的管理员可以查看当前的授权列表

+ 方法did：
    ~~~rust
    service : {
        "acl_revoke": (text) -> (text) query;
    }
    ~~~
+ 请求参数说明：
    - 需要删除的对象
+ 返回参数说明：
    - 授权列表
+ 请求示例：
  ~~~sh
    dfx canister call ic_graph acl_revoke '("g4lfy-u4kk4-vlixd-jufrl-7x2ro-myhu2-ptpz3-lx4vu-u5ruj-ohwvl-hqe")'
  ~~~

## 数据操作

### sparql_update

具备Relation Graph admin/user 角色的用户可以进行数据操作。

+ 方法did：
    ~~~rust
    service : {
        "sparql_update": (text) -> (text);
    }
    ~~~
+ 请求参数说明：
    - 需要执行的SparQL语句
+ 返回参数说明：
    - SUCCESS: 执行成功
    - ACCESS_DENIED ： 无权限调用
+ 请求示例：
  ~~~sh
    dfx canister   call ic_graph sparql_update '("  
		INSERT DATA
		{ 
			:P1024 :name \"GraphDB\" ;
				:gender \"Male\" ;
				:age 30 ;
				:birthdate \"1992-03-19\"^^xsd:date ;
				:friends :P1 .
		}
	")'
  ~~~


### sparql_query

查询 Relation Graph的数据，无权限校验。

+ 方法did：
    ~~~rust
    service : {
        "sparql_query": (text, text) -> (text) query;
    }
    ~~~
+ 请求参数说明：
    - param0： 数据展示形式,有以下枚举值：
      -  tsv
      -  csv
      -  json
      -  xml
    - param1: 需要执行的SparQL语句 
+ 返回参数说明：
    - 查询结果
+ 请求示例：
  ~~~sh
	dfx canister   call ic_graph sparql_query '("tsv","
		SELECT * 
		WHERE {
			?s :name ?name;
				:age ?age ;
				:gender ?gender ;
				:birthdate ?birthdate.
		} 
	")'
  ~~~