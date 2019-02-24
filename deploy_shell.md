知道truffle怎么使用之后一键部署就很简单了。

# 创建脚本
打开truffle项目根目录，首先写好这样的脚本：
```
# start ganache testrpc
gnome-terminal -t “ganache-cli” -- ganache-cli -p 8545 
# clean up the migration
rm ./migrations/* 
# write migrations
echo "var mc = artifacts.require(\"./$1\");" >> ./migrations/1_autodepoly_all.js
echo "module.exports = function(deployer) {" >> ./migrations/1_autodepoly_all.js
echo "deployer.deploy(mc);};" >> ./migrations/1_autodepoly_all.js
# migrate
sudo truffle migrate
truffle console
```
保存为，比如说，`deploy.sh`。这里我没有写`#!/bin/sh`竟然也过了，不过还是建议加上。

然后给这个脚本添加可执行权限：
```
chmod +x "deploy.sh"
```

# 使用方法
首先，到`./contracts`放置你的.sol文件，然后记住你需要部署的合约名称。下面提供一个测试合约：
```
pragma solidity ^0.5.0;

contract HelloWorld {

  //say hello world
  function say() public pure returns (string memory) {
    return "Hello World";
  }

  //print name
  function print(string memory name) public pure returns (string memory) {
    return name;
  }
}
```

然后，在终端运行
```
./deploy HelloWorld
```

这时候你就会看到新打开了一个窗口，那是ganache-cli。然后应该会让你输入sudo命令的密码。

接着程序自动部署并打开truffle console。要测试合约的话，先生成合约实例：
```
truffle(development)> let instance = await HelloWorld.deployed()
undefined
# truffle(development)> instance
# TruffleContract {
#  constructor: 
#   { [Function: TruffleContract]
#     _constructorMethods: 
#      { setProvider: [Function: setProvider],
#        new: [Function: new],
# ...
```

尝试一下
```
truffle(development)> instance.say()
'Hello World'
```

最后退出console，输入`.exit`。
