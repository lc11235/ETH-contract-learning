> 通过前边的Solidity基础语法学习，我们已经有了Solidity编程经验， 在这节就要学习Ethereum开发的技术细节，编写真正的DApp时必须知道的：智能协议的所有权，Gas的花费，代码优化和代码安全。

## 一、智能协议的永固性

到现在为止，我们将Solidity和其他语言没有质的区别，它长得也很像JavaScript。

但是，在有几点以太坊上DApp跟普通的应用程序有着天壤之别。

第一个例子，在你把智能协议传上以太坊之后，它就变得不可更改，这种永固性意味着你的代码永远不能被调整或更新。

你编译的程序会一直，永久的，不可更改的，存在以太网上。这就是Solidity代码的安全性如此重要的一个原因。如果你的智能协议有任何漏洞，即使你发现了也无法补救。你只能让你的用户们放弃这个只能协议，然后转移到一个新的修复后的合约上。

但这恰好也是只能合约的一大优势。代码说明一切。如果去读智能合约的代码，并验证它，你会发现，一旦函数被定义下来，每一次的运行，程序都会严格按照函数中原有的代码逻辑一丝不苟地执行，完全不用担心函数被人篡改而得到意外的结果。

### 外部依赖关系

在上边的文章中，我们将加密小猫（CryptoKitties）合约的地址硬编码到DApp中去了。有没有想过，如果加密小猫出了点问题，比方说，集体消失了会怎么样？虽然这种事情几乎不可能发生，但是，如果小猫没了，我们的DApp也会随之失效——因为我们在DApp的代码中用“硬编码”的形式指定了加密小猫的地址，如果根据这个地址找不到小猫，我们的僵尸也就吃不到小猫了，而按照前面的描述，我们却没法修改合约去应付这个变化！

因为，我们不能硬编码，而要采用“函数”，以便于DApp的关键部分可以以参数形式修改。

比方说，我们不再一开始就把猎物地址给写入代码，而是写个函数setKittyContractAddress，运行时再设定猎物的地址，这样我们就可以随时地去锁定新的猎物，也不用担心加密小猫集体消失了。

#### 实战演练

请修改前边的代码，使得可以通过程序更改CryptoKitties合约地址。

* 1.删除采用硬编码方式的ckAddress代码行。
* 2.之前创建kittyContract变量的那行代码，修改为对kittyContract变量的声明——暂时不给它指定具体的实例。
* 3.创建名为setKittyContractAddress的函数，它带一个参数_address（address类型），可见性设为external。
* 4.在函数内部，添加一行代码，将kittyContract变量设置为返回值：KittyInterface(_address)。

> 注意：你可能会注意到这个功能有个安全漏洞，别担心——下一章里解决它。

zombiefeeding.sol

```solidity
pragma solidity ^0.4.19;

import "./zombiefactory.sol";

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

  // 1. 移除这一行:
  // address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;

  // 2. 只声明变量:
  // KittyInterface kittyContract = KittyInterface(ckAddress);
  KittyInterface kittyContract;

  // 3. 增加 setKittyContractAddress 方法
  function setKittyContractAddress(address _address) external {
    kittyContract = KittyInterface(_address);

  }

  function feedAndMultiply(uint _zombieId, uint _targetDna, string species) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    if (keccak256(species) == keccak256("kitty")) {
      newDna = newDna - newDna % 100 + 99;
    }
    _createZombie("NoName", newDna);
  }

  function feedOnKitty(uint _zombieId, uint _kittyId) public {
    uint kittyDna;
    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
    feedAndMultiply(_zombieId, kittyDna, "kitty");
  }

}
```

##二、Ownable Contracts

上面代码中，你有没有发现任何安全漏洞呢？

其实setKittyContractAddress可见性居然声明为“外部的”（external），岂不是任何人都可以调用它！也就是说，任何调用该函数的人都可以更改CryptoKitties合约的地址，使得其他人都没法再运行我们的程序了。

我们确实是希望这个地址能够在合约中修改，但我们可没说让每个人都能去改它。

> 要对付这样的情况，通常的做法是指定合约的“所有权”——就是说，给它指定一个主人（没错，就是你自己），只有主人对它享有特权。

#### Ownable

下面是一个Ownable合约的例子：来自OpenZeppelin Solidity库的Ownable合约。OpenZeppelin是主打安保和社区审查的智能合约库，您可以在自己的DApp中引用。

把下面这个合约通读，是不是还有些没见过的代码？别担心，我们随后会解释。

```solidity
/**
 * @title Ownable
 * @dev The Ownable contract has an owner address, and provides basic authorization control
 * functions, this simplifies the implementation of "user permissions".
 */
contract Ownable {
  address public owner;
  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

  /**
   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
   * account.
   */
  function Ownable() public {
    owner = msg.sender;
  }

  /**
   * @dev Throws if called by any account other than the owner.
   */
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }

  /**
   * @dev Allows the current owner to transfer control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0));
    OwnershipTransferred(owner, newOwner);
    owner = newOwner;
  }
}
```

下面有没有你没学过的东西？

* 构造函数：function Ownable()是一个constructor（构造函数），构造函数不是必须的，它与合约同名，构造函数一生中唯一的一次执行，就是在合约最初被创建的时候。
* 函数修饰符：modifier onlyOwner()。修饰符跟函数很类似，不过是用来修饰其他已有函数用的，在其他语句执行前，为它检查下先验条件。在这个例子中，我们就可以写个修饰符onlyOwner检查下调用者，确保只有合约的主人才能运行本函数。我们下一章会详细讲述修饰符，以及那个奇怪的下划线_。
* indexed关键字：别担心，我们还用不到它。

所以Ownable合约基本都会这么干：

* 1.合约创建，构造函数先行，将其owner设置为msg.sender（其部署者）
* 2.为它加上一个修饰符onlyOwner，它会限制陌生人的访问，将访问某些函数的权限锁定在owner上。
* 3.允许将合约所有权转染给他人。

onlyOwner简直人见人爱，大多数人开发自己的Solidity DApp，都是从复制粘贴Ownable开始的，从它再继承出的子类，并在其之上进行功能开发。

既然我们想把setKittyContractAddress限制为onlyOwner，我们也要做同样的事情。

#### 实战演练

首先，将Ownable合约的代码复制一份到新文件ownable.sol中。接下来，创建一个ZombieFactory，继承Ownable。

* 1.在程序中导入ownable.sol的内容。如果你不记得怎么做了，参考下zombiefeeding.sol。
* 2.修改ZombieFactory合约，让它继承Ownable。如果你不记得怎么做了，看看zombiefeeding.sol。

ownable.sol文件：

```solidity
/**
 * @title Ownable
 * @dev The Ownable contract has an owner address, and provides basic authorization control
 * functions, this simplifies the implementation of "user permissions".
 */
contract Ownable {
  address public owner;

  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

  /**
   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
   * account.
   */
  function Ownable() public {
    owner = msg.sender;
  }


  /**
   * @dev Throws if called by any account other than the owner.
   */
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }


  /**
   * @dev Allows the current owner to transfer control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0));
    OwnershipTransferred(owner, newOwner);
    owner = newOwner;
  }

}
```

zombiefactory.sol

```solidity
pragma solidity ^0.4.19;

// 1. 在这里导入
import "./ownable.sol";

// 2. 在这里继承:
contract ZombieFactory is Ownable{

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
        randDna = randDna - randDna % 100;
        _createZombie(_name, randDna);
    }

}
```

## 三、onlyOwner函数修饰符

现在我们有了这个基本版的合约ZombieFactory了，它继承自Ownable接口，我们也可以给ZombieFeeding加上onlyOwner函数修饰符。

这就是合约继承的工作原理。记得：

```solidity
ZombieFeeding 是个 ZombieFactory
ZombieFactory 是个 Ownable
```

#### 函数修饰符

函数修饰符看来跟函数没什么不同，不过关键字modifier告诉编译器，这是一个modifier（修饰符），而不是一个function（函数）。它不能像函数那样被直接调用，只能被添加到函数定义的末尾，用以改变函数的行为。

再仔细读读onlyOwner：

```solidity
/**
 * @dev 调用者不是‘主人’，就会抛出异常
 */
modifier onlyOwner() {
  require(msg.sender == owner);
  _;
}
```

onlyOwner函数修饰符是这么用的：

```solidity
contract MyContract is Ownable {
  event LaughManiacally(string laughter);

  //注意！ `onlyOwner`上场 :
  function likeABoss() external onlyOwner {
    LaughManiacally("Muahahahaha");
  }
}
```

注意likeABoss函数上的onlyOwner修饰符。当你调用likeABoss时，首先执行onlyOwner中的代码，执行到onlyOwner中的**_;**语句时，程序再返回并执行likeABoss中的代码。

可见，尽管函数修饰符也可以应用到各种场合，但最常见的还是放在函数执行之前添加快速的require检查。

因为给函数添加了修饰符onlyOwner，使得唯有合约的主人（也就是部署者）才能调用它。

> 注意：主人堆合约享有特权当然是正常的，不过也可能被恶意使用，比如，万一主人添加了个后门，允许他偷走别人的僵尸呢？
>
> 所以非常重要的是，部署在以太坊上的DApp，并不能保证它真正做到去中心化，你需要阅读并理解它的源代码，才能防止其中没有被部署者恶意植入后门；作为开发人员，如何做到既要给自己留下修复bug的余地，又要尽量地放权给使用者，以便让他们放心你，从而愿意把数据放在你的DApp中，这确实需要个微妙的平衡。

#### 实战演练

现在我们可以限制第三方对setKittyContractAddress的访问，除了我们自己，谁都无法去修改它。

* 1.将onlyOwner函数修饰符添加到setKittyContractAddress中。

zombiefeeding.sol

```solidity
pragma solidity ^0.4.19;

import "./zombiefactory.sol";

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

  KittyInterface kittyContract;

  // 修改这个函数,添加权限onlyOwner
  function setKittyContractAddress(address _address) external onlyOwner {
    kittyContract = KittyInterface(_address);
  }

  function feedAndMultiply(uint _zombieId, uint _targetDna, string species) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    if (keccak256(species) == keccak256("kitty")) {
      newDna = newDna - newDna % 100 + 99;
    }
    _createZombie("NoName", newDna);
  }

  function feedOnKitty(uint _zombieId, uint _kittyId) public {
    uint kittyDna;
    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
    feedAndMultiply(_zombieId, kittyDna, "kitty");
  }

}
```

## 四、Gas

现在我们懂了如何在禁止第三方修改我们的合约的同时，留个后门给咱们自己去修改。

让我们来看另一种使得Solidity编程语言与众不同的特征：

#### Gas——驱动以太坊DApps的能源

在Solidity中，你的用户想要每次执行你的DApp都需要支付一定的gas，gas可以用以太币购买，因此，用户每次跑DApp都得花费以太币。

一个DApp收取多少gas取决于功能逻辑的复杂程度。每个操作背后，都在计算完成这个操作所需要的计算资源（比如，存储数据就比做个加法运算贵的多），一次操作所需要花费的gas等于这个操作背后的所有运算花销的总和。

由于运行你的程序需要花费用户的真金白银，在以太坊中代码的编程语言，比其他任何编程语言都强调优化。同样的功能，使用笨拙的代码开发的程序，比起经过精巧优化的代码来说，运行花费更高，这显然会给成千上万的用户带来大量不必要的开销。

#### 为何要gas来驱动？

以太坊就像一个巨大、缓慢、但非常安全的电脑。当你运行一个程序的时候，网络上的每一个节点都在进行相同的运算，以验证它的输出——这就是所谓的“去中心化”，由于数以千计的节点同时在验证每个功能的运行，这可以确保它的数据不会被监控，或者被刻意修改，

可能会有拥护用无限循环堵塞网络，抑或用密集运算来占用大量的网络资源，为了防止这种事情的发生，以太坊的创建者为以太坊上资源制定了价格，想要在以太坊上运算或存储，你需要先付费。

> 注意：如果你使用侧链，倒是不一定需要付费，比如咱们先在Loom Network上构建的CryptoZombies就免费。你不会想在以太坊主网上玩“魔兽世界”吧？——所需要的gas可能会买到你破产。但是可以找个算法理念不同的侧链来玩它。我们将在以后的课程中讨论，什么样的DApp应该部署到以太坊主链上，什么又最好放在侧链上。

#### 省gas的招数

##### 省gas的招数：结构封装（Struct packing）

在第一章中，我们提到除了基本版的uint外，还有其他变种uint：uint8，uint16，uint32等。

通常情况下我们不会考虑使用uint变种，因为无论如何定义uint的大小，Solidity都为它保留了256位的存储空间。例如，使用uint8而不是uint（uint256）不会为你节省任何gas。

除非，把uint绑定到struct里面。

如果一个struct中有多个uint，则尽可能使用较小的uint，Solidity会将这些uint打包在一起，从而占用较少的存储空间。例如：

```solidity
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// 因为使用了结构打包，`mini` 比 `normal` 占用的空间更少
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30);
```

所以，当 `uint` 定义在一个 `struct` 中的时候，尽量使用最小的整数子类型以节约空间。 并且把同样类型的变量放一起（即在 struct 中将把变量按照类型依次放置），这样 Solidity 可以将存储空间最小化。例如，有两个 `struct`：

uint c; uint32 a; uint32 b;` 和 `uint32 a; uint c; uint32 b;

前者比后者需要的gas更少，因为前者把`uint32`放一起了。

#### 实战演练



