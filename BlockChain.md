# 区块链笔记
    共识机制：
        PoW（工作量证明）：通过寻找一个随机数并进行双哈希计算，目标是找到一个符合特定难度标准的值，这一过程主要考验算
        PoS（权益证明）：不仅考验算力，还依赖于持有的币的数量，持币越多，获得记账权的机会越大。
        PoA (权威证明):私有区块链或联盟链使用，牺牲部分中心化，预定审核的验证者轮询获得记账权。
    以太坊虚拟机：EVM，运行在以太坊网络的各个节点上，可以执行智能合约代码。
    Gas机制：
        1、原计价方式，Gas fee= gas price * gas limit，即单价*最大 gas 数量
        2、新计价方式，TransactionCost=MaxFees * GasUsedTransactionCost = MaxFeesxGasUsed
            BaseFees：在以太坊上发送交易或完成操作所需的最低 Gas 价格。基本费由协议动态调整，并且这些基本费会被销毁，不会支付给矿工。
            PriorityFees（Tip）：优先级费用（也叫小费），用户支付给矿工的小费，以便矿工优先处理自己的交易或操作。用户可以自行设置这个费用，默认情况下，小费预计为 2 Gwei。
            MaxFees：等于 BaseFees 加上 PriorityFees，相当于之前的 GasPrice，但增加了一个“最高”的定语。实际用到的小费可能不会达到预设的 PriorityFees 这么高，剩余部分会退还。

    账户：
        外部账户：管理私钥，包含了余额等状态信息。
        合约账户：由外部账户创建，有合约代码控制。
    状态：
        nonce: 对于外部账户，这个字段表示该账户发出的交易数量，用以防止重复交易（双花问题）。对于合约账户，nonce 表示该账户创建的合约数量。
        balance: 账户的余额，单位为 wei。1 ether 等于 1018 wei。
        storageRoot: 这是一个指向 Merkle Patricia Tree（MPT）根节点的哈希值，树中存储了账户的状态信息。
        codeHash: 
                在外部账户中，这个字段通常是一个空字符串的哈希值；
                
                而在合约账户中，它表示账户中 EVM 代码（编译后的合约字节码）的哈希。当合约账户接收到一个消息调用时，相关代码将被执行。不同于其他字段，codeHash 是不可变的，它用作从状态数据库检索相应 EVM 代码的索引。
    以太坊节点：
        全节点：Full Node，存储并验证以太坊区块链的完整数据：括区块头（Header）、交易列表（Body）、状态树（StateRoot）、交易树（TxRoot）、收据树（ReceiptRoot）等
        轻节点：Light Node，轻节点仅下载并存储区块头（Block Header，依赖全节点提供具体交易数据
    以太币：
        最小单位wei，和最大单位ether,以及其他单位。
    智能合约：
        通常使用Solidity来实现，适合明确规则可自动化执行的程序。
        无需第三方执行： 智能合约自动执行，减少了合约执行的人力、物力和时间成本。
        不可更改： 合约条款一旦确定，不受人为干预，降低了用户受骗风险。
        安全可靠： 合约存储在区块链上，防止丢失风险。