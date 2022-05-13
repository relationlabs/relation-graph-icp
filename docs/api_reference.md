#  API Documentation

## Introduction


|   Module   |    Interface name        |       Operator role        |     Description  |
|   ---     |   ---             |       ---          |     ---     |
|   Authorization    |    acl_grant      |     admin          |   admin authorizes access to other users|
|   Authorization    |   acl_show        |      admin         |   admin views the authorization list    |
|   Authorization    |   acl_revoke        |      admin         |   admin deletes a authorized user    |
|   Data operation    |   sparql_update   |      admin / user    | execute SPARQL command to handle create, delete, and update |
|   Data operation    |   sparql_query    |      all        | execute SPARQL command to handle query|


Dapp can include the following js code in its project to use:
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

## Authorization

### acl_grant

Relation Graph's admin can authorize access to other users

**Note**：We set the deployer of the Relation Graph as the initial admin, who has access to all interfaces.

+ Method did：
    ~~~rust
    service : {
        "acl_grant": (text, text) -> (text);
    }
    ~~~
+ Explanation on the request parameters: 
    - The first parameter:  the authorized user's principal
    - The second parameter: the authorized user's role. Currently, it is admin. 
+ Explanation on the return parameters: 
    - SUCCESS: successful
    - ACCESS_DENIED ： denied access
+ Example of usage: 
  ~~~sh
    dfx canister call ic_graph acl_grant '("g4lfy-u4kk4-vlixd-jufrl-7x2ro-myhu2-ptpz3-lx4vu-u5ruj-ohwvl-hqe","user")'
  ~~~

### acl_show

Relation Graph's admin can view current authorization list

+ Method did：
    ~~~rust
    service : {
        "acl_show": (nat) -> (text) query;
    }
    ~~~
+ Explanation on the request parameters: 
    - how many records to query ?
+ Explanation on the return parameters: 
    - the authorization list
+ Example of usage: 
  ~~~sh
    dfx canister call ic_graph acl_show '(10)'
  ~~~

### acl_revoke

Relation Graph's admin can revoke someone from the authorization list. 

+ Method did：
    ~~~rust
    service : {
        "acl_revoke": (text) -> (text) query;
    }
    ~~~
+ Explanation on the request parameters: 
    - the user needed to be revoked
+ Explanation on the return parameters: 
    - the authorization list
+ Example of usage: 
  ~~~sh
    dfx canister call ic_graph acl_revoke '("g4lfy-u4kk4-vlixd-jufrl-7x2ro-myhu2-ptpz3-lx4vu-u5ruj-ohwvl-hqe")'
  ~~~

## Data operations

### sparql_update

Users with the roles of Relation Graph admin/user can perform data operations.

+ Method did：
    ~~~rust
    service : {
        "sparql_update": (text) -> (text);
    }
    ~~~
+ Explanation on the request parameters: 
    - SparQL command that needs to be executed
+ Explanation on the return parameters: 
    - SUCCESS: executed successfully
    - ACCESS_DENIED ： not authorized
+ Example of usage: 
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

Query data in the Relation Graph. This method does not involve privilege authentication.

+ Method did：
    ~~~rust
    service : {
        "sparql_query": (text, text) -> (text) query;
    }
    ~~~
+ Explanation on the request parameters: 
    - param0： how to display the data. It has the following enumeration values：
      -  tsv
      -  csv
      -  json
      -  xml
    - param1: SparQL command that need to be executed;
+ Explanation on the return parameters: 
    - Query result
+ Example of usage: 
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
