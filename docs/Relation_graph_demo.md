# Relation Graph Demo  


We will demonstrate on the local console how to use the sparql_update interface and sparql_query interface provided by Relation Graph to implement user information management and user friend data management.

## 1. Grant ACL 

+ Authorize the user
    ```sh
    dfx canister call ic_graph acl_grant '("g4lfy-u4kk4-vlixd-jufrl-7x2ro-myhu2-ptpz3-lx4vu-u5ruj-ohwvl-hqe","user")'

    ```
    The authorization is successful and SUCCESS is returned:
    ![](../images/2022-03-30-10-47-52.png)

+ Query the authorization list
    ```sh
    dfx canister call ic_graph acl_show '(10)'

    ```
    The query results are as follows, the following two users can perform the sparql_update operation:
    ![](../images/2022-03-30-10-50-35.png)



## 2. Insert user's information

+ Three users are saved, including attribute data for the users. Friendship of three users： P1001 --> P1002 <--> P1003
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
    If the execution is successful, a "SUCCESS" prompt will be returned

  + Query  results

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
    Query all user data in the current database:
    ![](../images/2022-03-28-19-24-28.png)

## 3. Query user's information

In this example, we mainly demonstrate sorting, query number, multi-condition query:

+ Query data, sort the query results, and limit the number of query items:
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
    Display the results in order of age:
    ![](../images/2022-03-28-19-25-47.png)

+ Query data, use multiple query conditions to filter:
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
    Query only the results of users whose name is Chris and age>=40:
    ![](../images/2022-03-28-19-26-34.png)

+ Fuzzy Enquiry：
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
    User results with the letter i in the name：
    ![](../images/2022-03-28-19-27-11.png)


## Query user's friend list

In this example, we mainly demonstrate  query one-degree relationship, and query second-degree relationship：

+ Query a user's friend list

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

+ Query a user's second-degree friend relationship (friend's friend list), and deduplicate the results

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
    The second friend of P1001 is P1003, so the query will return the data of P1003:
    ![](../images/2022-03-28-19-14-07.png)



## 4. Update user's information

+ Modify a user's age

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

+ Check the modified result

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

## 5. Deletion user

+ Delete some attributes of P1001 user
    ```sh
    dfx canister  call ic_graph sparql_update '("
        DELETE WHERE
        {
        :P1001 :age ?age;
            :name ?name .
        }
    ")'
    ```

+ Delete all attributes of P1003 user
    ```sh
    dfx canister  call ic_graph sparql_update '("  
        DELETE WHERE
        {
        :P1001 ?p ?o .
        }
    ")'
    ```



+ Check delete results
    ```sh
    dfx canister   call ic_graph sparql_query '("tsv","
        SELECT *
        WHERE { 
            :P1001 ?p ?o .
        }
    ")'
    ```
    After the deletion is successful, the corresponding data will not be queried:
    ![](../images/2022-03-28-19-12-51.png)


