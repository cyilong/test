# Optimism 组件流程图

```mermaid
graph TB
    subgraph Layer1[以太坊主网]
        L1[L1区块链]
        OptimismPortal[OptimismPortal合约<br/>处理跨链消息]
        L2OutputOracle[L2OutputOracle合约<br/>验证状态输出]
        L1CrossDomainMessenger[L1CrossDomainMessenger合约<br/>跨链消息管理]
        L1StandardBridge[L1StandardBridge合约<br/>资产跨链]
    end

    subgraph Layer2[Optimism网络]
        op-node[op-node<br/>共识层客户端<br/>负责区块生成和同步]
        op-geth[op-geth<br/>执行层客户端<br/>处理交易执行]
        op-batcher[op-batcher<br/>批处理器<br/>将L2交易打包提交到L1]
        op-proposer[op-proposer<br/>状态提议者<br/>提交L2状态输出到L1]
    end

    %% 组件间的交互关系
    op-node -->|Engine API| op-geth
    op-node -->|P2P同步| op-node
    op-batcher -->|获取交易数据| op-node
    op-batcher -->|提交交易批次| L1
    op-proposer -->|获取状态数据| op-node
    op-proposer -->|提交状态输出| L2OutputOracle

    %% 跨链消息流程
    L1 -->|区块数据| op-node
    OptimismPortal -->|处理存款交易| L1CrossDomainMessenger
    L1CrossDomainMessenger -->|跨链资产| L1StandardBridge
    L1StandardBridge -->|锁定/释放资产| L1

    %% 状态验证流程
    L2OutputOracle -->|验证状态| OptimismPortal
    OptimismPortal -->|确认提款| L1

    classDef L1Fill #ffcccc,stroke:#ff0000;
    classDef L2Fill #ccffcc,stroke:#00ff00;
    class L1,OptimismPortal,L2OutputOracle,L1CrossDomainMessenger,L1StandardBridge L1Fill;
    class op-node,op-geth,op-batcher,op-proposer L2Fill;
```

## 主要组件说明

### Layer 1组件
- **OptimismPortal**: 处理L1和L2之间的跨链消息传递
- **L2OutputOracle**: 验证和存储L2状态输出
- **L1CrossDomainMessenger**: 管理跨链消息的传递
- **L1StandardBridge**: 处理资产的跨链转移

### Layer 2组件
- **op-node**: 作为共识层客户端，负责区块生成和同步
- **op-geth**: 作为执行层客户端，处理交易执行
- **op-batcher**: 将L2交易打包并提交到L1
- **op-proposer**: 将L2状态输出提交到L1

## 主要流程说明

1. **区块同步流程**
   - op-node从L1获取区块数据
   - op-node通过Engine API与op-geth通信
   - op-node之间通过P2P网络同步数据

2. **交易处理流程**
   - op-batcher从op-node获取交易数据
   - op-batcher将交易打包后提交到L1
   - op-geth执行交易并更新状态

3. **状态提交流程**
   - op-proposer从op-node获取状态数据
   - op-proposer将状态输出提交到L2OutputOracle合约
   - L2OutputOracle验证状态输出

4. **跨链消息流程**
   - OptimismPortal处理跨链消息
   - L1CrossDomainMessenger管理消息传递
   - L1StandardBridge处理资产跨链