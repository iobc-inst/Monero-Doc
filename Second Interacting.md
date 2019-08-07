# 下载 Monero
单个存档包含开始使用 Monero 所需的所有内容(全节点和钱包)。
我们建议直接从 GitHub 下载 Monero 二进制文件:
* GUI + CLI:  https://github.com/monero-project/monero-GUI/releases
* CLI:  https://github.com/monero-project/monero/releases

GUI是一个图形化的桌面钱包。
CLI是一个命令行桌面钱包。
更多的相关信息可以查询 Monero 网站的[下载](https://web.getmonero.org/downloads/)版块。
请注意验证下载文件的签名。
## 下载哪个版本？ 
下载与你的操作系统和处理器架构匹配的最新版本。
CLI版本较早发布，适合服务器部署。
GUI 版本同时包含 CLI 和 GUI。 对用户来说，这点可以考虑。
所有版本都包含一个完整的节点和一个钱包。
## 为什么是Github而不是getmonero.org? 
GitHub上的文件发布得更早。
最重要的是，如果你没有正确地验证签名，GitHub 会更安全，仅仅是因为你不需要相信一个独立的网站就不会被破坏。 显然，你仍然应该仔细验证每个版本的签名。 签名认证一直是最主要的防线。

# 验证 Monero 二进制文件
在提取文件和使用 Monero 之前必须进行核查。
本文档中的命令已经在 Linux 上测试过了，稍微修改一下在 macOS也能运行。

## 1. 导入核心开发 PGP 密钥
这个操作一次就可以。安装后续的 Monero 版本时，可以省略这一步。
门罗币的开发团队在二进制包中做了哈希表的签名。
Riccardo“ fluffypony” Spagni 是 Monero 核心开发人员，由他负责对文件包签名。Riccardo的公钥可以在 GitHub 的项目源代码中找到。将Riccardo 的公钥导入你的密钥环（keyring）中:

```
curl https://raw.githubusercontent.com/monero-project/monero/master/utils/gpg_keys/fluffypony.asc | gpg --import
```

依次输入以下命令，信任Riccardo的公共密钥(指纹必须是这个字符串) :
```
gpg --edit-key 'BDA6BD7042B721C467A9759D7455C5E3C0CDCEB9'
trust
4
```

 
>**注意：**
>如果找不到这个密钥的指纹，需要立即移除导入的密钥(gpg -- delete-keys...)。找不到指纹意味着密钥被改变了(很可能被破坏了)。

 
1. 验证哈希列表的签名

二进制文件包及其哈希表的列表发布在 [getmonero.org](https://www.getmonero.org/downloads/hashes.txt) 和其他一些地方，如 [r/monero](https://reddit.com/r/monero) 上的发行说明。请注意，只要签名验证无误，发布通道无关紧要。

验证哈希列表是否被篡改 ，可以运行下面的命令：
```
curl https://www.getmonero.org/downloads/hashes.txt | gpg --verify
```

输出应该包含这样一行：

```
英文输出：
gpg: Good signature from "Riccardo Spagni <ric@spagni.net>" [full]

中文输出：
gpg: 完好的签名，来自于 “Riccardo Spagni <ric@spagni.net>”
```

1. 验证哈希表

通过上一步，我们确认了哈希列表没有被篡改。
最后一步是将发布的哈希与下载文件的 SHA-256哈希值进行比较。
如果你还没下载 Monero，现在可以[下载](https://monerodocs.org/interacting/download-monero-binaries)了(但不要解压缩)。然后将示例文件名替换为实际文件名：

```
file_name=monero-linux-x64-v0.14.0.0.tar.bz2

file_hash=`sha256sum $file_name | cut -c 1-64`

curl https://www.getmonero.org/downloads/hashes.txt &gt; /tmp/reference-hashes.txt

# verify the signature (previous step is repeated here for completeness)
# 验证签名（为了流程完整，在这里再重复一遍）
gpg --verify /tmp/reference-hashes.txt

# grep must print the hash (output cannot be empty)
# grep会输出哈希值（输出一定不为空）
grep $file_hash /tmp/reference-hashes.txt
```


>**注意：**
>如果 grep 输出是空的，那么再次检查所有内容，因为显然哈希值不匹配。
>如果 grep 打印文件名和一个哈希值，那么一切都是正常的！

 


# 与门罗币的交互
你可以通过桌面图形用户界面、命令行界面和编程 API 与门罗币交互。
最重要的是，门罗币的节点会在点对点网络中相互交互。
## 安装目录概述
解压后，你会看到几个可执行文件，以及一份很友好的门罗币图形化界面操作指南（pdf格式）。
门罗币项目很机智地将网络节点逻辑与钱包逻辑分离开来。钱包的逻辑通过三个独立的用户界面提供—— GUI、 CLI 和 HTTP API。
```
# cd monero-gui-v0.14.0.0
# ---- guide to Monero GUI ----
# ---- 门罗币图形化界面操作指南 ----
monero-GUI-guide.pdf

# ---- executable files -----------
# ---- 可执行文件 -----------
monerod

monero-wallet-cli
monero-wallet-gui
monero-wallet-rpc

monero-gen-trusted-multisig

monero-blockchain-export
monero-blockchain-import

monero-blockchain-mark-spent-outputs
monero-blockchain-usage
monero-blockchain-ancestry
monero-blockchain-depth

start-gui.sh

# ---- directories ----------------
# ---- 目录 ----------------
libs
plugins
qml
```
## 可执行文件说明
| 可执行文件   | 描述   | 
|:----|:----|
| monerod   | 全节点守护进程，不要求有钱包。  [说明文档](https://monerodocs.org/interacting/monerod-reference/).   | 
| monero-wallet-gui   | 钱包逻辑和图形化用户界面。需要monerod运行。   | 
| monero-wallet-cli   | 钱包逻辑和命令行界面。需要monerod运行。   | 
| monero-wallet-rpc   | 钱包逻辑和**HTTP API** (JSON-RPC 协议)。需要monerod运行。   | 
| monero-blockchain-export   | 将区块链导出至[blockchain.raw](https://downloads.getmonero.org/blockchain.raw)的工具。   | 
| monero-blockchain-import   | 导入[blockchain.raw](https://downloads.getmonero.org/blockchain.raw)的工具——理想情况下是你自己的受信任副本。   | 
| monero-gen-trusted-multisig   | 生成一组多重签名钱包的工具。  请参阅有关[多重签名](https://monerodocs.org/multisignature)的章节。   | 
| monero-blockchain-mark-spent-outputs   | 用于缓解与Monero forks相关的潜在隐私问题的高级工具。一般不需要你操心。  详情请参阅[提交](https://github.com/monero-project/monero/commit/df6fad4c627b99a5c3e2b91b69a0a1cc77c4be14#diff-0410fba131d9a7024ed4dcf9fb4a4e07)和[拉取](https://github.com/monero-project/monero/pull/3322)请求。   | 
| monero-blockchain-usage   | 用于缓解与Monero forks相关的潜在隐私问题的高级工具。一般不需要你操心。  详情请参阅[提交](https://github.com/monero-project/monero/commit/0590f62ab64cf023d397b995072035986931a6b4)和[拉取](https://github.com/monero-project/monero/pull/3322)请求。 | 
| monero-blockchain-ancestry   | 前沿的研究工具，可以用来探究交易，区块或链的脉络。普通用户可以不用操心。 请参阅此[拉取](https://github.com/monero-project/monero/pull/4147/files)请求。   | 
| monero-blockchain-depth   | 前沿的研究工具，可以用来探究交易，区块或链的深度。普通用户可以不用操心。 请参阅此[拉取](https://github.com/monero-project/monero/commit/289880d82d3cb206a2cf4ae67d2deacdab43d4f4#diff-34abcc1a0c100efb273bf36fb95ebfa0)请求。   | 

## 交互 
与门罗币交互的方法有很多。对于新手来说，有一点特别令人意外——monerod 守护进程在运行时，也能接受命令。

需要注意的是，HTTP API 分为 monerod 和 monero-wallet-rpc 两种。你需要同时运行并调用这两个守护进程，这样才能探索完整的 API。这点遵循的是前面提到的——网络节点逻辑与钱包逻辑的分离。

钱包的所有执行都依赖于** monerod** 的运行。
| Executable   | p2p network   | node commands via keyboard   | node HTTP API   | wallet commands via keyboard   | wallet HTTP API   | wallet via GUI   | 
|:----|:----|:----|:----|:----|:----|:----|
| monerod   | ✔   | ✔   | ✔   |    |    |    | 
| monero-wallet-cli   |    |    |    | ✔   |    |    | 
| monero-wallet-rpc   |    |    |    |    | ✔   |    | 
| monero-wallet-gui   |    |    |    |    |    | ✔   | 

## 数据目录 
区块链、日志文件和 p2p 网络存储器的存储位置如下。
默认的数据目录是：
* $HOME/.bitmonero/  # Linux系统
* $HOME/Library/Application\ Support/ # macOS系统
* C:\ProgramData\bitmonero\  # Windows系统

请注意:
* 按照操作系统约定，数据目录默认隐藏
* bitmonero 这个名字是历史文物，它来源于约公元前2000年，Monero分离出Bitmonero之前的历史人工制品

数据目录包含：
* lmdb/ #区块链数据库目录
* p2pstate.bin - # 储存发现和被评过级的节点 
* bitmonero.log # 日志文件

它还可以包含 stagenet 和 testnet 的子目录，反映了相同的结构：
* Stagenet /# Stagenet的数据目录
* Testnet / # Testnet的数据目录
# monerod - 参考 
## 概述
### 连接到门罗币网络
门罗币的守护进程 monerod 能保持你的计算机与门罗币网络同步。它从p2p网络下载并验证区块链。
### 没人知道你的私钥
Monerod 和你的钱包完全脱钩了。Monerod 不访问你的私钥——它没法知道你的交易和收支。所以你可以在单独的计算机或云端运行 monerod 。
事实上，你可以连接到半受信任的第三方提供的远程 monerod 实例。 这样的第三方并不能窃取你的资金。对于学习和实验这非常方便。
但是，使用远程、不可信节点会产生隐私和可靠性问题。对于任何真正的业务，你都应该运行自己的全节点。
## 语法
```
./monerod [options] [command]
./monerof [选项][命令]
```
选项定义守护进程应该如何工作。它们的名称遵循 --option-name 模式。
命令可以访问守护进程提供的特定服务。即使守护进程正在执行，命令也可以执行。 它们的名称遵循 command_name 模式。
## 运行
进入你的门罗币的目录。
在学习和测试阶段中，你最好使用[ stagenet](https://monerodocs.org/infrastructure/networks)。
```
./monerod --stagenet --detach                # 在后台运行守护进程
tail -f ~/.bitmonero/stagenet/bitmonero.log  # 查看日志文件
./monerod --stagenet exit                    # 要求守护进程退出
```

处理真正的门罗币（XMR）时，就需要 [mainnnet ](https://monerodocs.org/infrastructure/networks)了。

```
./monerod --detach                           # 在后台运行守护进程
tail -f ~/.bitmonero/bitmonero.log           # 查看日志文件
./monerod exit                               # 要求守护进程退出
```
## 选项
选项定义守护进程应该如何工作，它们的名称遵循 --option-name 模式。
下面的分类只是为了让参考更容易理解。守护进程本身不以任何方式对选项进行分组。

### 帮助和版本
| 选项   | 描述 | 
|:----|:----|
| --help   | 列出可用选项。   | 
| --version   | 标准输出 monerod的版本，比如：Monero 'Boron Butterfly' (v0.14.0.0-release)   | 
| --os-version   | 显示构建时间戳和目标操作系统。输出示例：  OS: Linux #1 SMP PREEMPT Fri Aug 24 12:48:58 UTC 2018 4.18.5-arch1-1-ARCH.   | 

### 挑选网络
| 选项   | 描述   | 
|:----|:----|
| (missing)   | 默认情况下，monerod假定为[mainnet](https://monerodocs.org/infrastructure/networks)。   | 
| --stagenet   | 在[stagenet](https://monerodocs.org/infrastructure/networks)上运行。记得用--stagenet运行你的钱包。   | 
| --testnet   | 在[ testnet ](https://monerodocs.org/infrastructure/networks)上运行。记得用 --testnet 运行你的钱包。   | 

### 日志
| 选项   | 描述   | 
|:----|:----|
| --log-file   | 日志文件的完整路径。 示例（注意文件权限）：  ./monerod --log-file=/var/log/monero/mainnet/monerod.log   | 
| --log-level   | 0-4：0表示最小日志记录，4表示完全跟踪，默认为0。这些是常规预设，不需要调到最高级别。 例如，即使调到0档，你也可能会看到一些最重要的INFO条目。暂时更改为1可以更好地了解整个节点的运行方式。操作示例：   ./monerod --log-level=1   | 
| --max-log-file-size   | 日志文件的软性限制（以字节为单位，大小默认为104850000bytes，刚好小于100MB)。 一旦日志文件超过该限制，monerod将使用UTC时间戳，以YYYY-MM-DD-HH-MM-SS的名字创建下一个日志文件。    在开发部署中，你可能更喜欢使用成型的解决方案，例如 logrotate。在这种情况下，请设置--max-log-file-size = 0以防止 monerod 管理日志文件。   | 
| --max-log-files   | 限制日志文件的数量（默认为50）。最旧的日志文件会被删除。在开发部署中，你可能更喜欢使用成型的解决方案，例如logrotate。   | 

### 服务
当你在同一台计算机上运行门罗币钱包时，门罗币允许你偶尔调整其默认值。
如果你打算让一个节点(比如远程服务器或你自己的电脑)一直运行，下面的选项会很有帮助。

| 选项   | 描述   | 
|:----|:----|
| --config-file   | [配置文件](https://monerodocs.org/interacting/monero-config-file)的完整路径。默认情况下，monerod 在 门罗币的[数据目录](https://monerodocs.org/interacting/overview/#data-directory)中查找bitmonero.conf。   | 
| --data-dir   | 数据目录的完整路径。这是存储区块链，日志文件和p2p网络内存的地方。有关默认值和详细信息，请参阅[数据目录](https://monerodocs.org/interacting/overview/#data-directory) 。   | 
| --pidfile   | PID文件的完整路径。仅适用于 --detach 。示例：  ./monerod --detach --pidfile=/run/monero/monerod.pid   | 
| --detach   | 转至后台（从终端解耦）。对长时间运行/服务器的方案会很有用。一般情况下，你还需要使用 systemd 或类似方法管理 monerod 守护程序。默认情况下，monerod 在前台运行。   | 
| --non-interactive   | Do not require tty in a foreground mode. Helpful when running in a container. By default monerod runs in a foreground and opens stdin for reading. This breaks containerization because no tty gets assigned and monerod process crashes. You can make it run in a background with --detach  but this is inconvenient in a containerized environment because the  canonical usage is that the container waits on the main process to exist  (forking makes things more complicated).  在前台模式下不需要tty（但在容器中运行时，tty会很有帮助）。默认情况下，monerod 在前台运行并打开stdin进行读取。不给TTY分配进程会打破容器化，monerod进程会崩溃。你可以用 --detach 使tty在后台运行，如果是在容器化环境中，这会不太方便，因为规范用法是容器等待主进程存在（分叉会让事情更复杂）。   | 
| --no-igd   | 禁用路由器上的UPnP端口映射（“Internet网关设备”）。如果你不使用NAT转换的话（可以直接绑定到公共IP或通过Tor运行），请添加此选项以提高安全性。   | 
| --max-txpool-weight   | 设置交易池的大小（单位为字节）。默认为648000000（~618MB）。这些是待确认的交易（未包含在任何块中）。   | 
| --enforce-dns-checkpointing   | 由 [MoneroPulse](https://monerodocs.org/infrastructure/monero-pulse/) 设置的检查节点是强制执行的。无人参与的节点比较适合设置强制执行。    碰到区块的哈希与相应的检查节点不匹配，本地区块链将会回滚几个区块，有效避免MoneroPulse 运算符认为的无效复刻。日志条目会记录报错：“ ERROR Local blockchain failed to pass a checkpoint, rolling back! （ERROR 本地区块链无法通过检查节点，回滚！）”最终无效复刻会被搁置，替代的（“修复的”）分叉文件将变大，节点会跟随替代的分叉。    默认情况下，检查节点仅通过生成以下日志条目来通知有关报错：错误警告：本地区块链无法通过 MoneroPulse 检查节点，你可能在分叉上。这时你需要从头开始同步，或者下载新的区块链引导程序，或者使用 --enforce-dns-checkpointing 命令选项启用检查节点强制执行。    参考：[源代码](https://github.com/monero-project/monero/blob/22a6591a70151840381e327f1b41dc27cbdb2ee6/src/cryptonote_core/blockchain.cpp#L3614)。   | 
| --disable-dns-checkpoints   | 核心开发人员设置的MoneroPulse检查节点将被放弃。尽管如此，检查节点程序仍然被保留。   | 

### P2P 网络
下列选项定义你的节点如何参与门罗币p2p网络，用于节点与节点间的通信。下列选项不影响[钱包到节点](https://monerodocs.org/interacting/monerod-reference/#node-rpc-api)的接口。
“节点”和“对等（peer）”的词汇可以互换使用。

| 选项   | 描述   | 
|:----|:----|
| --p2p-bind-ip   | 绑定到p2p网络协议的网络接口。默认值0.0.0.0 会绑定到所有网络接口。通常情况下你需要用这个。    如果要约束绑定，则必须更改此设置，例如通过Tor 来配置通过 Tor 的连接：  DNS_PUBLIC=tcp://1.1.1.1 TORSOCKS_ALLOW_INBOUND=1 torsocks ./monerod --p2p-bind-ip 127.0.0.1 --no-igd --hide-my-port   | 
| --p2p-bind-port   | 用于侦听p2p网络连接的TCP端口。主网默认为18080，testnet默认为28080，stagenet默认为38080。通常你不用改变它。如果你在计算机上运行多个节点来模拟私有Monero p2p网络（可能使用私有Testnet），这个选项会很有帮助。例：  ./monerod --p2p-bind-port=48080   | 
| --p2p-external-port   | 用于侦听路由器上p2p网络连接的TCP端口。如果你在NAT后面仍然希望接受传入连接，就会用到这个了。你必须将其设置为路由器上的相关端口。这是为了让 monerod 接收网络上的广播。默认值为0。   | 
| --hide-my-port   | monerod 仍将打开并在p2p端口上侦听。不过它不会说自己是备选节点。从技术上讲，它将在与p2p协议握手的响应中返回端口0（get_local_node_data函数中的node_data.my_port = 0）。实际上，你连接的节点不会将IP扩展到其他节点。总而言之，它并不是真正隐藏，更像是“不要广播出去”。   | 
| --seed-node   | Connect to a node to retrieve other nodes' addresses, and disconnect. If not specified, monerod will use hardcoded seed nodes on the first run, and peers cached on disk on subsequent runs.  连接到一个节点以检索其他节点的地址，然后断开连接。如果未指定，monerod将在第一次运行时使用硬编码的种子节点，并在后续运行时使用节点在硬盘上的缓存。   | 
| --add-peer   | 手动将节点添加到本地对等（peer）列表。   | 
| --add-priority-node   | 指定要连接的节点列表，然后尝试保持连接。  要添加多个节点，请多次使用该选项。 例：  ./monerod --add-priority-node=178.128.192.138:18081 --add-priority-node=144.76.202.167:18081   | 
| --add-exclusive-node   | 指定仅连接的节点列表。 如果给出此选项，则忽略选项--add-priority-node和--seed-node。  要添加多个节点，请多次使用该选项。 例：  ./monerod --add-exclusive-node=178.128.192.138:18081 --add-exclusive-node=144.76.202.167:18081   | 
| --out-peers   | 设置到其他节点的最大传出连接数，默认值为8。-1表示代码默认值。   | 
| --in-peers   | 设置最大传入连接数（主动连接到你的节点），默认无限制。 -1表示代码默认值。   | 
| --limit-rate-up   | 设置传出数据传输限制[kB / s]。 默认为2048 kB / s。 值-1表示代码默认值。   | 
| --limit-rate-down   | 设置传入数据传输限制[kB / s]，默认为8192 kB / s。 值-1表示代码默认值。   | 
| --limit-rate   | 为传入和传出数据设置相同的传输限制值。 默认情况下（-1）将使用单独的上/下默认限制。 最好使用--limit-up-up和--limit-rate-down来避免混淆。   | 
| --offline   | 不再侦听其他节点，也不连接到任何节点。适用于使用本地档案（archival）   区块链。   | 
| --allow-local-ip   | Allow adding local IP to peer list. Useful mostly for debug purposes  when you may want to have multiple nodes on a single machine.  允许将本地IP添加到对等节点列表。适用于当你希望在一台计算机上拥有多个节点时，进行调试。   | 

### 节点 RPC API
提供了强大的 API。 它有三个用途:
* 提供网络数据(统计、区块、 交易...)
* 提供本地节点信息(对等节点列表，挖矿的算力...)
* 为钱包提供接口(发送交易...)

这个 API 通常被称为“ RPC” ，因为它主要基于 json / RPC 标准。
以下选项定义 API 的行为方式。

| 选项   | 描述   | 
|:----|:----|
| --rpc-bind-ip   | 要侦听的IP。默认情况下为127.0.0.1，因为API为节点提供了完整的管理功能。将其设置为0.0.0.0可以侦听所有接口，但会仅与* -restricted- *选项和--confirm-external-bind 之一相连接。   | 
| --rpc-bind-port   | 要侦听的TCP端口。默认为18081（主网），28081（测试网），38081（stagenet）。   | 
| --rpc-restricted-bind-port   | 使用有限版本的API侦听TCP端口。有限的API可以公开，以创建开放节点。 同时，你可以防护完整的API端口，并继续使用本地查询和管理。   | 
| --confirm-external-bind   | 确认是你将 --rpc-bind-ip 设置为非本地主机IP，而且你了解其后果。   | 
| --restricted-rpc   | 限制API的仅查看命令，不返回隐私敏感数据。注意这对于 -rpc-restricted-bind-port没有意义，因为你最终会得到两个受限制的API。   | 
| --rpc-login   | 指定连接到API所需的用户名[：密码]。 实际使用似乎有限，因为API通信是通过HTTP的纯文本。   | 
| --rpc-access-control-origins   | 指定逗号分隔的原始列表以允许跨源资源共享。如果你想通过JavaScript直接从Web浏览器使用 monerod API（例如在纯粹的Web appp场景中），这非常有用。使用此选项，monerod将为其响应添加正确的HTTP CORS标头。如果使用此选项，还需要设置--rpc-login。 通常情况下，API由后端应用程序使用，并且此选项不是必需的。   | 

### 接收门罗币 
这些是 monerod 提供的网络通知和钱包通知，比如 monero-wallet-rpc 提供的 -- tx-notify。

| 选项   | 描述   | 
|:----|:----|
| --block-notify <arg>   | Run a program for each new block. The <arg> must be a **full path**. If the <arg> contains %s it will be replaced by the block hash. Example:   为每个新区块运行程序。参数（<arg>）必须是完整路径。如果参数中包含％s，它将被区块的哈希替换。 例：  ./monerod --block-notify="/usr/bin/echo %s"  区块通知有助于即时反应。但是，你应该一直假定会错过一些区块通知，你应该独立轮询API来查漏补缺。  注意区块链的重组。区块通知可以恢复到和过去相同的高度。小型重组是正常的，每天都会发生。   | 
| --block-rate-notify <arg>   | 当最近收到的块数明显偏离预期时，运行程序。 参数（<arg>）必须是完整路径。参数可以包含要插入的％t，％b，％e字符中的任何一个：  ％t：观察窗口中的分钟数  ％b：在该窗口中观察到的块数  ％e：预期在该窗口理想的块数中    该选项将让你知道网络的算力是否下降了很多。这表明可能大部分网络矿工正在开采一条私链，后来被释放到网络中。请注意，即时此事件触发，也并不一定说明事件发生了，只是说有这种可能。窗口中时间越久（％t参数），且实际和预期的块数之间的距离越大的话，预示可能的双花攻击的准备就越多。    建议：除非你进行重要的门罗币交易或其他操作，否则不要动它。它很难校准，容易被误解。如果这是一次真正的攻击，攻击针对的将高流动性实体而非小型商家。   | 
| --reorg-notify <arg>   | 区块链的重组发生时，运行程序（即，从区块链的顶部删除至少一个块）。 参数（<arg>）必须是完整路径。 参数可以包含要插入的％s，％h，％n字符中的任何一个：    ％s：分叉发生的高度    ％h：新区块链的高度    ％d：从旧链中丢弃的块数    ％n：要添加的块数    该选项会告诉你何时从链中删除块以由其他块替换。当发生51％的攻击时，会发生这种情况，但在正常情况下也会发生小的重组。 ％d参数将被设置为从旧链中丢弃的块数（因此，如果这高于你等待对付款进行操作的块数，则该付款可能已被取消）。％n参数将被设置为新链中的块数（因此，如果这高于你等待接收付款的块数，则第一个块中的任何收款都将在你的平台上自动执行）。    建议：除非你进行重要的门罗币交易或其他操作，否则不要动它。在运送贵重物品之前，通过要求至少10次确认来简单说明重组。   | 

### 性能
这些都是高级选项，允许你优化 monerod 节点的性能，有时以牺牲可靠性为代价。

| 选项   | 描述   | 
|:----|:----|
| --db-sync-mode   | 使用以下格式指定同步选项：  [safe\|fast\|fastest]:[sync\|async]:[<nblocks_per_sync>[blocks]\|<nbytes_per_sync>[bytes]]    默认值为fast：async：250000000bytes。    fast：async：*可以在系统崩溃时破坏区块链数据库。如果只是monerod崩溃，它不应该崩溃。如果你担心系统崩溃，请使用safe：sync。   | 
| --max-concurrency   | 用于并行作业的最大线程数，默认值0。使用CPU线程数。   | 
| --prep-blocks-threads   | 计算组中的区块哈希（PoW）时要使用的最大线程数。默认为4。如果在同步时不希望monerod 占用计算机，请减小此值。   | 
| --fast-block-sync   | 通过使用嵌入的“已知”区块哈希来同步大部分区块。传递1为打开，0为关闭，默认为1。通常，整个节点必须计算每个区块的哈希来验证矿工的工作证明。由于Monero中使用的CryptoNight PoW非常昂贵（即使是用于验证），因此monerod会为旧块提供跳过这些计算的功能。换句话说，它是一种机制，可以信任旧块PoW有效性的monerod二进制文件，加快同步速率。   | 
| --block-sync-size   | 在同步区块链期间，在一个批次中处理多少个区块。默认情况下，较新历史记录为20个块，较旧历史记录为100个块（“pre v4”）。默认操作由值0表示。直观地说，你拥有的资源越多，你可能想要尝试的批量越大。 例：  ./monerod --block-sync-size=500   | 
| --bootstrap-daemon-address   | 主机：“引导程序”远程打开节点的端口，当此节点仍未完全同步时，连接的钱包也可以使用该节点。 例：  ./monerod --bootstrap-daemon-address = opennode.xmr-tw.org:18089。   该节点将选定的RPC调用转发到引导节点。钱包将自动透明地处理这个问题。显然，这种引导阶段具有类似于直接使用远程节点的隐私含义。   | 
| --bootstrap-daemon-login   | 指定[用户名：bootstrap 守护进程登录的密码（如果需要）]。这考虑了钱包使用的RPC接口。通常，开放节点不需要任何凭据。   | 

### 挖矿 
下面的选项使用标准软件堆栈 monerod 配置单独的 CPU 挖矿。在以下方面会很有用：
* 生成你的 stagenet 或 testnet 代币
* 测试和学习
* 如果你的 CPU 资源充裕

请注意，真正的挖矿发生在矿池和高端的 GPU，而不是 CPU中。

| 选项   | 描述   | 
|:----|:----|
| --start-mining   | 指定要挖矿的钱包地址。这里必须是[标准地址](https://monerodocs.org/public-address/standard-address)！ 既不是subaddres ，也不是集成地址。   | 
| --mining-threads   | 指定挖矿线程数。默认情况下，只使用一个线程。为获得最佳结果，请将其设置为硬件的处理器数。   | 
| --extra-messages-file   | 指定要包含在coinbase 交易中的额外消息的文件。   | 
| --bg-mining-enable   | 实现“静悄悄”地挖矿。 在此模式下，挖矿只使用一小部分系统资源，从而不会明显降低计算机速度。这是为了鼓励人们挖矿来增强去中心化。据说只用CPU挖矿找到区块的机会越来越小，甚至在这个“静悄悄”的模式中也越来越小。 你可以使用下面的--bg- *选项调整 安静/功率 之间的度。   | 
| --bg-mining-ignore-battery   | If true, assumes plugged in when unable to query system power status.  如果为真（true），则假定在无法查询系统电源状态时插入。   | 
| --bg-mining-min-idle-interval   | 指定用于确定空闲状态的最小回溯间隔（单位为秒）。   | 
| --bg-mining-idle-threshold   | 指定回顾间隔的最小平均空闲百分比。   | 
| --bg-mining-miner-target   | 指定矿工使用的最大cpu百分比。   | 

### 测试门罗币
这些选项对 门罗币项目的开发人员和测试人员非常有用。普通用户可以略过这些。

| 选项   | 描述   | 
|:----|:----|
| --test-drop-download   | 在网络测试中：下载时，丢弃所有块，而不必检查/保存它们（非常快）。   | 
| --test-drop-download-height   | 和 --test-drop-download类似，但只在达到一定高度后丢弃。 默认为0。   | 
| --regtest   | 以回归测试模式运行。   | 
| --fixed-difficulty   | 修复了用于测试的难度。 默认为0。   | 
| --test-dbg-lock-sleep   | 睡眠时间以毫秒为单位，默认为0（关闭），用于在锁定互斥锁之前/之后进行调试。 值100到1000适用于测试。   | 
| --save-graph   | 保存dr Monero的数据。   | 

### 旧版
这些选项应该不再是必要的。为了向后兼容，它们仍然存在于 monerod 中。

| 选项   | 描述   | 
|:----|:----|
| --fluffy-blocks   | 致密区块中继。默认。致密区块只是一个标头和一个事务ID列表。   | 
| --no-fluffy-blocks   | 经典完整区块中继。经典区块包含所有交易。   | 
| --show-time-stats   | 官方文档说“在处理区块/ 交易和磁盘同步时显示时间 - 统计信息”，但在通常的区块链同步期间似乎没有产生任何输出。   | 
| --zmq-rpc-bind-ip   | 要侦听的ZMQ RPC服务器的IP。默认为127.0.0.1。这还没有被广泛使用，因为ZMQ接口目前没有提供优于传统JSON-RPC接口的有意义的优势。不幸的是，目前无法禁用ZMQ服务器。   | 
| --zmq-rpc-bind-port   | ZMQ RPC服务器侦听的端口。 默认情况下18082用于主网，38082用于stagenet，28082用于testnet。   | 
| --db-type   | 指定数据库类型。 默认且仅可用：lmdb。   | 

## 命令
命令可以访问守护进程提供的特定服务。对正在运行的守护进程执行命令。它们的名称遵循command_name 模式。

下面的小组只是为了让参考更易理解。守护进程本身不以任何方式对命令进行分组。

参见运行的[示例用法](https://monerodocs.org/interacting/monerod-reference/#running)。 你还可以直接在运行的 monerod 控制台中键入命令(如果不是分离的话)。
### 帮助，版本，状态
| 选项   | 描述   | 
|:----|:----|
| help [<command>]   | 显示指定命令的帮助。   | 
| version   | 显示版本信息。 示例输出：  Monero 'Boron Butterfly' (v0.14.0.0-release)   | 
| status   | 显示状态。 示例输出：  Height: 186754/186754 (100.0%)  on stagenet, not mining, net hash 317 H/s, v9, up to date, 8(out)+0(in)  connections, uptime 0d 3h 48m 47s   | 

### P2P网络

| 选项   | 描述   | 
|:----|:----|
| print_pl   | 显示完整的对等节点列表。   | 
| print_pl_stats   | 显示完整的对等节点列表统计信息（白色VS灰色的对等节点）。白色对等节点为在线且可以访问。灰色对等节点为离线，但你的monerod会记住它们过去的会话。   | 
| print_cn   | 显示具有连接主动性（传入/传出）和其他统计信息的已连接对等节点。   | 
| ban <IP> [<seconds>]   | 禁止给定的IP达到设定的时间（单位为秒）。默认禁令是24小时。 例：  ./monerod ban 187.63.135.161.   | 
| unban <IP>   | 取消对给定<IP>的限制。   | 
| bans   | 显示当前禁止的IP。 示例输出：  187.63.135.161 banned for 86397 seconds.   | 
| in_peers <max_number>   | 设置来自其他对等节点的传入连接。   | 
| out_peers <max_number>   | 设置与其他对等节点的传出连接。   | 
| limit [<kB/s>]   | 获取或设置下载和上传限制。   | 
| limit_down [<kB/s>]   | 获取或设置下载限制。   | 
| limit_up [<kB/s>]   | 获取或设置上传限制。   | 

### 交易池
| 选项   | 描述 | 
|:----|:----|
| flush_txpool [<txid>]   | 刷新交易池中的指定交易，如果指定交易不存在，则刷新整个交易池。   | 
| print_pool   | 使用详细格式打印交易池。   | 
| print_pool_sh   | 使用短格式打印交易池。   | 
| print_pool_stats   | 打印交易池的统计信息（事务数，内存大小，费用，双花攻击尝试等）。   | 

### 交易 

| 选项   | 描述   | 
|:----|:----|
| print_coinbase_tx_sum <start_height> [<block_count>]   | 显示指定范围内所有发送的代币和付费的总和。 例：  ./monerod print_coinbase_tx_sum 0 1000000000000   | 
| print_tx <transaction_hash> [+hex] [+json]   | 将指定的交易显示为JSON和/或HEX。   | 
| relay_tx <txid>   | Force relaying the transaction. Useful if you want to rebroadcast  the transaction for any reason or if transaction was previously created  with "do_not_relay":true.强制中继交易。如果你因任何原因要重新将交易广播出去，或者之前使用[“do_not_relay”:true ]创建交易，则非常有用。   | 

### 区块链

| 选项   | 描述   | 
|:----|:----|
| print_height   | 显示本地区块高度。   | 
| sync_info   | 显示区块链同步进度和连接的对等节点以及下载/上传统计信息。   | 
| print_bc <begin_height> [<end_height>]   | 显示范围<begin_height> .. <end_height>中的块。该信息将包括区块ID，高度，时间戳，版本，大小，重量（weight），非币基交易（non-coinbase transactions）的数量，难度，随机数和奖励。   | 
| print_block <block_hash> \| <block_height>   | 显示指定区块的详细数据。   | 
| hard_fork_info   | 显示当前的共识版本和未来的硬分叉区块高度（如果有）。   | 
| is_key_image_spent <key_image>   | 检查是否花费了指定的[钥匙镜像](https://monerodocs.org/cryptography/asymmetric/key-image/)。钥匙镜像是一个哈希值。   | 

### 管理守护进程

| 选项   | 描述   | 
|:----|:----|
| exit, stop_daemon   | 让守护进程正常退出。 exit和stop_daemon是相同的（一个是另一个的别名）。   | 
| set_log <level>\|<{+,-,}categories>   | 设置当前日志级别/类别，其中<level>是数字0-4。   | 
| print_status   | 显示守护程序是否正在运行。   | 
| update (check\|download)   | 检查更新是否可用，并可选择下载。哈希值基于SHA-256。在linux上使用sha256sum来验证。 示例输出：  Update  available: v0.13.0.4:  [https://downloads.getmonero.org/cli/monero-linux-x64-v0.13.0.4.tar.bz2,](https://downloads.getmonero.org/cli/monero-linux-x64-v0.13.0.4.tar.bz2,)  hash 693e1a0210201f65138ace679d1ab1928aca06bb6e679c20d8b4d2d8717e50d6Update downloaded to: /opt/monero-v0.13.0.2/monero-linux-x64-v0.13.0.4.tar.bz2   | 

### 挖矿
| 选项   | 描述   | 
|:----|:----|
| show_hr   | 要求monerod daemon 停止打印当前算力。 只有当monerod 挖矿时才有用。   | 
| hide_hr   | 要求monerod daemon 打印当前的算力。只有当monerod 挖矿时才有用。   | 
| start_mining <addr> [<threads>] [do_background_mining] [ignore_battery]   | 要求monerod daemon开始挖矿。区块奖励将转到<addr>。   | 
| stop_mining   | 要求monerod daemon 停止挖矿。   | 

### 测试门罗币
| 选项   | 描述   | 
|:----|:----|
| start_save_graph   | 开始为Dr Monero保存数据。   | 
| stop_save_graph   | 停止为Dr Monero保存数据。   | 

### 旧版
| 选项   | 描述   | 
|:----|:----|
| save   | 将区块链数据刷新到磁盘。通常不再需要，因为monerod会在退出时自动保存区块链。   | 
| output_histogram [@<amount>] <min_count> [<max_count>]   | Show number of outputs for each amount denomination. This was only  relevant in the pre-RingCT era. The old wallet used this to determine  which outputs can be used for the requested mixin. With RingCT  denominations are irrelevant as amounts are hidden. More info in [these SA answers](https://monero.stackexchange.com/search?q=%22output_histogram%22).显示不同面额代币的输出数量。这只是在环签交易（RingCT）诞生之前的时代有用。旧钱包用它来确定哪些输出可用于所请求的mixin。 在环签交易中，面额与隐藏金额无关。 这些在[SA答案](https://monero.stackexchange.com/search?q=%22output_histogram%22)中有更多信息。   | 


# 命令行门罗币 monero-wallet-cli 
 
>**注意**
>一个友好的 Monero CLI 钱包能让你体验好到爆。 它是门罗币最可靠和最完整的钱包。可使用 stagenet 进行学习。

 
## 概述
### 命令行门罗币monero-wallet-cli 
Monero 的“正式”命令行钱包。可用于 Linux，macOS 和 Windows。
钱包使用你的私人密钥，以了解你的总收支，交易历史，并辅助创建交易。
不过钱包并不存储区块链，也不直接参与 p2p 网络。
CLI 钱包是门罗币最可靠的，功能最齐全的钱包。

### 依赖全节点
钱包连接到[全节点](https://monerodocs.org/interacting/monerod-reference)后，为了输出你的交易会扫描区块链，并将你的交易发送到网络。
全节点可以是本地(同一台计算机)或远程的节点。
通常，你要在与钱包相同的计算机上(或在你的家庭网络中)运行全节点。
通过 HTTP 连接并使用相应的[ API](https://www.getmonero.org/resources/developer-guides/wallet-rpc.html)。
任何离开钱包的交易都会被门罗币的隐私功能隐藏。这意味着即使连接到远程节点，纯文本 HTTP 通信本身也不是问题。
但是，连接到远程节点还有其他细微的权衡，这是另一篇文章的主题。
## 语法

```
./monero-wallet-cli [options] [command]
```
例子:
```
./monero-wallet-cli --stagenet
```
## 运行
进入门罗币的目录。
运行完整的节点并等待它与网络同步(可能需要几天时间) :
```
./monerod --stagenet
```

如果要在一个单独的终端窗口中运行:
```
./monero-wallet-cli --stagenet --generate-new-wallet MoneroExampleStagenetWallet
```

## 选项
### 帮助和版本

| 选项   | 描述   | 
|:----|:----|
| --help   | 获得可用选项。   | 
| --version   | 将monero-wallet-cli版本显示到标准输出。 例：  Monero 'Boron Butterfly' (v0.14.0.0-release)   | 

### 选择网络
| 选项   | 描述   | 
|:----|:----|
| (missing)   | 默认情况下，钱包设定为[mainnet](https://monerodocs.org/infrastructure/networks)。   | 
| --stagenet   | 在stagenet上运行。 记得用--stagenet运行你的守护进程。   | 
| --testnet   | 在testnet上运行。 记得用--testnet运行你的守护进程。   | 

### 日志 
| 选项   | 描述   | 
|:----|:----|
| --log-file <arg>   | 日志文件的完整路径。   | 
| --log-level <arg>   | 0-4：0表示最小日志记录，4表示完全跟踪，默认为0。这些是常规预设，不需要调到最高级别。 例如，即使调到0档，你也可能会看到一些最重要的INFO条目。   | 
| --max-log-file-size <arg>   | 日志文件的软性限制（以字节为单位，大小默认为104850000bytes，刚好小于100MB)。 一旦日志文件超过该限制，monerod将使用UTC时间戳，以YYYY-MM-DD-HH-MM-SS的名字创建下一个日志文件。    在开发部署中，你可能更喜欢使用成型的解决方案，例如 logrotate。在这种情况下，请设置--max-log-file-size = 0以防止 monerod 管理日志文件。   | 
| --max-log-files <arg>   | 限制日志文件的数量（默认为50）。最旧的日志文件会被删除。在开发部署中，你可能更喜欢使用成型的解决方案，例如logrotate。   | 

### 全节点连接
钱包依赖所有非本地操作的全节点。下面的选项定义了如何连接到 monerod:

| 选项   | 描述   | 
|:----|:----|
| --daemon-address <arg>   | 以[主机:端口]使用monerod实例。例：  ./monero-wallet-cli --daemon-address monero-stagenet.exan.tech:38081 --stagenet   | 
| --daemon-host <arg>   | 在输入主机而不是localhost上使用monerod实例。   | 
| --daemon-port <arg>   | 在输入端口而不是18081上使用monerod实例。   | 
| --daemon-login <arg>   | 为monerod RPC API指定用户名[：密码]。它基于HTTP基础验证（Basic Auth）。请注意，默认情况下连接无加密。 只有建立安全连接（可能通过Tor，或SSH隧道或反向代理w / TLS），身份验证才有意义。   | 
| --trusted-daemon   | 启用依赖 monerod 可信实例的命令和行为。localhost连接的默认值。在这种情况下，信任会保护你的隐私。这个选项只在你控制monerod 时使用。    可信守护进程（Trusted daemon ）允许使用 rescan_spent，start_mining，import_key_images等命令，也会允许在发送交易时对可能引发攻击的瞬态问题不再警告的行为。   | 
| --untrusted-daemon   | 禁用依赖monerod 可信实例的命令和行为。非本地主机连接的默认值（ Default for a non-localhost connections）。 有关更多详细信息，请参阅 --trusted-daemon。   | 
| --do-not-relay   | 新创建的交易不会被中继到Monero网络。相反，它将以原始十六进制格式转储到文件中。 在你想通过[https://xmrchain.net/rawtx](https://xmrchain.net/rawtx)等网关发送交易时，这会非常有用。与Monero钱包相比，用Tor会更方便。   | 
| --allow-mismatched-daemon-version   | 允许与使用不同RPC版本的 monerod 进行通信。   | 

### 创造新钱包

| 选项   | 描述   | 
|:----|:----|
| --generate-new-wallet <arg>   | 创建一个新的门罗币钱包，并将其保存到<arg>文件。系统会要求你输入密码。密码用于加密钱包文件，但它与你的主要支出密钥（spend key）或助记词无关。可以用密码管理器生成一个非常强大的密码（256位比特）。 例：    ./monero-wallet-cli --stagenet --generate-new-wallet $HOME/.bitmonero/stagenet/wallets/MoneroExampleStagenetWallet   | 
| --kdf-rounds <arg>   | 关注对钱包文件的加密。钱包文件使用ChaCha流密码加密。加密密钥是根据用户提供的密码通过CryptoNight哈希算法派生的。此选项确定的是应用CryptoNight哈希计算的次数。默认为1轮哈希计算。    请注意，这与花费密钥（ spend key ）的生成无关。    哈希计算的轮数越多，打开钱包或发送交易等待的时间越长。但是攻击者暴力破解你钱包密码的时间也会越久。    注意：你必须记住并在每次访问钱包时提供相同的kdf轮次！    建议：不要更改默认值。最好使用密码管理器生成一个非常强大的钱包密码（256位比特）。   | 

### 打开已有钱包 
| 选项   | 描述   | 
|:----|:----|
| --wallet-file <arg>   | 打开已有钱包。 例：  ./monero-wallet-cli --stagenet --wallet-file $HOME/.bitmonero/stagenet/wallets/MoneroExampleStagenetWallet 这仅适用于使用monero-wallet-cli，monero-wallet-gui或monero-wallet-rpc 工具生成的钱包文件。如果你有其他类型的钱包，请参阅导入选项。   | 
| --password <arg>   | 钱包密码在这里是作为参数，而不是交互。根据需要记得escape/quote。    不推荐使用，因为密码将保留在命令历史记录中，并且也会在进程表中显示。 对于自动化首选 --password-file。    该选项还可与--generate-new-wallet 结合使用。   | 
| --password-file <arg>   | 钱包密码在这里是作为文件，而不是交互。读取密码文件时将丢弃结尾的“\ n”字符（Trailing \n are discarded when reading the password file. ）。    如果你自动化钱包的访问，这种方式会比--password更好（Prefer this over --password  if you automate wallet access.）。确保密码文件与钱包文件有意义地分开，否则这种方式也就失去了安全的效果了。    该选项还可与--generate-new-wallet 结合使用。   | 

### 恢复钱包
| 选项   | 描述   | 
|:----|:----|
| --generate-from-device <arg>   | 恢复/生成一个特殊钱包，以便与Ledger或Trezor等硬件设备一起使用，并将其保存到<arg>文件中。 例：  ./monero-wallet-cli --stagenet --generate-from-device MoneroExampleDeviceWallet --subaddress-lookahead 5:20 这个操作只需要一次。下次你只需[打开钱包](https://monerodocs.org/interacting/monero-wallet-cli-reference/#open-existing-wallet)。    默认情况下，该命令需要连接 Ledger 硬件。使用Trezor 硬件需添加--hw-device Trezor（预计2019年5月）。    使用默认设置最多需要25分钟。 这是因为硬件设备预生成子地址的速度很慢。为了减轻使用率--subaddress-lookahead 5:20（To mitigate use low --subaddress-lookahead 5:20）。    本地钱包不会有私人支出密钥（private spend key），也无法单独消费。 它被用作低功耗硬件设备的用户界面和桥接器。使用**私密花费密钥**（private spend key）的交易签名始终在硬件设备上进行。    请参阅硬件钱包设置的[完整指南](https://www.reddit.com/r/Monero/comments/8op6cp/ledger_cli_guides_requires_cli_v01220/)。   | 
| --generate-from-view-key <arg>   | 恢复仅查看版本的钱包以跟踪传入的交易，并将其保存到<arg>文件。钱包是基于**私密查看密钥（secret view key）**和标准地址创建的。查看密钥会被以十六进制的形式被复制。   | 
| --generate-from-spend-key <arg>   | 从**私密花费密钥**（**secret spend key**）恢复钱包并将其保存到<arg>文件。 花费密钥会被以十六进制的形式被复制(The secret spend key is meant to be pasted as hexadecimal.)。   | 
| --restore-deterministic-wallet   | 根据**助记词**（ **secret mnemonic seed**）恢复钱包。使用此功能从25字备份中恢复。    系统将要求你输入密码以加密钱包文件（恢复后）。 请注意，这不是助记词的密码。门罗币官方钱包产生的助记词是未加掩盖的。   | 
| --restore-height <arg>   | 仅扫描指定区块高度之后的交易，默认值为0。提高该值可使钱包的恢复速度更快。最佳值应与你最初创建钱包的日期相匹配（但不能比那晚）。区块高度和日期/时间之间的映射在区块浏览器（如-[https://xmrchain.net](https://xmrchain.net)）上可用。 例如，如果你在2019年创建了钱包，请使用1730000（if you created the wallet in 2019+ use 1730000）。   | 

### Multisig 钱包
| 选项   | 描述   | 
|:----|:----|
| --generate-from-multisig-keys <arg>   | 基于多重签名密钥(multisig keys)创建标准钱包。 如果将所有多重签名密钥组合回标准钱包（当你不再需要multisig时），这会非常有用。之后钱包将控制资金。它仅支持所有密钥齐全的情况，即便多重签名方案允许更少的密钥——仅N / N，而不是N / M。   | 
| --restore-multisig-wallet   | 从之前使用种子交互命令（ seed  interactive command）导出的秘密种子（**secret seed**）恢复多重签名钱包。这只会恢复你钱包的一部分。 其他多重签名参与者仍然需要签署交易。   | 

### 配置文件
| 选项   | 描述   | 
|:----|:----|
| --config-file <arg>   | [配置文件](https://monerodocs.org/interacting/monero-config-file)的完整路径。请注意，这应该是使用monerod 之外的单独配置（ this should be a separate config than monerod uses ），因为这些工具接受不同的选项集。   | 

### 性能
| 选项   | 描述   | 
|:----|:----|
| --subaddress-lookahead <arg>   | 接收m:n，默认为50:200。 第一个值是帐户数，第二个值是每个帐户的子地址（subaddresses）数。    钱包检查付款的范围只限于最近收到的付款到第n个子地址。 如果你为一行中的n个客户生成了唯一的子地址，但他们都没有付款，就会发生这种情况。    另一方面，你设置为向前检查的子地址越多，创建钱包所需的时间就越长，因为它们必须预先计算。 除硬件钱包外，这通常不是问题。 在Ledger上，默认值50：200可能需要20多分钟（创建钱包时一次）！   | 
| --max-concurrency <arg>   | 设置并行作业的最大线程数，默认值为0（使用CPU线程数）。   | 

### 国际化
| 选项   | 描述   | 
|:----|:----|
| --mnemonic-language <arg>   | 助记词的语言。包括英语，english_old，世界语，法语，德语，意大利语，日语，lojban，葡萄牙语，俄语，西班牙语，中文。    坚持默认英语会比较好，这是迄今为止最受欢迎且经过充分测试的。 它还避免了潜在的非ASCII字符陷阱或错误。   | 
| --use-english-language-names   | 如果你的显示屏卡住，请使用 ^C 退出，然后使用--use-english-language-names再次运行。当Monero提示在其本机字母表中显示语言名称的语言时，可能会发生这种情况。   | 

### 旧版
这些选项要么是遗留的，要么很少有用。

| 选项   | 描述   | 
|:----|:----|
| --non-deterministic   | 生成旧版的非确定性钱包。 **查看密钥**（view key）不会由花费密钥派生。你还必须备份 .keys文件。要恢复非确定性钱包（标准地址），请使用--generate-from-keys。要完全恢复，你将需要.keys文件。   | 
| --generate-from-keys <arg>   | 通过提供花费密钥和查看密钥以及标准地址来恢复旧版的非确定性钱包。   | 
| --shared-ringdb-dir <arg>   | 设置共享环数据库路径。 [不值再提](https://www.reddit.com/r/Monero/comments/9rtnpx/are_there_any_updated_blackball_databases/)了。   | 
| --create-address-file   | 并无效果。无论此选项如何，都会创建* .address.txt文件。   | 
| --electrum-seed <arg>   | 提供助记词作为--restore-deterministic-wallet的命令行选项，而非交互（instead of interactively）。 不推荐这样做，因为助记词将被保存在命令的历史记录中，并且也可以在进程列表中看到（This is not recommended b/c the seed will be  saved in your command history and also visible in the process list）。   | 
| --generate-from-json <arg>   | 你可以运行monero-wallet-rpc来使用此选项。在monero-wallet-cli中很少用到。   | 
| --tx-notify <arg>   | 你可以运行monero-wallet-rpc来使用此选项。 在monero-wallet-cli中很少用到。   | 

## 默认值Defaults
创建钱包文件并在工作目录中查找。一般很少需要这么做。 使用 -- wallet-file 和类似的选项可以对其操作。
日志文件在与 monero-wallet-cli 二进制文件相同的目录中创建。 使用 -- log-file 可以指定位置。
## 命令
命令在 monero-wallet-cli 提示符（prompt）中可以交互地使用。
你还可以通过提供一个命令行参数来运行一次性命令。但很少用到。自动化操作会更喜欢 monero-wallet-rpc（For automation prefer monero-wallet-rpc.）。
命令行钱包内置了针对单个命令的帮助——这里我们就不重复了。相反，我们将重点放在分组命令上，这样你就可以快速找到你要查找的内容。 使用 help command_name 可以了解更多信息。
### 帮助和版本
```
help #列出所有命令
help command_name #显示单个命令的帮助
version #显示 monero-wallet-cli 二进制文件的版本
```

### 开支 
```
account #总收支; 列出有关结余的帐户

balance detail #在最近的账户中，列出带有余额的地址
```

### 管理客户
```
account

account new

account switch

account label
```
### 管理地址
```
address all

address new

address label
```
### 网络状态
```
status #show if synced up to the blockchain height显示是否同步到了区块链的高度

fee - show current fee-per-byte and full node's mempool (the backlog of transactions depending on the priority)显示当前的每字节收费和完整的节点记忆池(根据优先级确定的交易积压)

wallet_info #显示钱包文件路径，标准地址，类型和网络
```

### 钱包信息Secret mnemonic seed 
```
seed #显示生成的助记词
encrypted_seed #创建助记词，你需要单独记住这些密码短语，没有这些短语就无法恢复钱包了
```

### 密钥
```
spendkey  #显示私密花费密钥和公开花费密钥

viewkey  #显示私密查看密钥和公开查看密钥
```
### 证明
```
get_reserve_proof -> check_reserve_proof #证明余额

get_spend_proof -> check_spend_proof #证明你支付了

sign <file> -> verify <filename> <address> <signature> #证明地址的所有权; 允许验证由特定 Monero 地址的所有者签名的文件

get_tx_proof -> check_tx_proof
```

### 多重签名
创建
```
prepare_multisig

make_multisig

finalize_multisig
```
更新

```
export_multisig_info

import_multisig_info
```
其他
```
submit_multisig

exchange_multisig_keys

export_raw_multisig_tx
```
### 交易私钥
它们允许查看（learn）和验证交易的私钥。 这对创建支付证明很有用，不过已经被 get spend proof 取代了。
```
get_tx_key <txid>

check_tx_key <txid> <txkey> <address>

set_tx_key <txid> <tx_key>
```
### 高级
```
unspent_outputs #显示未支出的列表和直方图(总余额中不可分割的部分indivisible pieces of your total balance)

export_key_images <file> -> import_key_images <file> #用来通知只可查看钱包（view-only wallet）发送出去的交易，以此计算真正的收支；通常只可查看钱包只了解收到的交易，而不是发送出去的

export_outputs <file>  #将这个钱包所拥有的一组输出导出到一个文件
```


### 挖矿 
```
start_mining

stop_mining
```
### 捐款
```
donate <amount> #捐款给开发团队
```

### 不必要的或者旧版的
```
address_book [(add ((<address> [pid <id>])|<integrated address>) [<description possibly with whitespaces>])|(delete <index>)]

set_description [free text note] -> get_description #管理钱包的便利描述（本地信息）

save #现在会自动发生

save_bc #现在会自动发生

bc_height #显示区块链高度(已经被status取代了)

sweep_unmixable # 只适用于非常早期版本的钱包（2016年以前），将所有不可混合的输出以ring_size为10大小的方式发送给自己
```
# GUI界面门罗币钱包
## 概述
### 桌面版GUI钱包 
这是门罗币的“官方”桌面钱包。适用于 Linux，macOS 和 Windows。
钱包使用你的私人密钥，以了解你的总收支，交易记录，并帮助你创建交易。
不过钱包并不存储区块链，也不直接参与 p2p 网络。
### 依赖全节点
如果要输出你的交易并将交易发送出去，需要将钱包连接到[全节点](https://monerodocs.org/interacting/monerod-reference)并扫描区块链。
全节点可以是本地(同一台计算机)，也可以是远程节点。一般在与运行钱包的同一台计算机上(或同一家庭网络中)运行全节点。
钱包通过 HTTP 连接全节点，并使用 [API](https://www.getmonero.org/resources/developer-guides/wallet-rpc.html)。
任何离开钱包的交易都会被 Monero 的隐私功能隐藏。这意味着即使连接到远程节点，纯文本 HTTP 通信本身也不是问题。
但是，连接到远程节点还有其他细微的权衡，这是另一篇文章的主题。
### 用户指南pdf
安装门罗币的目录内还有一份很棒的 PDF 指南，推荐你看一下！还有线上版本，地址如下:
Https://github.com/monero-ecosystem/monero-gui-guide/blob/master/monero-gui-guide.md
## 语法
```
./monero-wallet-gui [ options ]

例子:

./monero-wallet-gui --log-file /dev/null
```

## 运行

进入安装门罗币的目录。运行全节点并等待它与网络同步(可能需要几天时间) ：
```
./monerod
```
单独再开一个的终端窗口，运行钱包:
```
./monero-wallet-gui
```
## 选项
选项很少，因为所有的东西都是通过 GUI 设置的。

| 选项   | 描述   | 
|:----|:----|
| --help   | 列出可用选项   | 
| --log-file   | 显示日志文件的完整路径。 示例（注意文件权限）：  ./monerod --log-file=/var/log/monero/mainnet/monerod.log   | 

## 默认值
钱包是在 $HOME/Monero/wallets/ 中创建的。你可能想把地址改为 $HOME/.bitmonero/wallets/ ，以便把所有与 Monero 相关的文件放在一个地方。这可以在 GUI 钱包的创建向导中实现。

日志文件是直接在主目录中创建，地址为 $HOME/monero-wallet-gui.log 。 如果想改动，你可以使用 ---log-file=$HOME/.bitmonero/monero-wallet-gui.log 选项。

# 门罗币区块链导出参考 Monero-blockchain-export
 
>**注意**
>现在很少会用到区块链的导出/导入（export/import）。之前下载p2p 区块链的速度更慢。 备份区块链Raw 文件是用于加速引导节点的进程。

 
## 概述 
这个工具会将本地区块链转储为原始格式（blockchain.raw）的文件。
如果你想处理区块链有效与你的自定义工具，这会很有用，因为原始格式更易与门罗币的自定义 lmdb 数据库交互。
这个工具在你本地的区块链副本上工作。它不需要单独运行。
## 语法
```
./monero-blockchain-export [ options ]
例子:
./monero-blockchain-export --help
```
## 运行
进入你打开 Monero 的目录。
```
./monero-blockchain-export --stagenet --output-file /tmp/blockchain.raw
```
## 选项
### 帮助

| 选项   | 描述   | 
|:----|:----|
| --help   | 列出可用选项   | 

### 选择网络
| 选项   | 描述   | 
|:----|:----|
| (missing)   | 默认为主网（mainnet）。   | 
| --stagenet   | 导出 [stagenet](https://monerodocs.org/infrastructure/networks#stagenet) 区块链。   | 
| --testnet   | 导出 [testnet](https://monerodocs.org/infrastructure/networks#testnet) 区块链。   | 

### 日志
不支持指定日志文件的路径。

| 选项   | 描述   | 
|:----|:----|
| --log-level   | 0-4：0表示最小日志记录，4表示完全跟踪，默认为0。这些是常规预设，不需要调到最高级别。 例如，即使调到0档，你也可能会看到一些最重要的INFO条目。暂时更改为1可以更好地了解整个节点的运行方式。操作示例：   ./monerod --log-level=1   | 

### 输入

| 选项   | 描述   | 
|:----|:----|
| --data-dir   | 数据目录的完整路径。这是存储区块链，日志文件和p2p网络内存的地方。有关默认值和详细信息，请参阅[数据目录](https://monerodocs.org/interacting/overview/#data-directory) 。   | 
| --database, --db-type   | 指定数据库类型。 默认且仅可用：lmdb。   | 

### 输出

| 选项   | 描述   | 
|:----|:----|
| --output-file   | 输出文件路径。默认为$DATA_DIR/export/blockchain.raw。示例：   ./monero-blockchain-export --output-file=/tmp/blockchain.raw   | 
| --blocksdat   | 以 blocks.dat 格式输出。   | 
| --block-stop   | 输出到指定的区块号。默认全部输出（值为0）。   | 

## Reference 
* Https://github.com/monero-project/monero/tree/master/src/blockchain_utilities

# 门罗币区块链导入Monero-blockchain-import
>**注意**
>现在很少会用到区块链的导出/导入（export/import）。之前下载p2p 区块链的速度更慢。 备份区块链Raw 文件是用于加速引导节点的进程。

 
## 概述
该工具会引导你将 blockchain.raw 文件导入到你的全节点。
如果你出于一定原因(比如隔离测试性能)希望绕过验证单独下载，这个工具会很有用。在这种情况下，你可以登录 https://downloads.getmonero.org/blockchain.raw 下载文件。

尽管如此，理想情况下，你最好还是使用你此前导出的受信任的 blockchain.raw 文件。
请注意，导入 blockchain.raw 文件不会加快从 p2p 网络同步的进程。这是因为通常瓶颈来源于磁盘 I/O 和验证，而不是下载。
该工具适用于你的本地文件。 它不需要单独运行。
## 语法
```
./monero-blockchain-import [options]
```
例子:
```
./monero-blockchain-import --help
```
## 运行
进入你打开 Monero 的目录。
```
./monero-blockchain-import --stagenet --output-file=/tmp/blockchain.raw
```
## 选项
### Help 

| 选项   | 描述   | 
|:----|:----|
| --help   | 列出可用选项。   | 

### 选择网络
| 选项   | 描述   | 
|:----|:----|
| (missing)   | 默认为主网（mainnet）。   | 
| --stagenet   | 导入 [stagenet](https://monerodocs.org/infrastructure/networks#stagenet) 区块链。   | 
| --testnet   | 导出 [testnet](https://monerodocs.org/infrastructure/networks#testnet) 区块链。   | 

### 日志
不支持指定日志文件路径。

| 选项   | 描述   | 
|:----|:----|
| --log-level   | 0-4：0表示最小日志记录，4表示完全跟踪，默认为0。这些是常规预设，不需要调到最高级别。 例如，即使调到0档，你也可能会看到一些最重要的INFO条目。暂时更改为1可以更好地了解整个节点的运行方式。操作示例：   ./monerod --log-level=1   | 

### Input 

| 选项   | 描述   | 
|:----|:----|
| --input-file   | 区块链文件的完整路径。默认为$DATA_DIR/export/blockchain.raw。   | 
| --block-stop   | 只导入到指定区块高度，默认为全部导入（值为0）。   | 
| --pop-blocks   | 从引导区块链的顶端丢弃指定数量的区块。默认情况下不丢弃任何内容（值0）。   | 

### Output 

| 选项   | 描述   | 
|:----|:----|
| --data-dir   | 数据目录的完整路径。这是存储区块链，日志文件和p2p网络内存的地方。有关默认值和详细信息，请参阅[数据目录](https://monerodocs.org/interacting/overview/#data-directory) 。   | 
| --count-blocks   | 计算引导程序文件中的区块并退出。     | 
| --drop-hard-fork   | 是否丢弃硬分叉数据。 默认为关闭（0）。   | 
| --database   | 唯一有效的值为lmdb（默认值）。   | 

### Performance 

| 选项   | 描述   | 
|:----|:----|
| --dangerous-unverified-import   | 安全默认值是运行验证（值0）。 如果你从自己的可信区块链中导入（我们假设已经过验证），可以启用--dangerous-unverified-import。 “危险”模式将大大加快这一进程。   | 
| --batch   | 是否持续保存到磁盘（默认值为1），或者可以在RAM中执行所有操作，并将所有内容保存到最后（值0）。只有在没有验证（-dangerous-unverified-import）的情况下，非批处理才有效。另见--batch-size。   | 
| --batch-size   | 以区块数表示的保存到磁盘的频率。默认为每隔5000个区块（验证时）或每20000个块（未验证时）保存一次。 大批量更快但需要更多RAM。   | 
| --resume   | 如果输出数据库已存在，则从当前高度恢复（默认值为1）。 更改为--resume = 0的 变化不大——现有的区块很快被跳过，进程恢复（existing blocks are skipped pretty quickly and the process is resumed anyway.）。   | 

## Reference 
* Https://github.com/monero-project/monero/tree/master/src/blockchain_utilities
# 配置文件
## 适用性
默认情况下，Monero 在门罗币数据目录中查找 bitmonero.conf。
要使用特定的配置文件 add -- config-file 选项:
```
./monerod --config-file=/etc/monerod.conf
```
-- config-file 选项可用于:

*  monerod
* monero-wallet-cli
* monero-wallet-rpc
* monero-gen-trusted-multisig
## 语法
```
option-name=value
valueless-option-name=1 适用于非期望值
# comment
```
* whitespace is ignored

 
## 参考
所有配置选项与二进制文件的命令行选项相同。
* [monerod reference](https://monerodocs.org/interacting/monerod-reference)
* [monero-wallet-cli reference](https://monerodocs.org/interacting/monero-wallet-cli-reference)
* [monero-wallet-rpc referenc](https://monerodocs.org/interacting/monero-wallet-rpc-reference)

 
从 -- option-name 中跳过。
例子:
```
./monerod --log-level=4 --stagenet
```
翻译过来是:
```
log-level=4
stagenet=1     # use value "1" to enable the value-less options like --stagenet
```
## 示例 
### monerod.conf 
这个配置是为生产服务器的使用量身定制的。
```
# /etc/monero/monerod.conf

# Data directory (blockchain db and indices)
data-dir=/home/monero/.monero  # Remember to create the monero user first

# Log file
log-file=/var/log/monero/monerod.log
max-log-file-size=0            # Prevent monerod from managing the log files; we want logrotate to take care of that

# P2P full node
p2p-bind-ip=0.0.0.0            # Bind to all interfaces (the default)
p2p-bind-port=18080            # Bind to default port

# RPC open node
rpc-bind-ip=0.0.0.0            # Bind to all interfaces
rpc-bind-port=18081            # Bind on default port
confirm-external-bind=1        # Open node (confirm)
restricted-rpc=1               # Prevent unsafe RPC calls
no-igd=1                       # Disable UPnP port mapping

# Slow but reliable db writes
db-sync-mode=safe

# Emergency checkpoints set by MoneroPulse operators will be enforced to workaround potential consensus bugs
# Check https://monerodocs.org/infrastructure/monero-pulse/ for explanation and trade-offs
enforce-dns-checkpointing=1

out-peers=64              # This will enable much faster sync and tx awareness; the default 8 is suboptimal nowadays
in-peers=1024             # The default is unlimited; we prefer to put a cap on this

limit-rate-up=1048576     # 1048576 kB/s == 1GB/s; a raise from default 2048 kB/s; contribute more to p2p network
limit-rate-down=1048576   # 1048576 kB/s == 1GB/s; a raise from default 8192 kB/s; allow for faster initial sync
```
### monero-wallet-cli.conf 
这个配置是为 stagenet 上的桌面版量身定制的。
```
# $HOME/.bitmonero/stagenet/monero-wallet-cli.conf

# Pick network
stagenet=1

# Connect to a remote full node    
daemon-address=monero-stagenet.exan.tech:38081
untrusted-daemon=1

# Log file
log-file=/tmp/monero-wallet-cli.log

# wallet-file=/home/YOUR-USER/.bitmonero/stagenet/wallets/MoneroExampleStagenetWallet
```

