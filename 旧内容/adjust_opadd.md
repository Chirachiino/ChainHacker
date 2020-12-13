本章记录了如何修改OPCODE中的opAdd，并构造一个场景去测试overflow。从这个样例出发可以自行实现underflow的检测。

## 魔改evm 
下载`go-ethereum`，然后打开`/core/vm/instructions.go`。里面是所有OPCODE的go语句实现。

搜索`opAdd`，然后在其中加入一些记录语句。在控制台的环境下，你可以用`fmt.Println()`：
```
func opAdd(pc *uint64, interpreter *EVMInterpreter, contract *Contract, memory *Memory, stack *Stack) ([]byte, error) {
	x, y := stack.pop(), stack.peek()
	prevX := x // 记录被加数
	fmt.Println("ADD",x,y) // 展示操作数
	math.U256(y.Add(x, y))
	fmt.Println("ADD result",y) // 展示相加结果
	if(prevX.Cmp(y) > 0){ // big.Int对象不能直接比较大小，需要用.Cmp()来测试大小。考虑overflow时结果会比原加数小，进行这样的判断。
		fmt.Println("Result may overflow!")
	}

	interpreter.intPool.put(x)
	return nil, nil
}
```

然后在go-ethereum主目录下编译：`go install -v ./cmd/...`

## 部署合约
下面是一个简单的overflow测试代码，建议使用0.4.25去编译。
```
pragma solidity ^0.4.15;

contract Overflow {
    uint private sellerBalance=0;
    
    function add(uint value) public returns (bool){
        sellerBalance += value; // possible overflow
        
        // possible auditor assert
        // assert(sellerBalance >= value); 
    } 

    function safe_add(uint value) public returns (bool){
        require(value + sellerBalance >= sellerBalance);
        sellerBalance += value; 
    } 
    function getBalance() public view returns(uint){
        return sellerBalance;
    }
}
```
部署的方法见geth私链架设的文章。

## 产生overflow样例
有以下几个要注意的地方：
* 对于uint类型，它的长度是256。
* 在EVM内部可以保存精准的大值，但是在函数入口是没办法直接喂一个`Math.pow(2,256)-1`作参数的。
    * 函数参数还会把大数的尾数消掉！例如参数填`Math.pow(2,255)`在EVM中显示出来是`57896044618658100000000000000000000000000000000000000000000000000000000000000`
* 在调用add()之前，可能会遇到`invalid address`的错误。这是因为没有设置defaultAcconut。这是一个每次启动都要设置的参数，设置的方法是`eth.defaultAccount = eth.coinbase`

接下来就正式开始。首先，考虑到你要多次编译geth，你可以在创建合约实例的时候记录下合约的地址，比如：
```
> inst = cont.new({data:bytecode, from:eth.coinbase, gas:1000000})
INFO [03-28|16:09:46.450] Submitted contract creation              fullhash=0xb8fce0e30b98da43c215c4ea2dd2efdabb567e293252ff89100132fd4c84e26e contract=0xf9Ca42D126c270011Da14770EB9f76B9227bE90a
```

然后，你就可以用`.at()`去生成地址上的合约实例。

```
inst = eth.contract(abi).at('0xf9Ca42D126c270011Da14770EB9f76B9227bE90a')
```

一切准备妥当，先`inst.add(Math.pow(2,255))`，那么你会看到一串调试信息飘过。但是这还没有溢出。再加一两次，你大概就能看到overflow的提示信息了：
```
> inst.add(Math.pow(2,255))
ADD 4 32
ADD result 36
IADD 32 4
ADD result 36
NADD 57896044618658109152858029982624184293460030668718871921084831984173740720328 57896044618658100000000000000000000000000000000000000000000000000000000000000
FADD result 13729287044973936276440190046003078307881627247976260611080392
O Result may overflow!
[0ADD 32 128
3ADD result 160
-28|17:13:07.357] Submitted transaction                    fullhash=0x676a085bc85f6b8d8d80028b313e55302d309b1b458d501ddb6577f00839985b recipient=0xf9Ca42D126c270011Da14770EB9f76B9227bE90a
"0x676a085bc85f6b8d8d80028b313e55302d309b1b458d501ddb6577f00839985b"
>
```
