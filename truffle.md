本文将记录用truffle部署合约的方法。

## 安装
首先需要nodejs：
```
sudo apt install nodejs
```

然后还需要npm：
```
sudo apt install npm
```

用npm下载truffle和ganache-cli：
```
sudo npm install -g truffle 
sudo npm install -g ganache-cli
```
后者是提供测试网络用的。

## 初始化项目
新建一个目录，然后在里面输入
```
truffle init
```
来新建一个项目。

tree一下， 应该有这些东西：
```
$ tree
.
├── contracts
│   └── Migrations.sol
├── migrations
│   └── 1_initial_migration.js
├── test
└── truffle-config.js

3 directories, 3 files
```

## compile
接下来开始编译，注意使用sudo。编译过程需要在项目的根目录，然后结果会放在build里面。不要更改里面的东西。
```
$ sudo truffle compile
Compiling ./contracts/Migrations.sol...
Writing artifacts to ./build/contracts

$ tree
.
├── build
│   └── contracts
│       └── Migrations.json
├── contracts
│   └── Migrations.sol
├── migrations
│   └── 1_initial_migration.js
├── test
└── truffle-config.js

5 directories, 4 files

```

## ganache-cli
实际上，ganache-cli是命令行版本的测试网络。如果想搞GUI的可以用ganache。

不知道truffle migrate的默认端口和网络id是不是固定的，总之我这边端口是7545，网络id是5777。所以ganache-cli启动的时候也要对应这个参数：
```
ganache-cli -p 7545 -i 5777
```

## migration
这个好像是指导部署用的，需要自己写，当然后面可能可以自己搞个一键书写的工具。

如果是用`truffle init`的模板，里面应该已经有一个`1_initial_migration.js`了：
```
var Migrations = artifacts.require("./Migrations.sol");

module.exports = function(deployer) {
  deployer.deploy(Migrations);
};
```

我在doc上看到的和这里的有点出入，因为doc上写的是，artifacts.require的参数应该是合约名称的字符串而不是sol文件名，因为一个sol里可能有多个合约。这里不知道是不是可以将sol的所有合约都设进去。

然后基本上就是照着填就行了，deployer的用法可以看 https://truffleframework.com/docs/truffle/getting-started/running-migrations 的末尾。基本上没什么好看的，就是三点：
```
// Deploy a single contract without constructor arguments
deployer.deploy(A);

// Deploy a single contract with constructor arguments
deployer.deploy(A, arg1, arg2, ...);

// Deploy multiple contracts, some with arguments and some without.
// This is quicker than writing three `deployer.deploy()` statements as the deployer
// can perform the deployment as a single batched request.
deployer.deploy([
  [A, arg1, arg2, ...],
  B,
  [C, arg1]
]);
```

## migrate
首先确保已经开启了ganache-cli。然后回到根目录
```
sudo truffle migrate
```
migrate之前还会帮你compile一下，非常贴心。

## migrate与ganache-cli的配合
因为truffle不只是测试工具，所以它可以用在公链上，也可以用在私有网络上，调整的方法是查看`truffle-config.js`。

在migrate的时候，如果没有检测到合适的网络，它会列出正在监听的网路的参数：
```
Could not connect to your Ethereum client with the following parameters:
    - host       > 127.0.0.1
    - port       > 7545
    - network_id > 5777
```

这时候你就可以用合适的参数重新启动一下ganache-cli。

当然，这种调整的方法不是权宜之计，所以我们可以调`truffle-config.js`，确保migrate的网络设置是固定的。

举个例子，如果要部署到公链，可以写成这样：
```
module.exports = {
  networks: {
    "live": {
      network_id: 1,
      host: "127.0.0.1",
      port: 8546   // Different than the default below
    }
  },
  rpc: {
    host: "127.0.0.1",
    port: 8545
  }
};
```

然后在migrate时启用`--network live`。但是我们不需要到公链上跑，所以可以自己写一个。

在模板里其实已经写好一个设置了，但是被注释了，是这样子的：
```
...
// development: {
    //  host: "127.0.0.1",     // Localhost (default: none)
    //  port: 8545,            // Standard Ethereum port (default: none)
    //  network_id: "*",       // Any network (default: none)
    // },
...
```
如果取消掉注释，就可以确保mirgate总是可以连接ganache-cli了。

## 与合约实例互动
在ganache-cli开着的情况下，使用
```
truffle console
```
可以进入一个类似geth console的控制台，不过功能有些不同。

```
truffle(develop)> let instance = await MetaCoin.deployed()
truffle(develop)> instance

// outputs:
//
// Contract
// - address: "0xa9f441a487754e6b27ba044a5a8eb2eec77f6b92"
// - allEvents: ()
// - getBalance: ()
// - getBalanceInEth: ()
// - sendCoin: ()
// ...
```

基本上就是这样了。如果要用web3的功能，记得不能像geth那样直接`eth.balabala`，要加`web3`前缀。

如果需要退出console，使用`.exit`。退出ganache我还没发现有什么方法，ctrl+C吧……
