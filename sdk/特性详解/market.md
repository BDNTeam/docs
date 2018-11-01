# Market

下面介绍 SDK 中实现的基于 Ethereum 的 Market 流程。

## 示例

用户 Alice 将 asset_1 上传至 db chain，随后将 asset_1 出售给 Bob。也就是说，如果 Alice 需要将自己的资产出售给 Bob，她必须首先将资产上传到 db chain，然后发起出售的动作，类似挂单，然后由 Bob 进行买单。挂单和买单的过程，就是在与合约进行交互。

### 数据上传

首先，Alice 需要使用 SDK 将 asset_1 上传到 db chain，并获得一个该 asset 在 db chain 的标识，即 asset id。这里 Alice 要确保自己有足够多的 BDN 来完成该上传动作，因为资源在 db chain 的存储，是需要消耗 BDN 的。

数据会在 Alice 的客户端进行加密，加密时 SDK 会向 Alice 要求提供一个密匙 AK，用来加密 Alice 的资产 asset_1。所以最终 db chain 上存在的是加密后的 asset_1 即 encrypted_asset_1，密匙 AK 由 Alice 自己妥善保管。

数据上传前，Alice 可以对资源补充 Meta 信息，便于未来资源需求方的检索。

数据上传后，Alice 会得到一个 asset id，该 id 用来在 db chain 中标识该资产。

### 资源检索

资源需求方 Bob 可以通过在 db chain 中检索 Meta 字段，来查询满足自己条件的资源

### 注册资产

Bob 通过搜索资源的 Meta 信息，发现 Alice 的资源 encrypted_asset_1 满足其需求，于是他像 Alice 询问购买价格，当双方协商出具体的价格后，Alice 需要将资源变为待售状态，即挂单操作。

为了进行挂单操作，Alice 需要通过 SDK 调用 contract 方法：

```c
string sell(address from, string asset, uint price);
```

并等待来自 contract 的事件通知。其中 `from` 即 Alice 的地址，`asset` 即 `asset id`，`price` 即双方协商后的价格。调用该方法后，会返回一个 bill hash，Alice 需要将该 bill hash 告知 Bob

### 资源购买

在 Alice 挂单完成后，可以通知 Bob 该资源进行完成挂单，并告知其 bill hash，Bob 可以对该 bill hash 进行买单操作。

Bob 使用 SDK 调用 contract 方法

```c
bool buy(address from, string bill,  uint price);
```

来购买资源。其中 `from` 即 Bob 的地址，`bill` 即 `bill hash`，`price` 即双方协商的价格。

如果购买成功，Alice 将会接受到来自 contract 的事件通知，告知 Alice 购买方 Bob 的 public key。

Alice 接到购买通知后，需要完成如下动作：

1. 向 db chain 中追加(append) asset 信息，即使用 Bob 的 public key 加密的 AK
2. 将该资源 transfer 给 Bob

Bob 会接受来自 db chain 的事件通知，得知资源已经被转移到自己名下，此时 asset 在 db chain 中的结构类似：

```json
{
  "data": "encrypted data",
  "key": "Bob's public encrypted key of above data"
}
```

这时，Bob 就可以使用自己的私钥解开被自己公钥加密的 AK, 进而解开使用 AK 加密的资源。

如果 Bob 发现资源没有问题，则需要通过调用合约方法：

```c
bool deal(string bill);
```

来结束交易，此时交易金额进行将会转移给 Alice，在此之前，交易金额将会被固定在合约中。

## 合约实现

具体的合约实现可以参考 [sdk-eth-connector](https://github.com/BDNTeam/sdk-eth-connector)
