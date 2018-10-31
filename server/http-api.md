HTTP API 接口
==========================

链接说明
-------------------

接口地址前缀是域名加上端口，
比如本地测试链接 ``http://localhost:9984``
或者根据你的域名配置的 ``https://example.com:9984``

根链接
-----------------

根链接由链接前缀和接口版本号组成。
比如。 ``http://localhost:9984/api/v1/``
或者 ``http://example.com:9984/api/v1/``

Transactions
------------

**Get请求:: /api/v1/transactions/{transaction_id}**

通过交易ID参数``transaction_id``，获取交易详情。

如果交易ID``transaction_id`` 存在于区块里，则返回区块详情数据，否则返回 ``404 Not Found``。

*参数 transaction_id*: transaction ID
*类型 transaction_id*: 字符串

**请求示例**:
http://localhost:9984/api/v1/transactions/1

**返回示例**:

*resheader Content-Type*: ``application/json``

*statuscode 200*: 交易详情。

*statuscode 404*: 交易不存在。

如果没有传入任何交易ID参数则返回 ``400 Bad Request``。

**Get请求:: /api/v1/transactions?asset_id={asset_id}&operation={CREATE|TRANSFER}**

通过资源ID ``asset_id`` 搜索所有交易列表。

如果 ``operation`` 值是 ``CREATE``, 则返回所有操作类型为CREATE的交易数据。

如果 ``operation`` 值是 ``TRANSFER``, 则返回所有操作类型为TRANSFER的交易数据。

如果 ``operation`` 值为空，则返回所有类型的交易数据。

*参数 operation (可选)*: 字符串, ``CREATE`` 或者 ``TRANSFER``。

*参数 asset_id*: 字符串, 资源ID。

**请求示例**:

http://localhost:9984/api/v1/asset_id=1&operation=CREATE

**返回示例**:

*resheader Content-Type*: ``application/json``。

*statuscode 200*: 包含所对应资源ID ``asset_id`` 值的所有交易。

*statuscode 400*: 所对应的资源ID ``asset_id`` 查询的交易不存在。


**Post请求:: /api/v1/transactions?mode={mode}**

通过此接口发送交易数据到网络里。

*参数 mode (可选)*: 字符串, 有三种值可选分别是: ``async``, ``sync``, ``commit``。 默认值是``async``。

当交易数据通过接口提交后，节点会先检查交易是否合法，如果不合法，会返回400报错。
否则，节点会发送交易到Tendermint(`Tendermint broadcast API<https://tendermint.com/docs/tendermint-core/using-tendermint.html#broadcast-api>`)跟其他节点进行数据同步。
   
``mode=async`` 表示交易发起后,数据到某个节点后,不管是否合法,立即返回结果。

``mode=sync`` 表示当交易发起后,数据在节点通过Tendermint验证成功后,并返回结果。

``mode=commit`` 表示交易已经完全同步到整个区块网络后才返回结果。

async和sync模式下,接口正常返回结果后,交易也可能被拒绝。在返回HTTP响应之前,所有事务都由Tendermint在WAL（Write-Ahead Log）内部记录。 不过应注意一下几点:

- WAL中的交易（包括失败的交易）不会在任何节点或Tendermint API中公开。
- 永远不会从WAL获取交易数据。WAL永远不会被重复调用。
- 即使HTTP响应指示成功，也可能发生严重故障（例如系统磁盘空间不足），从而阻止交易存储在WAL中。
- 如果交易验证失败，因为它与同一块的其他交易冲突，Tendermint将其包含在其块中，但网络节点不存储这些事务，也不在API中提供有关它们的任何信息。

每次提交的交易数据都应该被验证，此文档`<>`解释一笔交易如何进行验证，毕竟验证有效的过程。

**请求示例**:

http://localhost:9984/api/v1/transactions?mode=commit

**返回示例**:

:resheader Content-Type: ``application/json``。

:statuscode 202: 返回结果取决于``mode``参数的设置。

:statuscode 400: 无效的交易。

``mode``参数如果不传值，默认值即为``async``。

Transaction Outputs
-------------------

``/api/v1/outputs``接口根据public_key参数查询某个公钥下的所有交易结果数据。

**Get请求:: /api/v1/outputs**

 ``public_key``参数必须是一个基于base58加密的ed25519公钥，如果有对应的交易数据是此公钥的值则返回交易列表数据。

*参数public_key*: Base58加密的公钥地址，匹配交易数据的拥有者的公钥的值。如果此参数为空，则请求返回``400``。

*参数spent(可选)*:  布尔值 (``true`` or ``false``)，此参数为空时，返回所有spent和unspent的数据。

**请求示例**:

http://localhost/api/v1/outputs?public_key=TsDeu3QLojKNAqNEopcLb8GY6D3Z6J17rNrpVN3cddN

**返回示例**:

 HTTP/1.1 200 OK
 Content-Type: application/json

 ```
[
   {
     "output_index": 0,
     "transaction_id": "6233762c91b4e619fbe162848f7f157293c31b3b52fe3ad405a34ffb07cddbf1"
   },
   {
     "output_index": 1,
     "transaction_id": "2d431073e1477f3073a4693ac7ff9be5634751de1b8abaa1f4e19548ef0b4b0e"
   }
 ]
 ```
 
*statuscode 200*: 返回交易的列表数据，格式如上。

*statuscode 400*: ``public_key``对应的值搜不到相应的交易数据。

Assets
------

**Get请求:: /api/v1/assets**

通过输入的字符串模糊搜索对应的资源数据。

*参数 search*: 字符串,输入字符串进行模糊搜索。

*参数 limit(可选)*: 整型，返回的资源数据的数量，默认值为0，不传值返回匹配到的所有的资源数据。

注意事项:

资源数据的``id``和它关联的操作类型为CREATE的交易数据的``id``是一样的。此交易即为创建此资源的交易。

如果没有搜索到对应关键词的结果，则返回空列表数据。

如果search变量为空，则返回``400 Bad Request``。

**请求示例**:

http://localhost/api/v1/assets/?search=bdn

**返回示例**:

HTTP/1.1 200 OK
Content-type: application/json

```
[
    {
        "data": {"msg": "Hello ChainDB 1!"},
        "id": "51ce82a14ca274d43e4992bbce41f6fdeb755f846e48e710a3bbb3b0cf8e4204"
    },
    {
        "data": {"msg": "Hello ChainDB 2!"},
        "id": "b4e9005fa494d20e503d916fa87b74fe61c079afccd6e084260674159795ee31"
    },
    {
        "data": {"msg": "Hello ChainDB 3!"},
        "id": "fa6bcb6a8fdea3dc2a860fcdc0e0c63c9cf5b25da8b02a4db4fb6a2d36d27791"
    }
]
```

*resheader Content-Type*: ``application/json``。

*statuscode 200*: 搜索结果成功返回。

*statuscode 400*: 搜索失败，search字段为空。

Transaction Metadata
--------------------

**Get请求:: /api/v1/metadata**

通过输入的字符串模糊搜索对应的metadata数据。

*参数 search*: 字符串，模糊搜索的字符串的值。

*参数 limit(可选)*:  整型，返回的资源数据的数量，默认值为0，不传值返回匹配到的所有的metadata数据。

注意事项:
metadata数据的``id``和它关联的操作类型为CREATE的交易数据的``id``是一样的。此交易即为创建此metadata的交易。

如果没有搜索到对应关键词的结果，则返回空列表数据。

如果search变量为空，则返回``400 Bad Request``。

**请求示例**:

http://localhost/api/v1/metadata/?search=bdn 

**返回示例**:

HTTP/1.1 200 OK
Content-type: application/json

```
[
    {
        "metadata": {"metakey1": "Hello ChainDB 1!"},
        "id": "51ce82a14ca274d43e4992bbce41f6fdeb755f846e48e710a3bbb3b0cf8e4204"
    },
    {
        "metadata": {"metakey2": "Hello ChainDB 2!"},
        "id": "b4e9005fa494d20e503d916fa87b74fe61c079afccd6e084260674159795ee31"
    },
    {
        "metadata": {"metakey3": "Hello ChainDB 3!"},
        "id": "fa6bcb6a8fdea3dc2a860fcdc0e0c63c9cf5b25da8b02a4db4fb6a2d36d27791"
    }
]
```

*resheader Content-Type*: ``application/json``。

*statuscode 200*: 搜索结果成功返回。

*statuscode 400*: 搜索失败，search字段为空。

Validators
--------------------

**Get请求:: /api/v1/validators**

返回当前接口的服务节点的validators设置。

**请求示例**:

http://localhost/api/v1/validators

**Example response**:

HTTP/1.1 200 OK
Content-type: application/json

```
[
    {
        "pub_key": {
               "data":"4E2685D9016126864733225BE00F005515200727FBAB1312FC78C8B76831255A",
               "type":"ed25519"
        },
        "power": 10
    },
    {
         "pub_key": {
               "data":"608D839D7100466D6BA6BE79C320F8B81DE93CFAA58CF9768CF921C6371F2553",
               "type":"ed25519"
         },
         "power": 5
    }
]
```

*resheader Content-Type*: ``application/json``。

*statuscode 200*: 查询成功，返回validators设置的数据。

Blocks
------

**Get请求:: /api/v1/block_list**

返回所有的区块数据，可根据``page``参数和``pagesize``参数进行分页设置。

**参数 page(可选)**: 整型, 当前页码，默认值为1。

**参数 pagesize(可选)** : 整型, 当前页码返回数据条数，默认值为10。

**请求示例**:

http://localhost/api/v1/block_list

**返回示例**:

*resheader Content-Type*: ``application/json``。

*statuscode 200*: 返回区块数据列表。

*statuscode 400*: 服务异常不可用。

HTTP/1.1 200 OK
Content-type: application/json

```
[
  {
    "id": "5bd9670657f9fd000654dd11",
    "height": 3,
    "app_hash": "4fb0ca93c120c72a05b87cadc21dad00c6c5df3012ca4463eeca3a6f535bdb5e",
    "transactions": [],
    "transaction_count": 0
  },
  {
    "id": "5bd9670557f9fd000654dd10",
    "height": 2,
    "app_hash": "4fb0ca93c120c72a05b87cadc21dad00c6c5df3012ca4463eeca3a6f535bdb5e",
    "transactions": [
    "6233762c91b4e619fbe162848f7f157293c31b3b52fe3ad405a34ffb07cddbf1"
    ],
    "transaction_count": 1
  },
  {
    "id": "5bd9640f57f9fd000654dd0c",
    "height": 1,
    "app_hash": "",
    "transactions": [],
    "transaction_count": 0
  }
]
```

**Get请求:: /api/v1/blocks/{block_height}**

通过``block_height``参数获取对应区块数据。

**参数 block_height: 整型，区块高度。**

**请求示例**:

http://localhost/api/v1/blocks/1

**返回示例**:

*resheader Content-Type*: ``application/json``。

*statuscode 200*: 调用成功，返回对应的区块数据。

*statuscode 400*: 请求错误，一般是``block_height``参数没有传值。

*statuscode 404*: 要获取的高度的区块不存在。


**Get请求:: /api/v1/blocks?transaction_id={transaction_id}**

通过``transaction_id``获取对应的区块的数据，返回交易数据列表。

注意事项:

如果没有查询到对应交易ID的区块数据，也可以返回空数据，表示查询成功。

*参数 transaction_id*: 字符串，交易ID，必传参数，可以为空。

**请求示例**:

http://localhost/api/v1/blocks?transaction_id=6233762c91b4e619fbe162848f7f157293c31b3b52fe3ad405a34ffb07cddbf1

**返回示例**:

*resheader Content-Type*: ``application/json``。

*statuscode 200*: 调用成功，返回0个或多个对应交易ID的区块的数据。

*statuscode 400*: 调用失败，一般是没有传transaction_id参数。

Common
--------------------

**Get请求:: /api/v1/global_search?tx_or_block_id={tx_or_block_id}**

通过交易ID或区块高度查询交易数据或者区块数据。如果查到区块的数据则返回数据类型为区块，如查询到交易数据则返回数据类型为交易。如都没有匹配到数据，返回false。

*参数 tx_or_block_id*: 字符串，交易ID或者区块高度。

**请求示例**:

http://localhost/api/v1/global_search?tx_or_block_id=6233762c91b4e619fbe162848f7f157293c31b3b52fe3ad405a34ffb07cddbf1

**返回示例**:

*resheader Content-Type*: ``application/json``。

*statuscode 200*: 调用成功，对应的数据。

HTTP/1.1 200 OK
Content-type: application/json

```
{
    "result": true,
    "result_type": "block"
}
```

**Get请求:: /api/v1/transaction_count**

查询全网所有交易的数量。

**请求示例**:

http://localhost/api/v1/transaction_count

**返回示例**:

*resheader Content-Type*: ``application/json``。

*statuscode 200*: 调用成功，返回对应的数据。

HTTP/1.1 200 OK
Content-type: application/json

```
{
    "count": 5
}
```

**Get请求:: /api/v1/last_block**

获取全网的最后一条区块的数据信息。数据包含区块的哈希值，区块高度，和此区块下的所有交易的数据列表，交易数据可能为空。

**请求示例**:
http://localhost/api/v1/last_block

**返回示例**:

*resheader Content-Type*: ``application/json``。

*statuscode 200*: 调用成功，返回对应的数据。

HTTP/1.1 200 OK
Content-type: application/json

```
{
    "app_hash": "4fb0ca93c120c72a05b87cadc21dad00c6c5df3012ca4463eeca3a6f535bdb5e",
    "height": 3,
    "transactions": []
}
```