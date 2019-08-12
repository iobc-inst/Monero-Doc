# POW简介
>工作量证明是使不信任的一方合法化的一种方式
## 究竟什么是POW？
工作量证明是一种加密证明，即不受信任的一方承诺提供大量算力解决谜题。

从技术上讲，“共识”只是解决当前问题的方法。
## 一切都是为了让不受信任的一方合法化

互联网上不受信任的一方如何能获得你的信任？

它可以通过解决预置的计算难题来证明其承诺。

例如，在与对方连接之前，要求非信任的一方执行硬计算，你只将连接开放给“已提交”的一方。

再例如，你可以对收到的电子邮件进行POW计算，从而增加发送垃圾邮件的成本。
## 工作必须“无用”

“计算难题”的工作及其解决方案除了证明承诺（Commitment）之外，不能有其他用处。

如果这项工作在其他地方有用，那么它并不能证明对你的承诺。

问题必须是人工设置的。否则奖励就会失效，整套模式（scheme）也会崩溃。
## 强非对称
工作量证明模式的要求是工作与验证资源之间存在强非对称性。

工作必须是随机的努力。同时，对验证的证明必须保持成本廉价（就计算资源而言）。

廉价验证至关重要，因为在这个阶段我们要处理大量潜在的非信任方，他们可以通过提交无效证据进行DOS攻击验证方。这类“伪证明”需要能被轻易去除。
# 加密货币中的POW
## PoW防止女巫攻击 
在分布式的加密货币中，交易（区块）是由非信任的参与者确认的。

如果以账户的方式投票，这套方案会很容易崩溃——如果有人伪造多个身份来操纵投票的话，你是没办法阻止的。在分布式系统中，这被称为女巫（Sybil）攻击。

相反，加密货币使用工作量证明。在工作量证明的模式中，重要的不是身份数量。它提供的是计算能力的证明。

这就会让操纵全盘困难得多。即使想威胁到这套模式，攻击者也必须控制大部分（> 50％）的计算资源。在实践中，这会大大增加攻击者的时间和算力成本。
## PoW是领导者选举机制 

在分布式系统中，“领导者选举”是确定哪个节点负责（临时）协调系统的过程。

在加密货币中，PoW用于选择出“赢得”下一个区块的节点。

使用PoW进行领导者选举是比特币引入的关键发明之一。

竞争节点（称为“矿工”）致力于解决计算难题。每隔一段时间，就会有节点找到解决方案，整个过程非常随机。机会与承诺的计算能力成线性比例。

获胜者使用其解决方案“覆写”其汇总的区块。网络仅接受具有有效解决方案的区块。

获胜者也会获得区块奖励。奖励是从无到有创造出的特定数量的加密货币，这部分奖励会发送到获胜者的账户。同时，获胜者还可以从区块中拿到交易费。

PoW问题的难度由网络动态调整，目标是以大致恒定的速率（通常每隔几分钟）找到区块。
# CryptoNight
## 背景 
CryptoNight最初是在2013年左右设计的，属于CryptoNote套件的一部分。

它的设计目标之一是通过采用以下方式，使其对目前的CPU比较友好：

* 本地AES加密
* 快速64位乘法器
* 适合精简版英特尔CPU上每核三级缓存的大小（约2MB）

更雄心勃勃的设计目标是使其在ASIC上无法有效计算。这个目标已经失败，因为“ASIC硬算法“能计算成功是不可避免的。高效的CryptoNight ASIC由Bitmain于2017年开发。

2014年，Monero将CryptoNight采用为自己的POW。在此之后，Monero略微改进了算法，故意破坏与已发布的ASIC-s的兼容性。目前，Monero实现了CryptoNight v2，这是对CryptoNight（v0，v1，v2）的第三次迭代。
## 目标是找到足够小的哈希
在基于哈希的PoW算法中，目标是找到足够小的哈希。

哈希只是一个整数（通常是一个非常大的整数）。大多数哈希函数生成256位哈希（0到2 ^ 256之间的整数）。这包括比特币的双SHA-256和Monero的CryptoNight。

矿机会随机调整输入数据，直至哈希符合指定的随机数。随机数（也是一个大整数）作为共识机制的一部分，由网络共同建立。如果哈希符合随机数，则仅认为PoW有效（已解决）。

因为哈希函数是单向的，所以足够小的哈希值是不可能通过分析计算出的。解决方案必须通过调整输入数据并反复重新计算才能得到。

矿工在输入方面有一些灵活性——最重要的是他们可以对随机数的值进行迭代。它们还具有将区块中包含交易组合在默克尔树（merkle）中，以及选择以什么样的方式组合的能力。
## 加密函数
CryptoNight基于：
* AES加密
* 5个哈希函数，全部都是NIST SHA-3的竞争函数：
  * Keccak（主要的）
  * BLAKE
  * Groestl
  * JH
  * Skein
## 输入数据 
在Monero中，哈希函数的输入是以下内容的连接：
* 序列化的区块头（大约46个字节;以varint格式表示）
* merkle树的根（32字节）
* 区块中包含的交易数（大约1-2个字节;以varint格式表示）

请参阅[get_block_hashing_blob（）]([https://github.com/monero-project/monero/blob/master/src/cryptonote_basic/cryptonote_format_utils.cpp#L1078](https://github.com/monero-project/monero/blob/master/src/cryptonote_basic/cryptonote_format_utils.cpp#L1078))函数以进一步探索。
## 算法 

>警告
>本文试图让读者对CryptoNight算法有更深入的理解。有关实现的详细信息，请参阅CryptoNote Standard和Monero源代码。请参阅底部的参考资料。
### 概览
CryptoNight试图使内存访问成为性能的瓶颈（“内存硬度”，memory hardness）。它有三个步骤：
* 使用伪随机数据初始化大面积内存。此内存称为暂存器。
* 在暂存器上的伪随机（但确定性）地址执行大量读/写操作。
* 哈希计算整个暂存器以产生结果。
### 第1步：暂存器初始化 
首先，使用Keccak-1600对输入数据进行哈希计算，生成200字节的伪随机数据（1600位== 200字节）。

使用AES-256加密，这200个字节成为生成更大的2MB宽的伪随机数据缓冲区的种子。

Keccak-1600哈希的前0-31字节用作AES密钥。

加密在128字节长的有效负载上执行，直到2MB准备就绪。第一个有效载荷是Keccak-1600的第66-191字节。下一个有效负载是先前有效负载的加密结果。

每个128字节的有效负载实际上加密了10次。

详细信息稍微有些细节，请参阅CryptoNote Standard中的[“Scratchpad Initialization”]([https://cryptonote.org/cns/cns008.txt](https://cryptonote.org/cns/cns008.txt))。
### 第2步：内存硬循环 
第二步是基本的简单重复524288次迭代。

每次计算迭代都会在在伪随机但确定的位置读取，并写回到暂存器。

关键的是，下一次迭代取决于先前迭代准备的状态，无法直接计算未来迭代的状态。

具体操作包括AES，XOR，8byte_mul，8byte_add这类对CPU友好的操作（在现代CPU上高度优化）。

这里的目标是使内存延迟成为填补潜在ASIC-s和通用CPU-s之间差距的瓶颈。
### 第3步：哈希计算
最后一步（简化）是：

* 使用上面的Keccak-1600输出
* 根据结果​​的2个低位选择哈希算法
  * 0=BLAKE-256
  * 1=Groestl-256
  * 2=JH-256 
  * 3=Skein-256
* 使用所选函数进行哈希计算

得到的256位哈希是CryptoNight算法的最终输出。
## Monero特定修改 
### CryptoNightv0 
Monero社区引用的CryptoNight原始实现。

### CryptoNightv1 
查看[源代码](https://github.com/monero-project/monero/pull/3253/files)差异。
### CryptoNightv2 
请参阅[基本原理](https://github.com/SChernykh/xmr-stak-cpu/blob/master/README.md)和[源代码](https://github.com/monero-project/monero/commit/5fd83c13fbf8dc304909345e60a853c15b0de1e5#diff-7000dc02c792439471da62856f839d62)差异。
### CryptoNight v3又名CryptoNightR 
请参阅[基本原理](https://github.com/monero-project/monero/pull/5126)和[源代码](https://github.com/monero-project/monero/pull/5126/files)差异。

查看源代码差异。
## 批评
* CryptoNight哈希验证相对昂贵。这会因为给DoS-ing节点带来伪证明而增加风险。 可查看[强非对称性](https://monerodocs.org/proof-of-work/what-is-pow/#strong-asymmetry)的要求。
* 哈希函数是从头开始设计的，同行评审有限。 虽然CryptoNight由经过验证和同行评审的函数组成，但组合在一起的加密系统并不一定真的会足够安全。
* CryptoNight最终未能阻止ASIC-s。
* CryptoNight的复杂性阻碍了ASIC制造业的竞争。

CryptoNight工作证明仍然是门罗币最具争议的方面之一。
## 参考 

* [CryptoNight hash function](https://cryptonote.org/cns/cns008.txt) 标准CryptoNote描述
* [CryptoNight v2 source code](https://github.com/monero-project/monero/blob/master/src/crypto/slow-hash.c)
  * 入口点是cn_slow_hash（）函数。 手动删除多个体系结构的支持和优化应该有助于您了解实际代码。
*   [CryptoNote whitepaper](https://downloads.getmonero.org/whitepaper_annotated.pdf) 的平等工作量证明（"Egalitarian Proof of Work"）章节
* David Andersen博士撰写的[First days of Monero mining](https://da-data.blogspot.com/2014/08/minting-money-with-monero-and-cpu.html) 
* 门罗币源码的一些 [test vectors](https://github.com/monero-project/monero/tree/master/tests/hash) 

