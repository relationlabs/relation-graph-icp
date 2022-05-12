# 快速开始

接下来，我们将在本地部署一个Relatioin Graph的服务

## 部署本地服务

1. 安装dfinity的SDK

    在**Ubuntu系统(20.04版本)**或**Mac系统**上安装dfinity的SDK, 建议切换成**root用户**执行以下命令
    ```sh
    sh -ci "$(curl -fsSL https://sdk.dfinity.org/install.sh)"

    # 检查是否安装成功
    dfx  --version
    ```
2. 创建一个demo工程
    ~~~sh
    dfx new --type=rust relation_graph_demo
    ~~~
3. 配置relation wasm包
   
   + 下载relation_graph 的wasm包以及did文件 的压缩文件，解压后放到项目根目录下
        ~~~sh
            cd relation_graph_demo/
            curl -LJO https://storageapi2.fleek.co/shadow-001-team-bucket/relationlabs/relation_graph_with_acl.zip
            unzip relation_graph_with_acl.zip
        ~~~
   + 在项目根目录下找到dfx.json文件，其中canisters的内容替换成下面的内容
        ~~~json
            "canisters": {
                "relation_graph": {
                    "type": "custom",
                    "wasm": "relation_graph.wasm",
                    "candid": "relation_graph.did"
                }
            }
        ~~~
4. 启动relation_graph项目
   + 开启dfx环境 
        ~~~sh
        dfx start --clean --background
        ~~~

        如果看到以下信息，则说明正确启动成功
        ![](../../images/2022-03-27-11-30-50.png)
   + 启动项目
       ~~~sh
       dfx deploy --no-wallet relation_graph
       ~~~

       如果看到以下信息，则说明项目启动成功
       ![](../../images/2022-03-27-11-58-48.png)

5. 调用服务接口

   + 存储数据
    调用Relation Graph的 sparql_update方法，存储一个用户信息
        ~~~sh
            dfx canister   call relation_graph sparql_update '("  
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
        调用成功后，将返回SUCESS信息：
        ![](../../images/2022-03-28-18-44-53.png)

   + 查询数据
      调用Relation Graph的 sparql_query方法，查询所有用户信息
     ~~~sh
        dfx canister   call relation_graph sparql_query '("tsv","
            SELECT * 
            WHERE {
                ?s :name ?name;
                    :age ?age ;
                    :gender ?gender ;
                    :birthdate ?birthdate.
            } 
        ")'
     ~~~
     调用成功，将返回上面插入的数据：
     ![](../../images/2022-03-28-18-46-58.png)

    
    至此，我们已经快速在本地搭建部署了Relation Graph服务，并通过接口进行数据的存储和查询。


## 部署至IC主网

如果我们需要在IC的主网上部署一个Relation Graph，可以接着本地部署步骤后面做如下操作：

1. 在主网上创建canister
   
   ```sh
    # query your principal 
    dfx identity get-principal

    # Create a new canister with cycles by transferring ICP tokens from your ledger account by running a command similar to the following 
    # e.g. dfx ledger --network ic create-canister  djp6k-xiv5n-udf77-eaphw-iv6pr-u43dl-7wm3i-oa5xc-2mg6u-xuyko-fae --amount  0.1
    dfx ledger --network ic create-canister <controller-principal-identifier> --amount <icp-tokens>
    
   ```

2. 配置canister_ids.json
   
   在项目根目录下增加canister_ids.json文件，内容如下：
   ```json
    {
        "relation_graph": {
            "ic": "${your canisterId}"
        }
    }
   ```
   ![](../../images/2022-04-15-16-10-50.png)

3. 部署命令

    ```sh
    dfx deploy --network ic --no-wallet relation_graph
    ```

4. 调用主网的Relation Graph

    在dfx canister命令后加上 --network ic  ,申明访问主网canister接口

   + 存储数据
        ~~~sh
            dfx canister --network ic  call relation_graph sparql_update '("  
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


   + 查询数据
     ~~~sh
        dfx canister  --network ic  call relation_graph sparql_query '("tsv","
            SELECT * 
            WHERE {
                ?s :name ?name;
                    :age ?age ;
                    :gender ?gender ;
                    :birthdate ?birthdate.
            } 
        ")'
     ~~~


## 视频演示

<iframe width="760" height="515" src="https://www.youtube.com/embed/N2Zx3WfqX_g" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>