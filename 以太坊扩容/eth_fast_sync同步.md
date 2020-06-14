[eth_fast_sync](https://github.com/ZtesoftCS/go-ethereum-code-analysis/blob/master/%E4%BB%A5%E5%A4%AA%E5%9D%8Afast%20sync%E7%AE%97%E6%B3%95.md)

## 缺点

fast节点同步容易受到女巫攻击

Blockchain protocols in general (i.e. Bitcoin, Ethereum, and the others) are susceptible to Sybil attacks, where an attacker tries to completely isolate a node from the rest of the network, making it believe a false truth as to what the state of the real network is. This permits the attacker to spend certain funds in both the real network and this "fake bubble". However, the attacker can only maintain this state as long as it's feeding new valid blocks it itself is forging; and to successfully shadow the real network, it needs to do this with a chain height and difficulty close to the real network. In short, to pull off a successful Sybil attack, the attacker needs to match the network's hash rate, so it's a very expensive attack.

常见的区块链(比如比特币，以太坊以及其他)是比较容易受女巫攻击的影响，攻击者试图把被攻击者从主网络上完全隔离开，让被攻击者接收一个虚假的状态。这就允许攻击者在真实的网络同时这个虚假的网络上花费同一笔资金。然而这个需要攻击者提供真实的自己锻造的区块，而且需要成功的影响真实的网络，就需要在区块高度和难度上接近真实的网络。简单来说，为了成功的实施女巫攻击，攻击者需要接近主网络的hash rate，所以是一个非常昂贵的攻击。

Compared to the classical Sybil attack, fast sync provides such an attacker with an extra ability, that of feeding a node a view of the network that's not only different from the real network, but also that might go around the EVM mechanics. The Ethereum protocol only validates state root hashes by processing all the transactions against the previous state root. But by skipping the transaction processing, we cannot prove that the state root contained within the fast sync pivot point is valid or not, so as long as an attacker can maintain a fake blockchain that's on par with the real network, it could create an invalid view of the network's state.

与传统的女巫攻击相比，快速同步为攻击者提供了一种额外的能力，即为节点提供一个不仅与真实网络不同的网络视图，而且还可能绕过EVM机制。 以太坊协议只通过处理所有事务与以前的状态根来验证状态根散列。 但是通过跳过事务处理，我们无法证明快速同步pivot point中包含的state root是否有效，所以只要攻击者能够保持与真实网络相同的假区块链，就可以创造一个无效的网络状态视图。

To avoid opening up nodes to this extra attacker ability, fast sync (beside being solely opt-in) will only ever run during an initial sync (i.e. when the node's own blockchain is empty). After a node managed to successfully sync with the network, fast sync is forever disabled. This way anybody can quickly catch up with the network, but after the node caught up, the extra attack vector is plugged in. This feature permits users to safely use the fast sync flag (--fast), without having to worry about potential state root attacks happening to them in the future. As an additional safety feature, if a fast sync fails close to or after the random pivot point, fast sync is disabled as a safety precaution and the node reverts to full, block-processing based synchronization.

为了避免将节点开放给这个额外的攻击者能力，快速同步(特别指定)将只在初始同步期间运行(节点的本地区块链是空的)。 在一个节点成功与网络同步后，快速同步永远被禁用。 这样任何人都可以快速地赶上网络，但是在节点追上之后，额外的攻击矢量就被插入了。这个特性允许用户安全地使用快速同步标志（--fast），而不用担心潜在的状态 在未来发生的根攻击。 作为附加的安全功能，如果快速同步在随机 pivot point附近或之后失败，则作为安全预防措施禁用快速同步，并且节点恢复到基于块处理的完全同步。