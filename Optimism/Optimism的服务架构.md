# Optimism 服务架构

```mermaid
graph TB
    subgraph Layer1
        L1[以太坊主网]
    end

    subgraph Layer2
        direction TB
        op-node[op-node<br/>共识节点<br/>负责区块生成和同步]
        op-geth[op-geth<br/>执行客户端<br/>处理交易执行]
        op-batcher[op-batcher<br/>批处理器<br/>将L2交易打包提交到L1]
        op-proposer[op-proposer<br/>状态提议者<br/>提交L2状态输出到L1]
        op-challenger[op-challenger<br/>挑战者<br/>验证状态输出正确性]
    end

    %% 服务间调用关系
    op-node -->|Engine API| op-geth
    op-node -->|P2P Gossip| op-node
    op-batcher -->|RPC| op-node
    op-batcher -->|提交交易批次| L1
    op-proposer -->|RPC| op-node
    op-proposer -->|提交状态输出| L1
    op-challenger -->|验证| L1

    %% 核心合约
    subgraph L1合约
        direction TB
        OptimismPortal[OptimismPortal<br/>跨链消息传递]
        L2OutputOracle[L2OutputOracle<br/>状态输出验证]
        L1CrossDomainMessenger[L1CrossDomainMessenger<br/>跨链消息管理]
        L1StandardBridge[L1StandardBridge<br/>资产跨链桥]
    end

    L1 --- OptimismPortal
    L1 --- L2OutputOracle
    OptimismPortal --- L1CrossDomainMessenger
    L1CrossDomainMessenger --- L1StandardBridge

    classDef default fill:#f9f9f9,stroke:#333,stroke-width:2px;
    classDef L1 fill:#ffcccc,stroke:#ff0000;
    classDef L2 fill:#ccffcc,stroke:#00ff00;
    class L1 L1;
    class op-node,op-geth,op-batcher,op-proposer,op-challenger L2;
```

## 核心服务说明

### op-node
- 功能：作为共识层客户端，负责区块生成和同步
- 接口：
  - Engine API：与执行层(op-geth)通信
  - P2P：节点间数据同步
  - RPC：供其他服务调用

### op-geth
- 功能：作为执行层客户端，处理交易执行
- 接口：
  - Engine API：与共识层(op-node)通信
  - JSON-RPC：标准以太坊RPC接口

### op-batcher
- 功能：将L2交易打包并提交到L1
- 接口：
  - RPC：从op-node获取交易数据
  - 合约调用：向L1提交交易批次

### op-proposer
- 功能：将L2状态输出提交到L1
- 接口：
  - RPC：从op-node获取状态数据
  - 合约调用：向L2OutputOracle提交状态输出

### op-challenger
- 功能：验证L2状态输出的正确性
- 接口：
  - 合约调用：验证L2OutputOracle中的状态输出

## 核心合约说明

### OptimismPortal
- 功能：处理L1和L2之间的跨链消息传递
- 接口：存款和提款交易处理

### L2OutputOracle
- 功能：验证和存储L2状态输出
- 接口：状态输出提交和验证

### L1CrossDomainMessenger
- 功能：管理跨链消息的传递
- 接口：消息打包和解析

### L1StandardBridge
- 功能：处理ETH和ERC20代币的跨链
- 接口：存款和提款操作