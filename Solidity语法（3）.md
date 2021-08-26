## 一、使用接口

继续前面上一节NumberInterface的例子，我们既然将接口定义为：

```solidity
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

我们可以在合约中这样使用：

```solidity
contract MyContract {
  address NumberInterfaceAddress = 0xab38...;
  // ^ 这是FavoriteNumber合约在以太坊上的地址
  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);
  // 现在变量 `numberContract` 指向另一个合约对象

  function someFunction() public {
    // 现在我们可以调用在那个合约中声明的 `getNum`函数:
    uint num = numberContract.getNum(msg.sender);
    // ...在这儿使用 `num`变量做些什么
  }
}
```

通过这种方式，只要将你合约的可见性设置为public（公共）或external（外部），它们就可以与以太坊区块链上的任何其他合约进行交互。

#### 实战演练

我们来建个自己的合约去读取另一个只能合约——CryptoKitties的内容吧！

* 1.我已经将代码中CryptoKitties合约的地址保存在一个名为ckAddress的变量中。在下一行中，请创建一个名为kittyContract的KittyInterface，并用ckAddress为它初始化——就像我们为numberContract所做的一样。

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

  address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
  // Initialize kittyContract here using `ckAddress` from above
  KittyInterface kittyContract = KittyInterface(ckAddress);

  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    _createZombie("NoName", newDna);
  }

}
```

## 二、处理多返回值

getKitty是我们所看到的的第一个返回多个值的函数。我们来看看是如何处理的：

```solidity
unction multipleReturns() internal returns(uint a, uint b, uint c) {
  return (1, 2, 3);
}

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

#### 实战演练

是时候与CryptoKitties合约交互起来了！

我们来定义一个函数，从kitty合约中获取它的基因：

* 1.创建一个名为feedOnKitty的函数。它需要2个uint类型的参数，_zombieId和\_kittyId，这是一个public类型的函数。
* 2.函数首先要声明一个名为kittyDna的uint。

> 注意：在我们的KittyInterface中，genes是一个uint256类型的变量，但是如果你还记得，我们在第一章中提到过，uint是uint256的别名，也就是说它们是一回事。

* 3.这个函数接下来调用kittyContract.getKitty函数，传入_kittyId，将返回的genes存储在kittyDna中。记住——getKitty会返回一大堆变量。但是我们只关心最后一个——genes。
* 4.最后，函数调用feedAndMultiply，并传入_zombieId和kittyDna两个参数。

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

  address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
  KittyInterface kittyContract = KittyInterface(ckAddress);

  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    _createZombie("NoName", newDna);
  }

  // define function here
  function feedOnKitty(uint _zombieId, uint _kittyId) public {
     uint kittyDna;  // 声明一个参数

     // 多参数返回，前边不需要的可以用空格，只获取需要的返回参数
     (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);

     feedAndMultiply(_zombieId, kittyDna);
  }

}
```

## 三、if语句用法

我们的功能逻辑主体已经完成了，现在让我们来添加一个奖励功能吧。

这样吧，给从小猫制造出的僵尸添加一些特征，以显示它们是猫僵尸。

要做到这一点，咱们在新僵尸的DNA中添加一些特殊的小猫代码。

还记得吗，第一课中我们提到，我们目前只使用16位DNA的前12位数来指定僵尸的外观，所以现在我们可以使用最后两个数来处理“特殊的”特征。

这样吧，把猫僵尸DNA的最后两个数字设定为99（因为猫有9条命）。所以在我们这么来写代码：如果这个僵尸是一只猫便来的，就将它DNA的最后两位数字设置为99.

#### if语句

if语句的语法在Solidity中，与在JavaScript中差不多：

```solidity
function eatBLT(string sandwich) public {
  // 看清楚了，当我们比较字符串的时候，需要比较他们的 keccak256 哈希码
  if (keccak256(sandwich) == keccak256("BLT")) {
    eat();
  }
}
```

#### 实战演练

让我们在我们的僵尸代码中实现小猫的基因。

* 1.首先，我们修改下feedAndMultiply函数的定义，给它传入第三个参数：一条名为_species的字符串。
* 2.接下来，在我们计算出新的僵尸的DNA之后，添加一个if语句来比较_species和字符串“kitty”的keccak256哈希值。
* 3.在if语句中，我们用99替换了新僵尸DNA的最后两位数字。可以这么做：newDna = newDna - newDna%100 + 99;
* 4.最后，我们修改了feedOnKitty中的函数调用。当它调用feedAndMultiply时，增加“kitty”作为最后一个参数。

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

  address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
  KittyInterface kittyContract = KittyInterface(ckAddress);

  // Modify function definition here:
  function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    // Add an if statement here,添加if语句，并且将后两位数替换为99
    if (keccak256(_species) == keccak256("kitty")) {
      newDna = newDna - newDna % 100 + 99;
    }
    _createZombie("NoName", newDna);
  }

  function feedOnKitty(uint _zombieId, uint _kittyId) public {
    uint kittyDna;
    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
    // And modify function call here:添加参数
    feedAndMultiply(_zombieId, kittyDna, "kitty");
  }

}
```

## 四、部署以太坊实现

#### JavaScript实现

我们只用编译和部署ZombieFeeding，就可以将这个合约部署到以太坊了。我们最终完成的这个合约继承自ZombieFactory，因此它可以访问自己和父辈合约中的所有public方法。

我们再来看一个与我们的刚部署的合约进行交互的例子，这个例子使用了JavaScript和web3.js。

```javascript
var abi = /* abi generated by the compiler */
var ZombieFeedingContract = web3.eth.contract(abi)
var contractAddress = /* our contract address on Ethereum after deploying */
var ZombieFeeding = ZombieFeedingContract.at(contractAddress)

// 假设我们有我们的僵尸ID和要攻击的猫咪ID
let zombieId = 1;
let kittyId = 1;

// 要拿到猫咪的DNA，我们需要调用它的API。这些数据保存在它们的服务器上而不是区块链上。
// 如果一切都在区块链上，我们就不用担心它们的服务器挂了，或者它们修改了API，
// 或者因为不喜欢我们的僵尸游戏而封杀了我们
let apiUrl = "https://api.cryptokitties.co/kitties/" + kittyId
$.get(apiUrl, function(data) {
  let imgUrl = data.image_url
  // 一些显示图片的代码
})

// 当用户点击一只猫咪的时候:
$(".kittyImage").click(function(e) {
  // 调用我们合约的 `feedOnKitty` 函数
  ZombieFeeding.feedOnKitty(zombieId, kittyId)
})

// 侦听来自我们合约的新僵尸事件好来处理
ZombieFactory.NewZombie(function(error, result) {
  if (error) return
  // 这个函数用来显示僵尸:
  generateZombie(result.zombieId, result.name, result.dna)
})
```

