## 合法捕鱼

这个 Github 仓库是原仓库的汉化，展示了如何使用零知识证明建立一个 Fluree 账本。

在本例中，Fluree 被部署为一个后端数据管理平台，用于管理可信数据。这个用例是一个简单的例子:

1. 建立一个允许捕鱼的动态区域
2. 船舶输入捕获位置而不透露捕获的确切位置的能力
3. 第三方能够验证在允许区域内捕获的鱼，但不披露捕获地点

请查看附带的 [博客文章](https://flur.ee/2020/02/05/using-zero-knowledge-proofs-with-fluree/) 和 [视频](https://youtu.be/LlBBaorIzgs) !

展示的特性: 零知识证明、数据不变性和可跟踪性。

![Legal Fishing App](public/zero-knowledge.gif)

### 开始

1. `启动Fluree`

下载并解压这个 [Fluree 包](https://fluree-examples.s3.amazonaws.com/fluree-zero-knowledge-packet.zip). 这个数据包包含 Fluree，版本0.13.0，以及一个预先填充的数据库，其中包含模式、种子数据和一些示例证明(`resources/schema.json` 和 `resources/seed.json` 分别有模式和种子事务, respectively).

导航到下载数据包的文件夹，然后运行 `./fluree_start.sh`. 。如果你已经安装了 Java 8 + ，这应该会启动 Fluree，并且管理控制台将提供给你在`http://localhost:8080`中浏览。 `resources/example_queries.js` 有一些示例查询，您可以直接在管理控制台中进行测试。

2. `启动应用`

```
git clone https://github.com/fluree/legal-fishing.git
```

```
cd legal-fishing
```

```
npm install
```

```
npm start
```

### 工作原理

零知识证明允许你证明你的秘密信息符合某些参数，而不需要共享你的秘密信息。我们使用 Iden3的 [circom](https://github.com/iden3/circomlib) 来创建这个应用程序中使用的电路，使用 Iden3的 [snarkjs] 来处理证明的生成和验证。


请查看附带的[博客文章](https://flur.ee/2020/02/05/using-zero-knowledge-proofs-with-fluree/)以获得更多关于本例中使用的零知识证明的信息。

#### 查询获取电路

这个应用程序使用一个单一的电路，我们用来生成所有的证明。我们将电路、证明密钥和验证密钥存储在 `id` `legalFishing`下的 `snarkConfig` 集合中。我们使用下面的查询来获取电路和证明的关键。

```
{   "selectOne": ["?circuit", "?provingKey"],
    "where": [
            ["?snark", "snarkConfig/id", "legalFishing"],
            ["?snark", "snarkConfig/circuit", "?circuit"],
            ["?snark", "snarkConfig/provingKey", "?provingKey"]]
}
```

#### 交易提交证明

下面的事务用于创建一个新的证明，并将该证明链接到 `snarkConfig`，该配置存储相关的电路和密钥。

```
[{
    "_id": ["snarkConfig/id", "legalFishing"],
    "proofs": ["proof$1"]
  }, 
  {
    "_id": "proof$1",
    "proof": "PROOF HERE",
    "instant": "#(now)",
    "publicSignals": "PUBLIC SIGNALS HERE"
  }]
```

#### 查询获取与合法捕鱼电路相关的所有证据

下面的查询获取所有连接到 legalFishing 电路的证明。

```
{
    "select": ["?proof", "?proofBody", "?publicSignals", "?verificationKey", "?instant"],
    "where": [
                    ["?proof", "proof/proof", "?proofBody"],
                    ["?proof", "proof/publicSignals", "?publicSignals"],
                    ["?proof", "proof/instant", "?instant"],
                    ["?config", "snarkConfig/id", "legalFishing"],
                    ["?config", "snarkConfig/verificationKey", "?verificationKey"]
    ]
}
```

在`/resources/example_queries.js`中还可以尝试其他示例查询。

![Legal Fishing Queries](public/queries.gif)

#### Visual Studio Code
如果使用Visual Studio Code查看此仓库，则可以使用拓展发出示例查询。

1. 下载 `Fluree: Beta extension`. 在顶部菜单栏选择 `View` > `Extensions`. 然后搜索 `Fluree: Beta` 并点击安装.
2. Open the Command Palette by going to `View` > `Command Palette`, and issue `Fluree: Set Config`.
3. Highlight any query in `resources/example_queries.js`, using the Command Palette, issue, `Fluree: Query`, and the results of the query will appear in a `flureeResponse.txt` file. Note: every time you issue a query or transaction, this file gets overwritten.

![Visual Studio Code](public/vscode.gif)

### 资源

To see more example projects, visit our [example repo](https://github.com/fluree/examples). 

This example also has an accompanying [blog post](https://flur.ee/2020/02/05/using-zero-knowledge-proofs-with-fluree/) and [video](https://youtu.be/LlBBaorIzgs).


Check out our entire [documentation](https://docs.flur.ee/) or jump directly to the section on [full-text search](https://docs.flur.ee/docs/database-setup/database-settings#language).

You can also engage with us via email, `support@flur.ee`.

Or by [Slack](https://launchpass.com/flureedb).


