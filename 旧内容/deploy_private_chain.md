参考博客：https://bigishdata.com/2017/12/15/how-to-write-deploy-and-interact-with-ethereum-smart-contracts-on-a-private-blockchain/


# 私有链的创建

首先把这个博客指定的项目代码复制到本机：

`git clone https://github.com/jackschultz/privEth`

这里面有一个genesis.json，包含的是初始区块链的信息。

```
{
  "alloc": {},
  "config": {
    "chainID": 72,
    "homesteadBlock": 0,
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0
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

貌似你可以填个参数作为passphrase，比如`personal.newAccount('123456')`这样。

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

同样，也可以把passphrase写到解锁里面：`personal.unlockAccount(eth.coinbase, '111111')`。

注意，每次重新打开链的时候，都需要解锁才能使用账号。就类似于登录吧。

然后，到链2查看账户地址，也就是`personal.listAccount()`。把这个地址复制到链1，设成一个变量（当然你也可以直接用，就是麻烦了点）：`toAccount = "0x2e60025fd4c3b97f7873275b69ee72c5f089365b"`

接下来就是转账了：`eth.sendTransaction({from: eth.coinbase, to: toAccount, value:100000000})`，其中value就是转账金额，这是用最小单位wei来计算的，并不是eth值。

为了使交易成功，需要用挖矿的形式结算这个交易，也就是刚才提到的`miner.start()`。

当挖了一个区块之后停下，就可以转到链2这边查看余额，看看转账是否成功了。

我是在链2这边挖的，所以同时也获得了转账手续费。如果在链1挖大概就只会少掉转账的金额。

# 中场休息

等会就可以开始用remix搞智能合约了！

# 编写合约

remix的地址：http://remix.ethereum.org

要注意，版本是很重要的……教程中的示例给的编译器版本是0.4.0的，记得到右上角改好。合约的代码如下：

```
pragma solidity ^0.4.0;
contract Questions {

  //global variables that aren't in a struct
  mapping(address => uint) public answers; //integer where 0 means hasn't answered, 1 means yes, 2 means no
  string question;
  address asker;
  uint trues;
  uint falses;

  /// __init__
  function Questions(string _question) public {
    asker = msg.sender;
    question = _question;
  }
  
  //We need a way to validate whether or not they've answered before.
  //The default of a mapping is 
  function answerQuestion (bool _answer) public {
    if (answers[msg.sender] == 0 && _answer) { //haven't answered yet
      answers[msg.sender] = 1; //they vote true
      trues += 1;
    }
    else if (answers[msg.sender] == 0 && !_answer) {
      answers[msg.sender] = 2; //falsity
      falses += 1;
    }
    else if (answers[msg.sender] == 2 && _answer) { // false switching to true
      answers[msg.sender] = 1; //true
      trues += 1;
      falses -= 1;
    }
    else if (answers[msg.sender] == 1 && !_answer) { // true switching to false
      answers[msg.sender] = 2; //falsity
      trues -= 1;
      falses += 1;
    }
  }
 
  function getQuestion() public constant returns (string, uint, uint, uint) {
    return (question, trues, falses, answers[msg.sender]);
  }
}
```

把这个东西复制过去，然后编译。看到绿框框大概就完事了。

看到下面的Details，里面会有abi和bytecode。那个bytecode里面有很多信息的，记得只要那串数字。

接下来和NodeJS相关的就没看不太懂了，这里用别的方法部署合约。

# 部署合约

先把abi和bytecode送进geth。

## abi

复制了abi之后，你的剪贴板会是这种东西：

```
[
	{
		"constant": true,
		"inputs": [
			{
				"name": "",
				"type": "address"
			}
		],
		"name": "answers",
		"outputs": [
			{
				"name": "",
				"type": "uint256"
			}
		],
		"payable": false,
		"type": "function",
		"stateMutability": "view"
	},
	{
		"constant": true,
		"inputs": [],
		"name": "getQuestion",
		"outputs": [
			{
				"name": "",
				"type": "string"
			},
			{
				"name": "",
				"type": "uint256"
			},
			{
				"name": "",
				"type": "uint256"
			},
			{
				"name": "",
				"type": "uint256"
			}
		],
		"payable": false,
		"type": "function",
		"stateMutability": "view"
	},
	{
		"constant": false,
		"inputs": [
			{
				"name": "_answer",
				"type": "bool"
			}
		],
		"name": "answerQuestion",
		"outputs": [],
		"payable": false,
		"type": "function",
		"stateMutability": "nonpayable"
	},
	{
		"inputs": [
			{
				"name": "_question",
				"type": "string"
			}
		],
		"type": "constructor",
		"payable": true,
		"stateMutability": "payable"
	}
]
```

这东西好像没办法就这么喂进去，所以你需要先转义，然后用内部JSON库处理。

转义的网站在：http://www.bejson.com/jsonviewernew/

点进去之后，粘贴，删除空格并转义，得到类似下面这样的东西：

```
[{\"constant\":true,\"inputs\":[{\"name\":\"\",\"type\":\"address\"}],\"name\":\"answers\",\"outputs\":[{\"name\":\"\",\"type\":\"uint256\"}],\"payable\":false,\"type\":\"function\",\"stateMutability\":\"view\"},{\"constant\":true,\"inputs\":[],\"name\":\"getQuestion\",\"outputs\":[{\"name\":\"\",\"type\":\"string\"},{\"name\":\"\",\"type\":\"uint256\"},{\"name\":\"\",\"type\":\"uint256\"},{\"name\":\"\",\"type\":\"uint256\"}],\"payable\":false,\"type\":\"function\",\"stateMutability\":\"view\"},{\"constant\":false,\"inputs\":[{\"name\":\"_answer\",\"type\":\"bool\"}],\"name\":\"answerQuestion\",\"outputs\":[],\"payable\":false,\"type\":\"function\",\"stateMutability\":\"nonpayable\"},{\"inputs\":[{\"name\":\"_question\",\"type\":\"string\"}],\"type\":\"constructor\",\"payable\":true,\"stateMutability\":\"payable\"}]
```

在geth里面，用JSON.parse：

`abi = JSON.parse('[{\"constant\":true,\"inputs\":[{\"name\":\"\",\"type\":\"address\"}],\"name\":\"answers\",\"outputs\":[{\"name\":\"\",\"type\":\"uint256\"}],\"payable\":false,\"type\":\"function\",\"stateMutability\":\"view\"},{\"constant\":true,\"inputs\":[],\"name\":\"getQuestion\",\"outputs\":[{\"name\":\"\",\"type\":\"string\"},{\"name\":\"\",\"type\":\"uint256\"},{\"name\":\"\",\"type\":\"uint256\"},{\"name\":\"\",\"type\":\"uint256\"}],\"payable\":false,\"type\":\"function\",\"stateMutability\":\"view\"},{\"constant\":false,\"inputs\":[{\"name\":\"_answer\",\"type\":\"bool\"}],\"name\":\"answerQuestion\",\"outputs\":[],\"payable\":false,\"type\":\"function\",\"stateMutability\":\"nonpayable\"},{\"inputs\":[{\"name\":\"_question\",\"type\":\"string\"}],\"type\":\"constructor\",\"payable\":true,\"stateMutability\":\"payable\"}]')`

然后返回处理结果，那就对了。

## bytecode

从remix复制出来的bytecode很多别的信息，包括了OPCODE啥的。这个的话需要自己手动把bytecode挑选出来。

`bytecode = '0x123456...'`

注意两点：字节码要拿引号括起来，然后记得在前面加`0x`。最后返回的是一个绿色的字符串。

## 创建合约对象以及生成对象实例

现在已经有两个关键信息，`abi`和`bytecode`，现在要拿它们创建合约了。

下面是根据abi创建合约对象

`myContract = eth.contract(abi)`

然后生成对象实例。记得解锁账号。

`contractInstance = myContract.new("nmd, why?",{data:bytecode, from:eth.coinbase, gas:10000000})`

第1到第n-1个参数是传给构造函数的，最后的是合约的相关信息。gas我还不懂具体含义，部署的手续费是`eth.estimateGas({data: bytecode})`，不要超过gaslimit就行了（这个也不会超过的）。

生成好之后就可以开始挖矿`miner.start()`，把合约送进区块链。

挖了一次之后就可以`miner.stop()`。现在就可以试一下调用合约。

`ci.getQuestion()`

返回`["nmd, why?", 0, 0, 0]`的话，恭喜你，部署成功!
