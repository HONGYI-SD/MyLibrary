# 使用 ordi 协议铸造铭文

## 启动 `bitcoin full node`

- bitcoin node 安装参考官方教程，安装最新版本即可

- 启动节点，其中测试网类型选择 signet, testnet3 坑比较多，暂不考虑。

```sh
bitcoind -signet -daemon -txindex
```

## 启动 `ord`

- ord 安装参考官方文档，安装最新版本

- 启动 ord server

```sh
ord --signet server --http-port 9988
```

- 创建钱包，默认名称为 ord `ord --signet wallet create`

```sh
ubuntu:$ ord --signet wallet create
{
  "mnemonic": "mercy hungry moment reveal during march degree put shoot pilot clump screen",
  "passphrase": ""
}
```

- 生成一个新的地址 `ord --signet wallet --server-url http://127.0.0.1:9988 receive`

```sh
ubuntu:$ ord --signet wallet --server-url http://127.0.0.1:9988 receive
{
  "addresses": [
    "tb1pl85p97dd9d86aehj2j4jr2vd00daj7rj7a5hf4qzvxejg7fclxgs854ayc"
  ]
}
```

- 通过水龙头向这个地址充点钱 [https://signetfaucet.com/]

- 铸造铭文 `ord --signet wallet --name wallet-dd --server-url http://127.0.0.1:9988 inscribe --fee-rate 1 --file domicon.txt`

```json
{
  "commit": "22aec4027ac3874e6f128b1cb749047f50557cb89a85bd5ae81a856181bb4500",
  "commit_psbt": null,
  "inscriptions": [
    {
      "destination": "tb1p9lj680gqjgvnywkl0muznf6tw2f2t66d0wjtm79nx26frtaqwhuqkgwu9d",
      "id": "734768c29ee35e64a0ee52f933c16c841af18ecda1775e288b00e2859333ac3ei0",
      "location": "734768c29ee35e64a0ee52f933c16c841af18ecda1775e288b00e2859333ac3e:0:0"
    }
  ],
  "parent": null,
  "reveal": "734768c29ee35e64a0ee52f933c16c841af18ecda1775e288b00e2859333ac3e",
  "reveal_broadcast": true,
  "reveal_psbt": null,
  "rune": null,
  "total_fees": 295
}
```

- 查看铭文信息 `ord --signet wallet --name wallet-dd --server-url http://127.0.0.1:9988 inscriptions`

```json
[
  {
    "inscription": "734768c29ee35e64a0ee52f933c16c841af18ecda1775e288b00e2859333ac3ei0",
    "location": "734768c29ee35e64a0ee52f933c16c841af18ecda1775e288b00e2859333ac3e:0:0",
    "explorer": "https://signet.ordinals.com/inscription/734768c29ee35e64a0ee52f933c16c841af18ecda1775e288b00e2859333ac3ei0",
    "postage": 10000
  }
]
```

## 其他命令

- 创建钱包并指定名称 `bitcoin-cli -signet createwallet "wallet-dd"`

- 通过指定wallet创建新的地址

```sh
bitcoin-cli -signet -rpcwallet=wallet-dd getnewaddress
tb1qp746zfeys8fw53k7kklhxm9qegsly7r56pjnkc
```

- 发送交易 `bitcoin-cli -signet -rpcwallet="wallet-dd" sendtoaddress "tb1qp746zfeys8fw53k7kklhxm9qegsly7r56pjnkc" 0.00001`

```txt
98c7a07fecc1a352ba5391a16caf2d284a10a7c0580d8539d8d073405dc31ecb
```

- ord 启动服务, ubuntu 默认不让启动80端口，需要指定其他端口

```sh
ord --signet server --http-port 9988
```

- ord 会使用 bitcoin 的索引，启动 bitcoin 节点需指定 -txindex

```sh
bitcoind -signet -daemon -txindex
```

- 查看 index 信息

```sh
bitcoin-cli -signet -datadir=./.bitcoin getindexinfo
```
