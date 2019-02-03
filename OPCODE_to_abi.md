# 储存机制
在EVM中有三个储存位置：
* stack
* mem
* storage
其中前两个在evm初始化时定义，最后一个储存在区块链上。

不同的储存位置有不同的读写操作。
* stack
   * PUSH - 将数据放入栈，数字代表放入数据的字节长度
   * POP
* mem
   * MSTORE(arg0, arg1) - 将arg1放入arg0
   * MLOAD(arg0) - 取出arg0
* storage
   * SSTORE
   * SLOAD
   * 参数和作用同mem。获取storage位置的方法是eth.getStorageAt(addressOfContract, slot)

# 变量
## 定长变量
slot按256bits对齐，不够就往后另外加空间。如果一个slot足够存下两个变量，就可以这么放进去。

`
uint a;       // slot = 0, 256bits
address b;    // 1, len = 160bits
ufixed c;     // 2, 256bits
bytes32 d;    // 3, 256bits
`
> 至于同slot内的读取原理，日后再补。

## 映射变量
类似这样的变量：`mapping(address => uint) a;`，
储存方法是`SLOAD(sha3(key.rjust(64, "0") + slot.rjust 64, "0")))`。

其中rjust是向右对齐，这条代码里面即是在左边补零直到64位。

> 至于为什么key和slot可以直接无序地加起来，后面再看看是怎么回事

## 变长变量
就是数组这样的变量。此时变量的slot存的是变量长度。

为了避免越界访问，会先拿index和SLOAD(变量所在slot)比较。

访问时，先计算slot的sha3，然后第X个元素就是SLOAD(hashValue + X)。

> string的过长储存是什么意思？

## 结构体
结构体仅仅是一个方便的声明，在slot中还是会把变量全部展开再储存。

#
