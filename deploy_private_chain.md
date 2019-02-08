参考博客：https://bigishdata.com/2017/12/15/how-to-write-deploy-and-interact-with-ethereum-smart-contracts-on-a-private-blockchain/


# 私有链的创建

首先把这个博客指定的项目代码复制到本机：

`git clone https://github.com/jackschultz/privEth`

这里面有一个genesis.json，包含的是初始区块链的信息。

```
{
  "alloc": {},
  "config": {
    "homesteadBlock": 0,
    "chainID": 72,
    "eip155Block": 0,
    "eip158Block": 0
  },
  "nonce": "0x0000000000000000",
  "difficulty": "0x4000",
  "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000", 
  "coinbase": "0x0000000000000000000000000000000000000000",
  "timestamp": "0x00",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "extraData": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "gasLimit": "0xffffffff"
}
```

其中：
* nonce和mixhash共同作为这个区块的工作量证明的验证。
* difficulty是工作量证明的计算难度，在这里设置得很低来保证区块的快速生成。
* alloc包含预设的钱包，这里就不必填写了。
* coinbase貌似是本区块的奖励？
* timestamp用于平衡区块生成速度
* parentHash包含上一区块的hash信息，只有在genesis里被指定为了0
* gasLimit指定了单个区块使用gas的上限，在这里设得很高，保证不被限制而出错。

当复制结束之后，进入那个包含`genesis.json`的目录，然后运行：

`geth --datadir "~/.ethereum/privEth" init genesis.json`

因为要和各种数据文件夹分开，所以要用`--datadir`指定本私链的保存文件夹。数据文件夹的路径就在`~/.ethereum/`，自己指定本私链的名称就是了。

接下来再创建一个区块链（怎么回事？），用相同的命令，不同的名称：

`geth --datadir "~/.ethereum/privEth2" init genesis.json`

现在你到`~/.ethereum`里面`ls`一下，应该就会有这两个文件夹了。

# 控制台操作

现在就得开始在两个控制台里分别打开这两个新建的链，命令：

`geth --datadir "~/.ethereum/privEth" --networkid 72 --port 30301 --nodiscover console`

其中：
* --datadir要和对应上刚才两个区块链。
* --networkid是和genesis那里对应的。我也不懂genesis那里写的什么意思，总之不要用1-4就是了，因为那些是默认保留的网络id，比如3就是ropsten测试链。
* --port指定数据连接的端口之类的。因为默认是30303，所以这里挑了个30301。大概没有什么特殊的。
* --nodiscober是让别的节点不要发现本链，意义稍后再谈。
* console就是以控制台界面运行了。

# 创建coinbase账号

`personal.listAccounts`可以查看当前的账户列表。当然现在是没有账户的，需要自己建立一个。

`personal.newAccount()`就可以建立一个新账户，它会要求你用一个passphrase，作者用了'passphrase'，这里咱用的是111111和222222。

创建过程可能会爆内存，可以考虑开大一点。

# 把这两个节点连接起来

为了方便，接下来把两个链称作链1和链2。

首先，在链1用`admin.nodeInfo.enode`把这个节点的node信息弄出来，大概长这样的：

`"enode://72f6cd2f459bfa87ff2fe77ef2105f3d2236b080f9c35f45d235f53729b1a471071482d01219a585e26c2b695c39a15c6c286c1303f8a40285450cec8d13a7c9@127.0.0.1:30301?discport=0"`

那么，在链2用`admin.addPeer`连接节点。

`admin.addPeer("enode://72f6cd2f459bfa87ff2fe77ef2105f3d2236b080f9c35f45d235f53729b1a471071482d01219a585e26c2b695c39a15c6c286c1303f8a40285450cec8d13a7c9@127.0.0.1:30301?discport=0")`

正常的话会返回一个true。这样，无论在链1还是2，就可以用`admin.peers`即可查看已连接的节点。

# 开始挖矿

在链1用`miner.start()`开始挖矿。可能要等一会才会开始，不急。

大概觉得挖够了就`miner.stop()`结束挖矿。

查询余额`eth.getBalance(eth.coinbase)`。

# 转账

在链1这边，发送eth之前需要用`personal.unlockAccount(eth.coinbase)`把coinbase账号解锁。输入刚才设定的passphrase。

然后，到链2查看账户地址，也就是`personal.listAccount()`。把这个地址复制到链1，设成一个变量（当然你也可以直接用，就是麻烦了点）：`toAccount = "0x2e60025fd4c3b97f7873275b69ee72c5f089365b"`

接下来就是转账了：`eth.sendTransaction({from: eth.coinbase, to: toAccount, value:100000000})`，其中value就是转账金额，这是用最小单位wei来计算的，并不是eth值。

为了使交易成功，需要用挖矿的形式结算这个交易，也就是刚才提到的`miner.start()`。

当挖了一个区块之后停下，就可以转到链2这边查看余额，看看转账是否成功了。

我是在链2这边挖的，所以同时也获得了转账手续费。如果在链1挖大概就只会少掉转账的金额。

# 中场休息

等会就可以开始用remix搞智能合约了！
