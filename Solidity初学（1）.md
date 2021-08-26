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

## 三、合约设计

我们使用[Solidity]([Truffle | Overview | Documentation | Truffle Suite](https://www.trufflesuite.com/docs/truffle/overview))来编写合约。如果你熟悉面向对象的开发和JavaScript，那么学习Solidity应该非常简单。可以将合约类比于OOP的类：合约中的属性用来声明合约的状态，而合约中的方法则提供修改状态的访问接口。

重点：

* 合约状态是持久化到区块链上的，因此对合约状态的修改需要消耗以太币。
* 只有在合约部署到区块链的时候，才会调用构造函数，并且只调用一次。
* 与web世界里每次部署代码都会覆盖旧代码不同，在区块链上部署的合约是不可改变的，也就是说，如果你更新合约并再次部署，旧的合约仍然会在区块链上存在，并且合约的状态数据也依然存在。新的部署将会创建合约的一个新的实例。

## 四、合约语法

从最基本的入手：

Solidity的代码都包括在合约里面，一份合约就是以太币应用的基本模块，所有的变量和函数属于一份合约，它是你所有应用的起点。

一份名为HelloWorld的空合约如下：

```solidity
contract HelloWorld {

}
```

### 1、版本指令

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

#### 实战演习1：

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

### 2、状态变量和整数

状态变量是被永久地保存在合约中。也就是说它们被写入以太币区块链中，想象成写入到一个数据库。

示例：

```solidity
contract Example {
	// 这个无符号整数将会永久的被保存在区块链中
	uint myUnsignedInteger = 100;
}
```

在上面的例子中，定义myUnsignedInteger为uint类型，并赋值100.

#### 无符号整数：uint

uint无符号数据类型，指其值不能是负数，对于有符号的整数存在名为int的数据类型。

> 注：Solidity中，unit实际上是uint256代名词，一个256位的无符号整数。你也可以定位位数少的units——uint8，uint16，uint32，等.....但一般来讲你更愿意使用简单的uint，除非在某些特殊情况下，这我们后面会讲。

#### 实战演练2

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

### 3、数学运算

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

#### 实战演练3

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

### 4、结构体

有时你需要更复杂的数据类型，Solidity提供了结构体：

```solidity
struct Person {
	uint age;
	string name;
}
```

结构体允许你生成一个更复杂的数据类型，它有多个属性。

>注：我们刚刚引进了一个新类型，string。字符串用于保存任意长度的UTF-8编码数据。如：string greeting  = "Hello World!"。

#### 实战演练4

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

### 5、数组

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

#### 公共数组

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

### 6、定义函数

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

#### 实战演练6

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

### 7、使用结构体和数组

#### 创建新的结构体

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

#### 实战演练7

让我们创建名为createZombie的函数来做点什么吧。

* 1.在函数体里新创建一个Zombie，然后把它加入zombies数组中。新创建的僵尸的name和dna，来自于函数的参数。
* 2.让我们用一行代码简洁地完成。

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
         // 8.使用结构体和数组（初始化全局数组）
        zombies.push(Zombie(_name, _dna));
    }
}
```

### 8、私有/公共函数

Solidity定义的函数的属性默认为公共。这就意味着任何一方（或其他合约）都可以调用你合约里的函数。

显然，不是什么时候都需要这样，而且这样的合约易于收到攻击。所以将自己的函数定义为私有是一个好的编程习惯，只有当你需要外部世界调用它时才将它设置为公共。

如何定义一个私有的函数呢？

```solidity
uint[] numbers;
function _addToArray(uint _number) private {
  numbers.push(_number);
}
```

这意味着只有我们合约中的其他函数才能调用这个函数，给numbers数组添加新成员。

可以看到，在函数名字后面使用关键字private即可。和函数的参数类似，私有函数的名字用下划线起始。

#### 实战演练8

我们合约函数createZombie的默认属性时公共的，这意味着任何一方都可以调用它去创建一个僵尸。咱们来把它变成私有吧！

* 1.变createZombie为私有函数，不要忘记遵守命名的规矩哦

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
    
    /*
    // 7.创建函数
    function createZombie(string _name, uint _dna){
         // 8.使用结构体和数组（初始化全局数组）
        zombies.push(Zombie(_name, _dna));
    }
    */
    
     // 7.创建函数(改为私有方法）
    function _createZombie(string _name, uint _dna) private {
         // 8.使用结构体和数组（初始化全局数组）
        zombies.push(Zombie(_name, _dna));
    }
    
   
}
```

### 9、函数的更多属性

本节中我们将学习函数的返回值和修饰符。

#### 返回值

要想函数返回一个数值，按如下定义：

```solidity
string greeting = "What's up dog";

function sayHello() public returns (string) {
  return greeting;
}
```

Solidity里，函数的定义里可包含返回值的数据类型（如本例中string）。

#### 函数的修饰符view，returns

上面的函数实际上没有改变Solidity里的状态，即，它没有改变任何值或者写任何东西。这种情况下我们可以把函数定义为view，意味着它只能读取数据不能更改数据：

```solidity
function sayHello() public view returns (string) {}
```

Solidity还支持pure函数，表明这个函数甚至都不访问应用里的数据，例如：

```solidity
function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}
```

这个函数甚至都不读取应用的状态——它的返回值完全取决于它的输入参数，在这种情况下我们把函数定义为pure。

> 注：可能很难记住何时把函数标记为pure/view。幸运的是，Solidity编辑器会给出提示，提醒你使用这些修饰符。

#### 实战演练9

我们想建立一个帮助函数，它根据一个字符串随机生成一个DNA数据。

* 1.创建一个private函数，命名为_generateRandomDna。它只接受一个输入变量\_str（类型string），返回一个unit类型的数值。
* 2.此函数只读取我们合约中的一些变量，所以标记为view。
* 3.函数内部暂时留空，以后我们再添加代码。

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
    
    /*
    // 7.创建函数
    function createZombie(string _name, uint _dna){
         // 8.使用结构体和数组（初始化全局数组）
        zombies.push(Zombie(_name, _dna));
    }
    */
    
     // 7.创建函数(改为私有方法）
    function _createZombie(string _name, uint _dna) private {
         // 8.使用结构体和数组（初始化全局数组）
        zombies.push(Zombie(_name, _dna));
    }
    
    // 9.函数修饰符 private, view, returns 返回值
    function _generateRandomDna(string _str) private view returns (uint){
        
      }
    
  
}
```

### 10、Keccak256和类型转换

#### 散列函数Keccak256

如何让_generateRandomDna函数返回一个全（半）随机的uint？

Ethereum内部有一个 散列函数keccak256，它用了SHA3版本。一个散列函数基本上就是把一个字符串转换为一个256位的16进制数字。字符串的一个微小变化会引起散列数据极大变化。

这在Ethereum中有很多应用，但是现在我们只是用它造一个伪随机数。

示例：

```solidity
//6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5
keccak256("aaaab");
//b1f078126895a1424524de5321b339ab00408010b7cf0e6ed451514981e58aa9
keccak256("aaaac");
```

显而易见，输入字符串只改变了一个字母，输出就已经天壤之别了。

> 注：在区块链中安全地产生一个随机数是一个很难的问题，本例的方法不安全，但是在我们的Zombie DNA算法里不是那么重要，已经很好地满足我们的需要了。

#### 类型转换

有时你需要变换数据类型。例如：

```solidity
uint8 a = 5;
uint b = 6;
// 将会抛出错误，因为 a * b 返回 uint, 而不是 uint8:
uint8 c = a * b;
// 我们需要将 b 转换为 uint8:
uint8 c = a * uint8(b);
```

> 上面，a * b返回类型是uint，但是当我们尝试用uint8类型接收时，就会造成潜在的错误。如果把它的数据类型转换为uint8，就可以了，编译器也不会出错。

#### 实战演练10

给_generateRandomDna函数添加代码!它应该完成如下功能：

* 1.第一行代码获取_str的keccak256散列值生成一个伪随机十六位进制数，类型转换为uint，最后保存在类型为unit名为rand的变量中。
* 2.我们只想让我们的DNA的长度为16位（还记得dnaModulus？）。所以第二行代码应该return上面计算的数值对dnaModulus求余数（%）。

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
    
    /*
    // 7.创建函数
    function createZombie(string _name, uint _dna){
         // 8.使用结构体和数组（初始化全局数组）
        zombies.push(Zombie(_name, _dna));
    }
    */
    
     // 7.创建函数(改为私有方法）
    function _createZombie(string _name, uint _dna) private {
         // 8.使用结构体和数组（初始化全局数组）
        zombies.push(Zombie(_name, _dna));
    }
    
    // 9.函数修饰符 private, view, returns 返回值
    function _generateRandomDna(string _str) private view returns (uint){
        // 10.散列并取模
        uint rand = uint(keccak256(_str));  // 注意这里需要将string类型转为uint类型
        return rand % dnaModulus;
    }
        
      }

}
```

### 11、综合应用

我们就快完成我们的随机僵尸制造器了，来写一个公共的函数把所有的部件连接起来。

写一个公共函数，它有一个参数，用来接收僵尸的名字，之后用它生成僵尸的DNA。

#### 实战演练11

* 1.创建一个public函数，命名为createRandomZombie，它将被传入一个变量_name(数据类型是string)。（注：定义公共函数public和定义一个私有private函数的做法一样）。
* 2.函数的第一行应该调用_generateRandomDna函数，传入\_name参数，结果保存在一个类型为uint的变量里，命名为randDna。
* 3.第二行调用_createZombie函数，传入参数\_name和randDna。
* 4.整个函数应该是4行代码（包括函数的结束符号}）。

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
    
    /*
    // 7.创建函数
    function createZombie(string _name, uint _dna){
         // 8.使用结构体和数组（初始化全局数组）
        zombies.push(Zombie(_name, _dna));
    }
    */
    
     // 7.创建函数(改为私有方法）
    function _createZombie(string _name, uint _dna) private {
         // 8.使用结构体和数组（初始化全局数组）
        zombies.push(Zombie(_name, _dna));
    }
    
    // 9.函数修饰符 private, view, returns 返回值
    function _generateRandomDna(string _str) private view returns (uint){
        // 10.散列并取模
        uint rand = uint(keccak256(_str));  // 注意这里需要将string类型转为uint类型
        return rand % dnaModulus;
    }
        
     
     // 11、事件
    function createRandomZombie(string _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```

### 12、事件

我们的合约几乎就要完成了！让我们加上一个事件。

事件是合约和区块链通讯的一种机制。你的前端应用“监听”某些事件，并做出反应。

示例：

```solidity
// 这里建立事件
event IntegersAdded(uint x, uint y, uint result);

function add(uint _x, uint _y) public {
  uint result = _x + _y;
  //触发事件，通知app
  IntegersAdded(_x, _y, result);
  return result;
}
```

你的app前端可以监听整个事件。JavaScript实现如下：

```javascript
YourContract.IntegersAdded(function(error, result) { 
  // 干些事
}
```

#### 实战演练12

我们想每当一个僵尸创造出来时，我们的前端都能监听到这个事件，并将它显示出来。

* 1.定义一个事件叫做NewZombie。它有3个参数：zombieId（uint），name（string），和dna（uint）。
* 2.修改_createZombie函数使得当新僵尸造出来并加入zombies数组后，生成事件NewZombie。
* 3.需要定义僵尸id。array.push()返回数组的长度类型是uint——因为数组的第一个元素的索引是0，array.push() - 1将是我们加入的僵尸的索引。zombies.push() - 1就是id，数据类型是uint。在下一行中你可以把它用到NewZombie事件中。

Contract.sol

```solidity
// 1. 这里写版本指令
pragma solidity ^0.4.19; 

// 2. 这里建立智能合约
contract ZombieFactory {

  // 12.这里建立事件
  event NewZombie(uint zombieId, string name, uint dna);

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
    
    /*
    // 7.创建函数
    function createZombie(string _name, uint _dna){
         // 8.使用结构体和数组（初始化全局数组）
        zombies.push(Zombie(_name, _dna));
    }
    */
    
     // 7.创建函数(改为私有方法）
    function _createZombie(string _name, uint _dna) private {
         // 8.使用结构体和数组（初始化全局数组）
         // zombies.push(Zombie(_name, _dna));
         
        // 12、数组长度减一就是当前的数组ID
        uint id = zombies.push(Zombie(_name, _dna)) - 1;

        // 12、这里触发事件
        NewZombie(id, _name, _dna);
    }
    
    // 9.函数修饰符 private, view, returns 返回值
    function _generateRandomDna(string _str) private view returns (uint){
        // 10.散列并取模
        uint rand = uint(keccak256(_str));  // 注意这里需要将string类型转为uint类型
        return rand % dnaModulus;
    }
        
     
     // 11、综合函数
    function createRandomZombie(string _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```

## 五、Web3.js

我们的Solidity合约完工了！现在我们要写一段JavaScript前端代码来调用这个合约。

以太坊有一个JavaScript库，名为Web3.js。

在后面的文章里，我们会进一步地教你如何安装一个合约，如何设置Web3.js。但是现在我们通过一段代码来了解Web3.js是如何和我们发布的合约交互的吧。

如果下面的代码你不能完全理解，不用担心。

```javascript
// 下面是调用合约的方式:
var abi = /* abi是由编译器生成的 */
var ZombieFactoryContract = web3.eth.contract(abi)
var contractAddress = /* 发布之后在以太坊上生成的合约地址 */
var ZombieFactory = ZombieFactoryContract.at(contractAddress)
// `ZombieFactory` 能访问公共的函数以及事件

// 某个监听文本输入的监听器:
$("#ourButton").click(function(e) {
  var name = $("#nameInput").val()
  //调用合约的 `createRandomZombie` 函数:
  ZombieFactory.createRandomZombie(name)
})

// 监听 `NewZombie` 事件, 并且更新UI
var event = ZombieFactory.NewZombie(function(error, result) {
  if (error) return
  generateZombie(result.zombieId, result.name, result.dna)
})

// 获取 Zombie 的 dna, 更新图像
function generateZombie(id, name, dna) {
  let dnaStr = String(dna)
  // 如果dna少于16位,在它前面用0补上
  while (dnaStr.length < 16)
    dnaStr = "0" + dnaStr

  let zombieDetails = {
    // 前两位数构成头部.我们可能有7种头部, 所以 % 7
    // 得到的数在0-6,再加上1,数的范围变成1-7
    // 通过这样计算：
    headChoice: dnaStr.substring(0, 2) % 7 + 1，
    // 我们得到的图片名称从head1.png 到 head7.png

    // 接下来的两位数构成眼睛, 眼睛变化就对11取模:
    eyeChoice: dnaStr.substring(2, 4) % 11 + 1,
    // 再接下来的两位数构成衣服，衣服变化就对6取模:
    shirtChoice: dnaStr.substring(4, 6) % 6 + 1,
    //最后6位控制颜色. 用css选择器: hue-rotate来更新
    // 360度:
    skinColorChoice: parseInt(dnaStr.substring(6, 8) / 100 * 360),
    eyeColorChoice: parseInt(dnaStr.substring(8, 10) / 100 * 360),
    clothesColorChoice: parseInt(dnaStr.substring(10, 12) / 100 * 360),
    zombieName: name,
    zombieDescription: "A Level 1 CryptoZombie",
  }
  return zombieDetails
}
```

我们的JavaScript所做的就是获取由zombieDetails产生的数据，并且利用浏览器里的JavaScript神奇功能（我们用Vue.js），置换出图像以及使用CSS过滤器。在后面的文章中，你可以看到全部的代码。

## 六、Truffle框架学习

Truffle是一个DApp开发框架，它简化了去中心化应用的构建和管理。 
