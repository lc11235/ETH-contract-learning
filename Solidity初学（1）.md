## 一、合约开发流程

语言：使用node.js开发该项目

大概流程：合约代码编写（Solidity）-> 合约编译（solc）-> 合约部署（web3）

### 开发语言及工具

* 区块链节点：ganache-cli
* 基础环境：node
* 合约开发语言：Solidity
* 合约编译器：solc
* 合约访问库：web3.js

## 二、基础环境安装

* 1、安装node.js
* 2、安装ganache-cli

```sh
sudo npm install -g ganache-cli
```

运行：

```sh
ganache-cli
```

输出：

```sh
➜  ~ ganache-cli
Ganache CLI v6.1.0 (ganache-core: 2.1.0)

Available Accounts
==================
(0) 0x2c0e940b0732a3b49cb7dccc26e14cb4801dd1c3
(1) 0x65afabcf1fdb19ef88f8433af90136de56e7e412
(2) 0x65111c1fa94e15e8e3bdedb466004f67d6b46bab
(3) 0xfa44030a4216193d19a811267528e86cf1851e48
(4) 0xc29473dca76a2ebbb8b1badf6a8093c11b56ea84
(5) 0x06e55addeef67a46015e2790be1ada1deb3c9c70
(6) 0xc1ec7f3d08692d0bdd70d6ab3d5701f22f53a521
(7) 0x42e52cbb5e226ef8c2c9bf54737b87ccf94ebb08
(8) 0x8cebfdb948266022d2499118b0989b290d146d4c
(9) 0x17b791127c57dff3eb31cc203e404536ef7e0ba7

Private Keys
==================
(0) 3bb508f1c2c35083f7d69466830067c6582e4464ba61daffc947bb1aa98618e9
(1) fc06e722c10cd80b1b5b43355f81363dcbe6dcc8d3c59387f69c68ce99f36c53
(2) 07f37ed746ba88da289eaa780d6155d9fee456106d85169ad92a526c22192695
(3) 2619b581c083d20ff84db2688f4a9d836206ee37e993bc8cb1e089ad68c8673f
(4) c3f61de226b5d5c06cb941f93a2a3ec321dabc53a8fb68bee64d3aed5bc130e6
(5) f86e7b7e7a9cf7532004694cb22997ac521567b7c8e480dbee23e426ed787234
(6) 2035f13d8d64109f21e4eb32970e5934cddcd27bc55439634f49d4479c7abe77
(7) 3395049c4f8749b17e154c47199fa42ce538ed051b6240afc55f49d30406a4f0
(8) 976f56be1b1cd9f5c420a3fdb71eb3a8c3875a7bd3fba20c342389ba97b0a165
(9) a2a7a190ee76cdb0675b8af773fba55187ff4a0fc6c1e1021e717d19e0d591ee

HD Wallet
==================
Mnemonic:      result casino this poverty sleep joy toy sort onion spider bind evolve
Base HD Path:  m/44'/60'/0'/0/{account_index}

Listening on localhost:8545
```

ganache默认会自带创建10个账户，每个账户有100个以太币（ETH：Ether）。可以把账户视为银行账户，以太币就是以太坊生态系统中的货币。

里面输出的最后一句话，描述了节点仿真器的监听地址和端口为localhost:8585，在使用web3.js时，需要传入这个地址来告诉web3.js库应当链接到哪一个节点。

### 三、合约设计

我们使用[Solidity]([Truffle | Overview | Documentation | Truffle Suite](https://www.trufflesuite.com/docs/truffle/overview))来编写合约。如果你熟悉面向对象的开发和JavaScript，那么学习Solidity应该非常简单。可以将合约类比于OOP的类：合约中的属性用来声明合约的状态，而合约中的方法则提供修改状态的访问接口。

重点：

* 合约状态是持久化到区块链上的，因此对合约状态的修改需要消耗以太币。
* 只有在合约部署到区块链的时候，才会调用构造函数，并且只调用一次。
* 与web世界里每次部署代码都会覆盖旧代码不同，在区块链上部署的合约是不可改变的，也就是说，如果你更新合约并再次部署，旧的合约仍然会在区块链上存在，并且合约的状态数据也依然存在。新的部署将会创建合约的一个新的实例。

### 四、合约语法

从最基本的入手：

Solidity的代码都包括在合约里面，一份合约就是以太币应用的基本模块，所有的变量和函数属于一份合约，它是你所有应用的起点。

一份名为HelloWorld的空合约如下：

```solidity
contract HelloWorld {

}
```

#### 1、版本指令

所有的Solidity源码都必须冠以“version pragma”——标明Solidity编译器的版本。以避免将来新的编译器可能破坏你的代码。

例如：pragma solidity ^0.4.19; （当前Solidity的最新版本时0.4.19）。

综上所述，下面就是一个最基本的合约——每次建立一个新的项目时的第一段代码：

contract.sol合约文件

```solidity
// 1. 这里写版本指令
pragma solidity ^0.4.19;

// 2. 这里建立智能合约
contract HelloWorld {

}
```

##### 实战演习1：

为了建立我们的僵尸部队，让我们先建立一个基础合约，成为ZombieFactory。

* 建立一个版本为0.4.19，我们的合约基于这个版本的编译器。
* 建立一个空合约ZombieFactory。

Contract.sol

```solidity
pragma solidity ^0.4.19; // 1. 这里写版本指令

// 2. 这里建立智能合约
contract ZombieFactory {

}
```

#### 2、状态变量和整数

状态变量是被永久地保存在合约中。也就是说它们被写入以太币区块链中，想象成写入到一个数据库。

示例：

```solidity
contract Example {
	// 这个无符号整数将会永久的被保存在区块链中
	uint myUnsignedInteger = 100;
}
```

在上面的例子中，定义myUnsignedInteger为uint类型，并赋值100.

##### 无符号整数：uint

uint无符号数据类型，指其值不能是负数，对于有符号的整数存在名为int的数据类型。

> 注：Solidity中，unit实际上是uint256代名词，一个256位的无符号整数。你也可以定位位数少的units——uint8，uint16，uint32，等.....但一般来讲你更愿意使用简单的uint，除非在某些特殊情况下，这我们后面会讲。

##### 实战演练2

我们的僵尸DNA将由一个十六位数字组成。

* 定义dnaDigits为uint数据类型，并赋值16.

Contract.sol

```solidity
pragma solidity ^0.4.19; // 1. 这里写版本指令

// 2. 这里建立智能合约
contract ZombieFactory {

  // 3.定义 dnaDigits 为 uint 数据类型, 并赋值 16
  uint dnaDigits = 16;
}
```

#### 3、数学运算

在Solidity中，数学运算很直观明了，与其他程序设计语言相同：

* 加法：x + y
* 减法：x - y
* 乘法：x * y
* 除法：x / y
* 取模/求余：x % y（例如，13 % 5余3）

Solidity还支持乘方操作（如：x的y次方）

```solidity
uint x = 5 ** 2; // equal to 5^2 = 25
```

##### 实战演练3

为了保证我们的僵尸的DNA只含有16个字符，我们先造一个unit数据，让它等于10^16。这样一来以后我们可以用模运算符%把一个整数变成16位。

* 建立一个unit类型的变量，名字叫dnaModulus，令其等于10的dnaDigits次方。

Contract.sol

```solidity
pragma solidity ^0.4.19; // 1. 这里写版本指令

// 2. 这里建立智能合约
contract ZombieFactory {

  // 3. 定义 dnaDigits 为 uint 数据类型, 并赋值 16
  uint dnaDigits = 16;

  // 4. 10 的 dnaDigits 次方
  uint dnaModulus = 10 ** dnaDigits;
}
```

#### 4、结构体

有时你需要更复杂的数据类型，Solidity提供了结构体：

```solidity
struct Person {
	uint age;
	string name;
}
```

结构体允许你生成一个更复杂的数据类型，它有多个属性。

>注：我们刚刚引进了一个新类型，string。字符串用于保存任意长度的UTF-8编码数据。如：string greeting  = "Hello World!"。

##### 实战演练4

在我们的程序中，我们将创建一些僵尸！每个僵尸将拥有多个属性，所以这是一个展示结构体的完美例子。

* 1.建立一个struct命名为Zombie。
* 2.我们的Zombie结构体有两个属性：name（类型为string），和dna（类型为uint）。

Contract.sol

```solidity
// 1. 这里写版本指令
pragma solidity ^0.4.19; 

// 2. 这里建立智能合约
contract ZombieFactory {

  // 3. 定义 dnaDigits 为 uint 数据类型, 并赋值 16
  uint dnaDigits = 16;

  // 4. 10 的 dnaDigits 次方
  uint dnaModulus = 10 ** dnaDigits;

   // 5.结构体定义
   struct Zombie {
        string name;
        uint dna;

    }
}
```

#### 5、数组

如果你想建立一个集合，可以用数组这样的数据类型。Solidity支持两种数组：静态数组和动态数组：

```solidity
// 固定长度为2的静态数组:
uint[2] fixedArray;
// 固定长度为5的string类型的静态数组:
string[5] stringArray;
// 动态数组，长度不固定，可以动态添加元素:
uint[] dynamicArray;
```

你也可以建立一个结构体类型的数组。例如，上一节提到的Person：

```solidity
Person[] people; // dynamic Array, we can keep adding to it
```

> 记住：状态变量将永久保存在区块链中。所以在你的合约中创建动态数组来保存成结构的数据是非常有意义的。

##### 公共数组

你可以定义public的数组，Solidity会自动创建getter方法，语法如下：

```solidity
Person[] public people;
```

> 其他的合约可以从这个数组读取数据（但不能写入数据），所以这在合约中是一个有用的保存公共数据的模式。

#### 实战演练5

为了把一个僵尸部队保存在我们的APP里，并且能够让其他APP看到这些僵尸，我们需要一个公共数组。

* 创建一个数据类型为Zombie的结构体数组，用public修饰，命名为：zombies。

Contract.sol

```solidity
// 1. 这里写版本指令
pragma solidity ^0.4.19; 

// 2. 这里建立智能合约
contract ZombieFactory {

  // 3. 定义 dnaDigits 为 uint 数据类型, 并赋值 16
  uint dnaDigits = 16;

  // 4. 10 的 dnaDigits 次方
  uint dnaModulus = 10 ** dnaDigits;

   // 5.结构体定义
   struct Zombie {
        string name;
        uint dna;

    }
    
    // 6.数组类型为结构体的公共数组
    Zombie[] public zombies;
}
```

#### 6、定义函数

在Solidity中函数定义的句法如下：

```solidity
function eatHamburgers(string _name, uint _amount) {

}
```

这是一个名为eatHamburgers的函数，它接受两个参数：一个string类型的和一个uint类型的。现在函数内部还是空的。

> 注：习惯上函数里的变量都是以下划线开头（但不是硬性规定）以区别全局变量。我们整个教程都会沿用整个习惯。

我们的函数定义如下：

```solidity
eatHamburgers("vitalik", 100);
```

##### 实战演练6

在我们的应用里，我们要能创建一些僵尸，让我们写一个函数做这件事吧！

* 建立一个函数createZombie。它有两个参数：_name(类型为string)，和\_dna(类型为uint)。

暂时让函数空着——我们在后面会增加内容。

Contract.sol

```solidity
// 1. 这里写版本指令
pragma solidity ^0.4.19; 

// 2. 这里建立智能合约
contract ZombieFactory {

  // 3. 定义 dnaDigits 为 uint 数据类型, 并赋值 16
  uint dnaDigits = 16;

  // 4. 10 的 dnaDigits 次方
  uint dnaModulus = 10 ** dnaDigits;

   // 5.结构体定义
   struct Zombie {
        string name;
        uint dna;

    }
    
    // 6.数组类型为结构体的公共数组
    Zombie[] public zombies;
    
    // 7.创建函数
    function createZombie(string _name, uint _dna){
        
    }
}
```

#### 7、使用结构体和数组

##### 创建新的结构体

还记得上个例子中的Person结构吗？

```solidity
struct Person {
  uint age;
  string name;
}

Person[] public people;
```

现在我们学习创建新的Person结构，然后把它加入到名为people的数组中。

```solidity
// 创建一个新的Person:
Person satoshi = Person(172, "Satoshi");

// 将新创建的satoshi添加进people数组:
people.push(satoshi);
```

你也可以两步并一步，用一行代码更简洁：

```solidity
people.push(Person(16, "Vitalik"));
```

> 注：array.push()在数组的尾部加入新元素，所以元素在数组中的顺序就是我们添加的顺序，如：

```solidity
uint[] numbers;
numbers.push(5);
numbers.push(10);
numbers.push(15);
// numbers is now equal to [5, 10, 15]
```

