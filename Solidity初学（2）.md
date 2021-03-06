## 一、映射（Mapping）和地址（Address）

我们通过给数据库中的僵尸指定“主人”，来支持“多玩家”模式。

如此依赖，我们需要引入2个新的数据类型：mapping（映射）和address（地址）。

### Address（地址）

以太坊区块链由account（账户）组成，你可以把它想象成银行账户。一个账户的余额是以太（在以太坊区块链上使用的币种），你可以和其他账户之间支付和接受以太币，就像你的银行账户可以电汇资金刀其他银行账户一样。

每个账户都有一个“地址”，你可以把它想象成银行账号。这是账户唯一的标识符，它看起来长这样：

```solidity
0x0cE446255506E92DF41614C46F1d6df9Cc969183
```

我们将在后面的课程中介绍地址的细节，现在你只需要了解地址属于特定用户（或智能合约）的。

所以我们可以指定“地址”作为僵尸主人的ID。当用户通过与我们的应用程序交互来创建新的僵尸时，新僵尸的所有权被设置到调用者的以太坊地址下。

### Mapping （映射）

在上一篇博文中，我们看到了结构体和数组。映射是另一种在Solidity中存储有组织数据的方法。

映射是这样定义的：

```solidity
//对于金融应用程序，将用户的余额保存在一个 uint类型的变量中：
mapping (address => uint) public accountBalance;

//或者可以用来通过userId 存储/查找的用户名
mapping (uint => string) userIdToName;
```

映射本质上是存储和查找数据所用的键-值对。在第一个例子中，键是一个address，值是一个uint，在第二个例子中，键是一个uint，值是一个string。

#### 实战演练

为了存储僵尸的所有权，我们会使用到两个映射：一个记录僵尸拥有者的地址，另一个记录某地址所拥有僵尸的数量。

* 1.创建一个叫做zombieToOwner的映射。其键是一个uint（我们将根据他的id存储和查找僵尸），值为address。映射属性为public。
* 2.创建一个名为ownerZombieCount的映射，其中键是address，值是uint。

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
    
    // 13.在这里定义映射
    mapping(uint => address) public zombieToOwner;
    mapping(address => uint) ownerZombieCount;
    
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

## 二、Msg.sender

现在有了一套映射来记录僵尸的所有权了，我们可以修改_createZombie方法来运用它们。

为了做到这一点，我们要用到msg.sender。

### msg.sender

在Solidity中，有一些全局变量可以被所有函数调用。其中一个就是msg.sender，它指的是当前调用者（或智能合约）的address。

> 注意：在Solidity中，功能执行始终需要从外部调用者开始。一个合约只会在区块链上什么也不做，除非有人调用其中的函数。所以msg.sender总是存在的。

以下是使用msg.sender来更新mapping的例子：

```solidity
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  // 更新我们的 `favoriteNumber` 映射来将 `_myNumber`存储在 `msg.sender`名下
  favoriteNumber[msg.sender] = _myNumber;
  // 存储数据至映射的方法和将数据存储在数组相似
}

function whatIsMyNumber() public view returns (uint) {
  // 拿到存储在调用者地址名下的值
  // 若调用者还没调用 setMyNumber， 则值为 `0`
  return favoriteNumber[msg.sender];
}
```

在这个小小的例子中，任何人都可以调用setMyNumber在我们的合约中存下一个uint并且与他们的地址相绑定。然后，他们调用whatIsMyNumber就会返回他们存储的uint。

使用msg.sender很安全，因为它具有以太坊区块链的安全保障——除非窃取与以太坊地址相关联的私钥，否则是没有办法修改其他人的数据的。

#### 实战演练

我们来修改前边的_createZombie方法，将僵尸分配给函数调用者吧。

* 1.首先，在得到新的僵尸id后，更新zombieToOwner映射，在id下面存入msg.sender。
* 2.然后，我们为这个msg.sender名下的ownerZombieCount加1.

跟在JavaScript中一样，在Solidity中你也可以用++使uint递增。

```solidity
uint number = 0;
number++;
// `number` 现在是 `1`了
```

修改两个代码即可。

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    mapping (uint => address) public zombieToOwner;
    mapping (address => uint) ownerZombieCount;

    function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        // 从这里开始,msg.sender表示当前调用者的地址
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender] ++;
        NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }

    function createRandomZombie(string _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```

## 三、Require

我们成功让用户通过调用createRandomZombie函数并输入一个名字来创建新的僵尸。但是，如果用户能持续调用这个函数来创建无限多个僵尸加入他们的军团，这游戏就太没意思了。

于是，我们作出限定：每个玩家只能调用一次这个函数。这样一来，新玩家可以在刚开始玩游戏时通过调用它，为其军团创建初始僵尸。

**我们怎样才能限定每个玩家只调用一次这个函数呢？**

答案是使用require。require使得函数在执行过程中，当不满足某些条件时抛出错误，并停止执行：

```solidity
function sayHiToVitalik(string _name) public returns (string) {
  // 比较 _name 是否等于 "Vitalik". 如果不成立，抛出异常并终止程序
  // (敲黑板: Solidity 并不支持原生的字符串比较, 我们只能通过比较
  // 两字符串的 keccak256 哈希值来进行判断)
  require(keccak256(_name) == keccak256("Vitalik"));
  // 如果返回 true, 运行如下语句
  return "Hi!";
}
```

如果你这样调用函数sayHiToVitalik（“Vitalik”），它会返回“Hi！”。而如果调用的时候使用了其他参数，它则会抛出错误并停止执行。

因此，在调用一个函数之前，用require验证前置条件是非常有必要的。

#### 实战演练

在我们的僵尸游戏中，我们不希望用户通过反复调用createRandomZombie来给他们的军队创建无限多个僵尸——这将使得游戏非常无聊。

我们使用了require来确保这个函数只有在每个用户第一次调用它的时候执行，用以创建初始僵尸。

* 1.在createRandomZombie的前面放置require语句。使得函数先检查ownerZombieCount[msg.sender]的值为0，否则就抛出一个错误。

> 注意：在Solidity中，关键词放置的顺序并不重要

* 虽然参数的两个位置是等效的。但是，由于我们的答案检查器比较呆板，它只能认定其中一个为正确答案。
* 于是在这里，我们就约定把ownerZombieCount[msg.sender]放前面吧。

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    mapping (uint => address) public zombieToOwner;
    mapping (address => uint) ownerZombieCount;

    function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }

    function createRandomZombie(string _name) public {
        
        // require 判断
        require(ownerZombieCount[msg.sender] == 0);

        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```

## 四、继承inheritance

我们的游戏代码越来越长。当代码过于冗长的时候，最好将代码和逻辑分拆到多个不同的合约中，以便于管理。

有个让Solidity的代码易于管理的功能，就是合约inheritance（继承）：

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

由于BabyDoge是从Doge那里inherits（继承）过来的。这意味着当你编译和部署了BabyDoge，它将可以访问catchphrase()和anotherCatchphrase()和其他我们在Doge中定义的其他公共函数。

这可以用于逻辑继承（比如表达子类的时候，Cat是一种Animal）。但也可以简单地将类似的逻辑组合到不同的合约中以组织代码。

#### 实战演练

在接下来的章节中，我们将要为僵尸实现各种功能，让它可以“猎食”和“繁殖”。通过将这些运算放到父类ZombieFactory中，使得所有ZombieFactory的继承者合约都可以使用这些方法。

在ZombieFactory下创建一个叫ZombieFeeding的合约，它是继承自ZombieFactory合约的。

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    mapping (uint => address) public zombieToOwner;
    mapping (address => uint) ownerZombieCount;

    function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }

    function createRandomZombie(string _name) public {
        require(ownerZombieCount[msg.sender] == 0);
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}

// Start here (合约继承）
contract ZombieFeeding is ZombieFactory {

}
```

## 五、引入Import

在这一节中，我们将对上边那个很大的合约进行拆分。

上边的代码已经够长了，我们把它分成多个文件以便于管理。通常情况下，当Solidity项目中的代码太长的时候我们就是这么做的。

在Solidity中，当你有多个文件并且想把文件导入另一个文件时，可以使用import语句。

```solidity
import "./someothercontract.sol";

contract newContract is SomeOtherContract {

}
```

这样当我们在合约（contract）目录下有一个名为someothercontract.sol的文件（./就是同一目录的意思），它就会被编译器导入。

#### 实战演练

现在我们已经建立了一个多文件架构，并用import来读取来自另一个文件中合约的内容：

* 1.将zombiefactory.sol导入到我们的新文件zombiefeeding.sol中。

zombiefactory.sol

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    mapping (uint => address) public zombieToOwner;
    mapping (address => uint) ownerZombieCount;

    function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }

    function createRandomZombie(string _name) public {
        require(ownerZombieCount[msg.sender] == 0);
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```

zombiefeeding.sol

```solidity
pragma solidity ^0.4.19;

// put import statement here(导入合约）
// import './zombiefactory.sol';  // 导入另一个文件不能用单引号，只能用双引号，否则会报错
import "./zombiefactory.sol";

contract ZombieFeeding is ZombieFactory {

}
```

## 六、Storage与Memory

在Solidity中，有两个地方可以存储变量——storage或memory。

Storage变量是指永久存储在区块链中的变量。Memory变量则是临时的，当外部函数对某合约调用完成时，内存型变量即被移除。你可以把它想象成存储在你电脑的硬盘或是RAM中数据的关系。

大多数时候你都用不到这些关键字，默认情况下Solidity会自动处理它们。状态变量（在函数之外声明的变量）默认为“存储”形式，并永久写入区块链；而在函数内部声明的变量是“内存”型的，它们函数调用结束后就消失。

然而也有一些情况下，你需要手动声明存储类型，主要用于处理函数内的结构体和数组时：

```solidity
contract SandwichFactory {
  struct Sandwich {
    string name;
    string status;
  }

  Sandwich[] sandwiches;

  function eatSandwich(uint _index) public {
    // Sandwich mySandwich = sandwiches[_index];

    // ^ 看上去很直接，不过 Solidity 将会给出警告
    // 告诉你应该明确在这里定义 `storage` 或者 `memory`。

    // 所以你应该明确定义 `storage`:
    Sandwich storage mySandwich = sandwiches[_index];
    // ...这样 `mySandwich` 是指向 `sandwiches[_index]`的指针
    // 在存储里，另外...
    mySandwich.status = "Eaten!";
    // ...这将永久把 `sandwiches[_index]` 变为区块链上的存储

    // 如果你只想要一个副本，可以使用`memory`:
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // ...这样 `anotherSandwich` 就仅仅是一个内存里的副本了
    // 另外
    anotherSandwich.status = "Eaten!";
    // ...将仅仅修改临时变量，对 `sandwiches[_index + 1]` 没有任何影响
    // 不过你可以这样做:
    sandwiches[_index + 1] = anotherSandwich;
    // ...如果你想把副本的改动保存回区块链存储
  }
}
```

如果你还没有完全理解究竟应该使用哪一个，也不用担心——在本教程中，我们将告诉你何时使用storage或是memory，并且当你不得不使用到这些关键字的时候，Solidity编译器也会发出警告提醒你的。

现在，只需要知道在某些场合下也需要你显式地声明storage或memory就够了！

#### 实战演练

是时候给我们的僵尸增加“猎食”和“繁殖”功能了！

当一个僵尸猎食其他生物体时，它自身的DNA将与猎物生物的DNA结合在一起，形成一个新的僵尸DNA。

* 1.创建一个名为feedAndMultiply的函数。使用两个参数：_zombieId（uint类型）和\_targetDna（也是uint类型）。设置属性为public的。
* 2.我们不希望别人用我们的僵尸去捕猎。首先，我们确保对自己僵尸的所有权。通过添加一个require语句来确保msg.sender只能是这个僵尸的主人（类似于我们在createRandomZombie函数中做过的那样）。

> 注意：同样，因为我们的答案检查器比较呆萌，只认识把msg.sender放在前面的答案，如果你切换了参数的顺序，它就不认得了。但你正常编码时，如何安排参数顺序都是正确的。

* 3.为了获取这个僵尸的DNA，我们的函数需要声明一个名为myZombie数据类型为Zombie的本地变量（这是一个storage型的指针）。将其值设定为在zombies数组中索引为_zombieId所指向的值。

到目前为止，包括函数结束符}的那一行，总共4行代码。

zombiefeeding.sol

```solidity
pragma solidity ^0.4.19;

import "./zombiefactory.sol";

contract ZombieFeeding is ZombieFactory {

  // Start here
  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
      require(msg.sender == zombieToOwner[_zombieId]);
      Zombie storage myZombie = zombies[_zombieId];
  }

}
```

## 七、实战应用

我们来把feedAndMultiply函数写完吧。

获取新的僵尸DNA的公式很简单：计算猎食僵尸DNA和被猎僵尸DNA之间的平均值。

示例：

```solidity
function testDnaSplicing() public {
  uint zombieDna = 2222222222222222;
  uint targetDna = 4444444444444444;
  uint newZombieDna = (zombieDna + targetDna) / 2;
  // newZombieDna 将等于 3333333333333333
}
```

以后，我们也可以让函数变得复杂些，比方给新的僵尸的DNA增加一些随机性之类的。但现在先从最简单的开始——以后还可以回来完善它。

#### 实战演练

* 1.首先我们确保_targetDna不长于16位。要做到这一点，我们可以设置\_targetDna为\_targetDna%dnaModulus，并且只取其最后16位数字。
* 2.接下来为我们的函数声明一个名叫newDna的uint类型的变量，并将其设置为myZombie的DNA和_targetDna的平均值（如上例所示）。

> 注意：您可以用myZombie.name或myZombie.dna访问myZombie的属性。

* 一旦我们计算出新的DNA，再调用_createZombie就可以生成新的僵尸了。如果你忘了调用这个函数所需要的参数，可以查看zombiefactory.sol选项卡。请注意，需要先给它命名，所以现在我们把新的僵尸的名字设为NoName——我们回头可以编写一个函数来更改僵尸的名字。

> 注意：对于Solidity高手，你可能会注意到我们的代码存在一个问题。别担心，下一章会解决这个问题的。

zombiefeeding.sol

```solidity
pragma solidity ^0.4.19;

import "./zombiefactory.sol";

contract ZombieFeeding is ZombieFactory {

  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    // start here

    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;

    _createZombie("NoName", newDna);

  }

}
```

## 八、函数可见性

我们上面的代码有问题！

编译的时候编译器就会报错。

错误在于，我们尝试从ZombieFeeding中调用_createZombie函数，但\_createZombie却是ZombieFactory的private函数。这意味着任何继承自ZombieFactory的子合约都不能访问它。

#### internal和external

除public和private属性之外，Solidity还使用了另外两个描述函数可见性的修饰词：internal（内部）和external（外部）。

internal和private类似，不过，如果某个合约继承自其父合约，这个合约即可以访问父合约中定义的“内部”函数。

external与public类似，只不过这些函数只能在合约之外调用——它们不能被合约内的其他函数调用。稍后我们将讨论什么时候使用external和public。

声明函数internal或external类型的语法，与声明private和public类型相同。

```solidity
contract Sandwich {
  uint private sandwichesEaten = 0;

  function eat() internal {
    sandwichesEaten++;
  }
}

contract BLT is Sandwich {
  uint private baconSandwichesEaten = 0;

  function eatWithBacon() public returns (string) {
    baconSandwichesEaten++;
    // 因为eat() 是internal 的，所以我们能在这里调用
    eat();
  }
}
```

#### 实战演练

* 将_createZombie()函数的属性从private改为internal，使得其他的合约也能访问到它。

我们已经成功把你的注意力集中到zombiefactory.sol这个基类合约上了。

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    mapping (uint => address) public zombieToOwner;
    mapping (address => uint) ownerZombieCount;

    // 在这里修改函数的功能 private => internal
    function _createZombie(string _name, uint _dna) internal {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }

    function createRandomZombie(string _name) public {
        require(ownerZombieCount[msg.sender] == 0);
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```

## 九、与其他合约的交互

是时候让我们的僵尸去捕猎了！那僵尸最喜欢的食物时什么呢？

为了做到这一点，我们要读出CryptoKitties智能合约中的kittyDna。这些数据时公开存储在区块链上的。

别担心——我们的游戏并不会伤害到任何真正的CryptoKitty。我们只读取CryptoKitties数据，但却无法在物理上删除它。

#### 与其他合约的交互

如果我们的合约需要和区块链上的其他的合约回话，则需要想定义一个interface（接口）。

先举一个简单的例子。假设区块链上有这么一个合约：

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

这是个很简单的合约，您可以用它存储自己的幸运号码，并将其与您的以太坊地址关联。这样其他人就可以通过您的地址查找您的幸运号码了。

现在假设我们有一个外部合约，使用getNum函数可读取其中的数据。

首先，我们定义LuckyNumber合约的interface：

```solidity
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

请注意，这个过程虽然看起来像在定义一个合约，但其实内里不同：

首先，我们只声明了要与之交互的函数——在本例中为getNum——在其中我们没有使用任何其他的函数或状态变量。

其次，我们并没有使用大括号（{和}）定义函数体，我们单单用分号（;）结束了函数声明。这使它起来像一个合约框架。

编译器就是靠这些特征认出它是一个接口的。

在我们的app代码中使用这个接口，合约就知道其他合约时怎样的，应该如何调用，以及可期待什么类型的返回值。

在下一章中，我们将真正调用其他合约的函数。目前我们只要声明一个接口，用于调用CryptoKitties合约就行了。

#### 实战演练

我们已经为你查看过了CryptoKitties的源代码，并且找到了一个名为getKitty的函数，它返回所有的加密猫的数据，包括它的“基因”（我们的僵尸游戏要用它来生成新的僵尸）。

该函数如下所示：

```solidity
function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
) {
    Kitty storage kit = kitties[_id];

    // if this variable is 0 then it's not gestating
    isGestating = (kit.siringWithId != 0);
    isReady = (kit.cooldownEndBlock <= block.number);
    cooldownIndex = uint256(kit.cooldownIndex);
    nextActionAt = uint256(kit.cooldownEndBlock);
    siringWithId = uint256(kit.siringWithId);
    birthTime = uint256(kit.birthTime);
    matronId = uint256(kit.matronId);
    sireId = uint256(kit.sireId);
    generation = uint256(kit.generation);
    genes = kit.genes;
}
```

这个函数看起来跟我们习惯的函数不太一样。它竟然返回了一堆不同的值！如果你用过JavaScript之类的编程语言，一定会感到奇怪——在Solidity中，您可以让一个函数返回多个值。

现在我们知道这个函数长什么样的了，就可以用它来创建一个接口：

* 1.定义一个名为KittyInterface的接口。请注意，因为我们使用了contract关键字，这过程看起来就像创建一个新的合约一样。
* 2.在interface里定义了getKitty函数（不过时复制/粘贴上面的函数，但在returns语言之后用分号，而不是大括号内的所有内容）。

zombiefeeding.sol

```solidity
pragma solidity ^0.4.19;

import "./zombiefactory.sol";

// Create KittyInterface here
contract KittyInterface {
    function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
    );
}

contract ZombieFeeding is ZombieFactory {

  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    _createZombie("NoName", newDna);
  }

}
```

