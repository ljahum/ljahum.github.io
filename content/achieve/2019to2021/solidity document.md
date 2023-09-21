---
title: "A tour of solidity"
subtitle: 
date: 2021-11-27 00:07:00+08:00
# weight: 1000
draft: true
author: "ljahum"
description: "join the evaluation"
tags: 
- codes
# crypto math codes bin Nuil 
categories: 
- posts
# - CTF posts notes 其他

# 内波标题图片
featuredImage: 

# 外部标图图片
featuredImagePreview: 

hiddenFromHomePage: false
math:
  enable: true
---
<!--more-->

## **A Simple Smart Contract**

To access a member (like a state variable) of the current contract, you do not typically add the `this.` prefix, you just access it directly via its name. Unlike in some other languages, omitting it is not just a matter of style, it results in a completely different way to access the member, but more on this later.



![](https://raw.githubusercontent.com/ljahum/images/main/img/Untitled.png)

remix使用的三种环境，分别指:

- js VM:remix自带的sandbox环境练习环境 虚拟了一个节点
- inject web3和provider web3都是想要连接真实节点

连接matemask一般用 inject web3 

连接ganache的本地虚拟用provider web3?

![](https://raw.githubusercontent.com/ljahum/images/main/img/Untitled%201.png)

![](https://raw.githubusercontent.com/ljahum/images/main/img/Untitled%202.png)

先搞一个数据存入storedata,get函数取出

## 练习平台

[https://cryptozombies.io/zh/](https://cryptozombies.io/zh/)

一个solidity练习平台，有点像js

## stage1

基础+event

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {
	//event 是合约和区块链通讯的一种机制。你的前端应用“监听”某些事件，并做出反应。
    event NewZombie(uint zombieId, string name, uint dna);
    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string _name, uint _dna) private {
			//array.push() - 1 将是我们加入的僵尸的索引。 
			//zombies.push() - 1 就是 id，数据类型是 uint。
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        NewZombie(id, _name, _dna);
    }
		//函数修饰符
		//view 意味着它只能读取数据不能更改数据
    function _generateRandomDna(string _str) private view returns (uint) {
			//Ethereum 内部有一个散列函数keccak256，它用了SHA3版本。
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }

    function createRandomZombie(string _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

} 
```

Solidity 还支持 ***pure*** 函数, 表明这个函数甚至都不访问应用里的数据，例如：

```solidity
function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}
```

## stage2

### addresses （地址

以太坊区块链由  **_ account _**  (账户)组成，你可以把它想象成银行账户。一个帐户的余额是  **ETH** （在以太坊区块链上使用的币种），你可以和其他帐户之间支付和接受以太币，就像你的银行帐户可以电汇资金到其他银行帐户一样。

每个帐户都有一个“地址”，你可以把它想象成银行账号。这是账户唯一的标识符，它看起来长这样：

`0x0cE446255506E92DF41614C46F1d6df9Cc969183`

此处的address是一种独立的数据类型

### **Mapping（映射）**

在第1课中，我们看到了 **_ 结构体 _** 和 **_ 数组 _** 。 **_映射_** 是另一种在 Solidity 中存储有组织数据的方法。

类似于python的字典，支持通过key查找value

```solidity
//对于金融应用程序，将用户的余额保存在一个 uint类型的变量中：
mapping (address => uint) public accountBalance;
//或者可以用来通过userId 存储/查找的用户名
mapping (uint => string) userIdToName;
```

### **msg.sender**

在 Solidity 中，有一些全局变量可以被所有函数调用。 其中一个就是 `msg.sender`，它指的是当前调用者（或智能合约）的 `addres`(即creator or admin)

### require

这玩意类似python的断言，若为假则会停止执行,并重置require管辖区域的修改

### **继承（Inheritance）**

 当代码过于冗长的时候，最好将代码和逻辑分拆到多个不同的**合约**中，以便于管理。

有个让 Solidity 的代码易于管理的功能，就是**合约 *inheritance* (继承)**：

```solidity
contract Doge {
  function catchphrase() public returns (string) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string) {
    return "Such Moon BabyDoge";
  }
}
```

### **Storage与Memory**

***Storage*** 变量是指永久存储在区块链中的变量。 ***Memory*** 变量则是临时的，当外部函数对某合约调用完成时，内存型变量即被移除。 你可以把它想象成存储在你电脑的硬盘或是RAM中数据的关系

### **函数可见性**

我们尝试从 `ZombieFeeding` 中调用 `_createZombie` 函数，但 `_createZombie` 却是 `ZombieFactory` 的 `private` （私有）函数。

`internal` 和 `private` 类似，不过， 如果某个合约继承自其父合约，这个合约即可以访问父合约中定义的“内部”函数。

`external` 与`public` 类似，只不过这些函数只能在合约之外调用

```solidity
pragma solidity ^0.4.19;

import "./zombiefactory.sol";

contract ZombieFeeding is ZombieFactory {

  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    _createZombie("NoName", newDna);
  }

}

-----
	function _createZombie(string _name, uint _dna) internal {
	        uint id = zombies.push(Zombie(_name, _dna)) - 1;
	        zombieToOwner[id] = msg.sender;
	        ownerZombieCount[msg.sender]++;
	        NewZombie(id, _name, _dna);
	 }
----
```

### **与其他合约的交互**

如果我们的合约需要和区块链上的其他的合约会话，则需先定义一个 ***interface*** (接口)。

先举一个简单的栗子。 假设在区块链上有这么一个合约：

```solidity
contract LuckyNumber {
  mapping(address => uint) numbers;

  function setNum(uint _num) public {
    numbers[msg.sender] = _num;
  }

  function getNum(address _myAddress) public view returns (uint) {
    return numbers[_myAddress];
  }
}

```

这是个很简单的合约，您可以用它存储自己的幸运号码，并将其与您的以太坊地址关联。 这样其他人就可以通过您的地址查找您的幸运号码了。

现在假设我们有一个外部合约，使用 `getNum` 函数可读取其中的数据。

首先，我们定义 `LuckyNumber` 合约的 ***interface*** ：

```solidity
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

请注意，这个过程虽然看起来像在定义一个合约，但其实内里不同：

首先，我们只声明了要与之交互的函数 —— 在本例中为 `getNum` —— 在其中我们没有使用到任何其他的函数或状态变量。

其次，我们并没有使用大括号（`{` 和 `}`）定义函数体，我们单单用分号（`;`）结束了函数声明。这使它看起来像一个合约框架。

编译器就是靠这些特征认出它是一个接口的。

### **使用接口**

继续前面 `NumberInterface` 的例子，我们既然将接口定义为：

```solidity
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}

```

我们可以在合约中这样使用：

```solidity
contract MyContract {
  address NumberInterfaceAddress = 0xab38... 
  // ^ The address of the FavoriteNumber contract on Ethereum
  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);
  // Now `numberContract` is pointing to the other contract

  function someFunction() public {
    // Now we can call `getNum` from that contract:
		//利用接口来访问其他地址中的合约,(solidity特有交互模式)
    uint num = numberContract.getNum(msg.sender);
    // ...and do something with `num` here
  }
}
```

通过这种方式，只要将您合约的可见性设置为`public`(公共)或`external`(外部)，它们就可以与以太坊区块链上的任何其他合约进行交互。

**NumberInterface** numberContract = **NumberInterface**(NumberInterfaceAddress);

> 至此，我们掌握了基本的交互方法，可以尝试去完成一些测试链的智能合约操作了
>
> 哦，注意一下，在开始前，最好找大哥们要一点测试链的ETH存入matemask以作为gas😀😀😀

### **处理返回值**

像python一样,sodity可以处理大量的返回值

```solidity
function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  // 这样来做批量赋值:
  (a, b, c) = multipleReturns();
}

// 或者如果我们只想返回其中一个变量:
function getLastReturnValue() external {
  uint c;
  // 可以对其他字段留空:
  (,,c) = multipleReturns();
}
```