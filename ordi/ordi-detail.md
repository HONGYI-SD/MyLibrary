- ORDINAL 的元数据并没有存储在一个特定的位置上，他们被嵌入到交易的见证数据（witness data）中，这些数据被像铭文一样刻在比特币交易中。

- 铭文铭刻的过程通过隔离见证（SegWit）和 “向Taproot支付"（Pay to Taproot，P2TR）的方式实现，其中包含了提交（commit）和 揭露（reveal）两个阶段，能够将任意形式的内容（如文本、图像或视频）铭刻在指定的聪上。

- SegWit 是将某些交易签名数据（见证数据）与交易分开。

- P2TR 是比特币的一种交易输出类型，它使得不同的交易条件可以更加隐私地存储在区块链中。

- 铭文本质上是将特定的数据内容嵌入到比特币交易中，P2TR 使得这种嵌入数据变得更加灵活和经济。

---

- Witness 数据存储在什么位置？

```txt
    witness 数据同样存放在区块链的 block 中，但是 witness 的引入，改变了区块大小的计算方式。
    它引入了一种新的计算区块大小的方法，即”区块重量“，区块重量最大为4,000,000重量单位（WU）,而不是原来的 1MB 限制。区块重量的计算方式如下：
    
    非 witness 数据： 每个字节记为 4WU
    witness 数据：每个字节记为 1WU

    这样，同样大小的交易数据，如果包含了 witness 数据，其总重量会比不包含 witness 数据时要轻很多， 从而在同样的区块中可以打包更多的交易。
```

---

# ordi 创建铭文流程梳理

- 构建铭文内容信息:会标注铭文内容类型、铭文具体数据。

- 选定当下铭文需要绑定的聪的位置，用 satpoint 来标识，同时会检查该 satpoint 没有绑定有其他铭文。

- 生成钥匙对 key_pair, 并基于该 key_pair 生成一个 XOnlyPubKey, 它是和 taproot 相关的公钥，可以用来验证 schnorr 签名。

- 构建 reveal_script

```txt
reveal_script 中存放的内容 [XOnlyPubKey, OP_CHECKSIG, 铭文信息*n]
其中铭文信息格式如下：
OP_FALSE
OP_IF
  OP_PUSH "ord"
  OP_PUSH 1
  OP_PUSH "text/plain;charset=utf-8"
  OP_PUSH 0
  OP_PUSH "Hello, world!"
OP_ENDIF
```

- 基于 reveal_script 和 XOnlyPubKey 构建 TaprootSpendInfo

```txt
有2种条件可以花费 taproot
1. 使用 key path, 即 secret key 绑定的 TweakedPublicKey， 在 TaprootSpendInfo 中也就是成员 output_key
2. 满足脚本花费路径中的任意的脚本。
```

- 通过 TaprootSpendInfo 的 output_key 得到 commit_tx_address

- 构建 unsigned_commit_tx , 其中会指定 satpoint commit_tx_address

- 将 commit tx 中接收地址为 TweakedPublicKey 的 output 设置为 reveal 的 input

- 通过 control_block , reveal_input, reveal_script 等数据构建 reveal_tx

- 为脚本花费(应该是第二种花费taproot的情况)计算sighash， 这里有点复杂……

- 使用 key_pair 为 sighash做 schnorr签名

- 为 reveal_tx 中的 input 为 commit_tx 的交易设置 witness

- 1. 填充 schnorr 签名
- 2. 填充 reveal_script
- 3. 填充 control_block

- 用钱包为 commit_tx 签名并广播。
- 用钱包为 reveal_tx 签名，同时还要指定reveal_tx 中有使用 commit_tx 的output 作为 utxo，签名完广播出去。
