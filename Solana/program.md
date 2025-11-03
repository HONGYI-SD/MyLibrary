# key points

- solana 智能合约，为栈空间分配4K， 堆空间分配32K（实际使用30K）。

- anchor 框架下， account 支持10MB空间，需要手动扩容。
- 预分配的空间，必须全部扩容完才能开始使用。