# 笔记
* 常识
  * EIP:是针对以太坊底层协议的改进建议, 涉及区块链协议的更改、核心功能的优化或新增，比如共识机制、虚拟机（EVM）改动、Gas 规则调整等。
  * ERC: 针对智能合约标准化的提案，定义了 应用层标准，特别是 token 和合约的接口规范
  * 在以太坊中，数据必须编码成字节码才能和智能合约交互
  * 可选择性import的内容有： 库、合约、接口、结构体、枚举， 其他不可以
* remix:
  * 合约中定义了一个public 数组之后，部署的合约在调试时 array后面会有一个输入框，首先展示的蓝色属性按钮也是一个函数，是你定义public的getter方法； 其次对于数组来说需要你输入索引才能拿到值，而不是输出整个数组值；
  * 左上角填写以太币金额的输入框不是部署的时候给设置金额初始值，是你在转账操作的时候的msg.value
* 数据类型： 值类型 引用类型， 外加一个映射类型，类似于js里面的map
  * 值类型： bool、 address、 整型(int uint uint256)、 enum、 定长字节数组(bytes1~32数值表示长度)
    * uint后面可以接数字，表示你的数字范围，但是必须是8的整数倍 比如 uint8 uint16等， 后面的数字表示 1至2^n-1 的数字范围
    * uint不加数字默认是 uint256
  * 引用类型： 结构体 数组 不定长字节数组 string
    * 数组定义必须指定存储位置 storage|memory|calldata， 合约状态必须定义在storage上，局部可以选择定义在内存或链上。
    * 数组不用给每个元素都指定类型，以第一个为准 [uint8(1),2,10] 后面都是uint8
    * 如果创建的是动态数组，你需要一个一个元素的赋值， 所谓批量赋值也只是用循环按索引赋值
    * 对于memory修饰的动态数组，可以用new操作符来创建，但是必须声明长度，并且声明后长度不能改变  `uint[] memory array8 = new uint[](5);`
    * 对于内存存储而言，初始化时就要一次性确定好存储大小和内容，后面不能再更改大小，也就是说 memory 数组不得使用push pop， 只有storage可以
    * 结构体必须定义在合约级别，而不是函数内部。定义结构体不消耗gas， 只有实例化的时候消耗；
    * 结构体初始化： `student = Student({id: 4, score: 60});`
  * 映射类型： mapping
    * 映射的存储位置必须是storage
* 内存分配
  * 一个存储槽为一个32字节的存储空间，为以太坊虚拟机(EVM)的最小存储空间 
  * bytes 属于动态字节数组，能让多个字符共享一个存储槽，而bytes1~32 每个都占用一个存储槽，即使bytes1非常短也占用一个，造成存储空间浪费
  * **函数内创建的storage 的变量必须指向一个已有的引用地址，而不能是新建的，同时也不能是memory/calldata的，只能来源于合约状态属性**
  * 函数内创建色storage引用类型变量起到一个存储中转站的作用，够避免多次访问合约的状态变量，从而减少不必要的 gas 消耗
  * 合约层级定义的引用类型状态可以在函数内被赋值
* 合约的状态变量存储在链上，gas费很贵，如果计算不改变链上状态，就可以不用付 gas
* 函数： function 函数名(参数类型 存储位置? 参数名,...) 可见性修饰符 功能性修饰符? 装饰器? 继承规则符(virtual父类可覆写|override子类已覆写) returns?(返回值类型 存储位置? 返回值名称?, ...) { 函数体 }
  * 可见性修饰符 
    * external 能节省gas费，减小合约的大小，语义化更明确
    * internal 更像是protected 关键字
  * 功能性修饰符
    * view 和 pure 在外部调用时不用付gas费，也就不改变状态，view 是只读，pure 不读不写，只执行逻辑（这里的读写都是针对于状态变量而言）
    * payable 函数即使是一个空函数也可以接收以太币，但是可以在逻辑中写一些内容：比如更新余额记录、发出事件或提供支付凭证
  * 用returns 标注了返回值名称后，只要给这些返回值名称赋值，默认这个函数就给你返回这些值, 但是也支持return
    ```js
    // 返回多个变量
    function returnMultiple() public pure returns(uint256, bool, uint256[3] memory){
        return(1, true, [uint256(1),2,5]);
    }
    // 命名式返回 
    function returnNamed() public pure returns(uint256 _number, bool _bool, uint256[3] memory _array){
        _number = 2;
        _bool = false;
        _array = [uint256(3),2,1];
    }
    ```
   * 也支持结构式赋值，区别是js用{} solidity用 ()
   * this 引用方法表示外部调用，会消耗更多的gas， 只有在模拟外部调用、调用payable方法的时候才用，合约内自己调自己只需要使用方法名即可
   * 函数参数可以显性的设置引用类型（无论长度是否可变的数组、结构体、mapping）变量的存储位置, 取值范围为storage(最费gas,存在链上) memory(存内存 可变) calldata(存内存 不可变)
   * **函数签名**格式是 `函数名(参数类型1, 参数类型2, ...)` 注意：uint和int要写为uint256和int256
   * 函数选择器： `contract.函数名.selector` 用于生成函数ABI编码、 代理合约、 call 方法用
   * abi全局变量:
     * 生成abi: `abi.encode`  `abi.encodePacked` `abi.encodeWithSignature` `abi.encodeWithSelector` 均可生成函数的abi编码， 前两个用于编码函数的参数
     * 编码参数需要结合bytes4 类型的函数选择器去生成整个函数调用的abi信息：
       ```solidity
       bytes4 funcSelector = bytes4(keccak256("setMessage(string)"));
       bytes memory param = abi.encode(inputMsg);
       bytes memory callfuncData = abi.encodePacked(funcSelector, param);
       (bool success, bytes memory data) = contractAddr.call(callfuncData);
       ```
     * 通过反编译不开源合约可以得到函数的`method id`，可以通过函数签名计算得到 `bytes4(keccak256("mint(address)"))` , 通过abi encodeWithSelector 可以调用这个函数 `bytes memory data = abi.encodeWithSelector(bytes4(0x533ba33a)); address(contract).staticcall(data);`
     * abi.decode用于解码abi.encode生成的二进制编码，将它还原成原本的参数
 * 事件
   * 事件的参数分为索引参数和普通参数，索引参数用来给监听事件的客户端过滤用的，最多只能有三个，一般都是地址或者字符串什么的 `event Transfer(address indexed from, address indexed to, uint256 value);`
   * 适合使用的场景
     * 权限变更：记录用户角色或权限的变化。
     * 状态更新：记录某些实体的状态改变（例如任务的状态、合约的状态等）。
     * 投票系统：记录投票的选项和投票者。
     * 合约升级：记录合约的升级或新版本的发布。
     * 用户注册：记录用户的注册事件或某些操作的触发。
 * 面向对象
   * 继承关键字 is
   * 支持单继承和多继承，多继承排序是 辈分从高到低
   * 修饰器（modifier）也可以和方法一下被覆写
   * 覆写不可以修改参数类型和返回值，这点和java ts一致
   * 当多个父合约（base contracts）具有相同的函数名和相同的参数类型时，子合约（derived contract）需要显式地覆写（override）这些方法，因为 Solidity 无法确定应该使用哪一个父合约的实现
   * 多继承时，override需要指明覆写的父合约，而且一个也不能落下 `function setScore() public override(Father,Father2) verifyOwner`
   * 父合约如果自己的方法也覆写了爷爷合约的方法，需要给方法表明既是覆写也是可覆写 `function setScore() public virtual  override verifyOwner`
   * 构造函数继承的时候，需要给父合约的构造函数也传值 ` constructor(uint8 inputAge, address owner) Father2(owner)` 自己的构造函数多加参数，然后扔给父合约
   * 菱形/钻石继承是指一个子合约有两个或两个以上的父合约，也就是多重继承
   * super.父合约函数名()  可以依次执行
   * 也有[接口和抽象合约](https://github.com/JhoneLee/WTF-Solidity/blob/main/14_Interface/readme.md)的概念，接口定义如下`interface IERC721 is IERC165` 也可以继承
     * 不能包含状态变量
     * 不能包含构造函数
     * 不能继承除接口外的其他合约
     * 所有函数都必须是external且不能有函数体
     * 继承接口的非抽象合约必须实现接口定义的所有功能
   * 可以只知道一个合约地址，只要它满足某个接口的规范，就可以直接生成可调用的合约 `IERC721 BAYC = IERC721(0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D);` BAYC就可以直接当成合约被调用
   * 重载也和其他主流面向对象语言规则一样
 * 异常处理
   * 有三个API  error  require  assert, 其中require和asset的第一个参数都是一个布尔判断表达式，成立就划过去，不成立就报错，require可以反馈报错原因，assert不可以
   * error 的定义类似事件/modifier, 必须搭配 revert关键字, revert 类似throw , 但它不是一个普通关键字，表示**回滚当前交易**
     ```solidity
     error TransferNotOwner(address sender); // 自定义的带参数的error
     function transferOwner1(uint256 tokenId, address newOwner) public {
         if(_owners[tokenId] != msg.sender){
             revert TransferNotOwner();
             // revert TransferNotOwner(msg.sender);
         }
         _owners[tokenId] = newOwner;
     }
     ```
 * 库
   * solidity 用库合约封装一些算法、函数， 专门有library 关键字修饰库合约 `library Strings`, **不能被继承**
   * solidity 本身没有包管理这个概念，需要把 library封装成单独的sol文件后，import进来。 如果用hardhat 就可以通过npm安装库合约包，然后再sol文件import
   * 使用库的方法：`using <Library> for <Type>`  表示把这个库应用于这个类型，之后这个类型的数据就有这个库的方法可以用了. 如果这个库有针对多个类型的方法，就多写几次using for 类型
   * 使用库的方法2： 直接 库.方法
   * 可以导入库的形式： 本地文件路径、node_modules的类全局路径`import '@openzeppelin/contracts/access/Ownable.sol';` 、 github网址，精确到.sol文件级别
   * 库中如果有很多合约很多接口，只想用其中一个或若干个，可以用 `import { Yeye,A,b } from './Yeye.sol'` 这一点和js一样
 * 奇葩操作
   * delete 操作符：js里面是删除属性，这里是把状态给重置
   * 只有数值变量可以声明constant和immutable；string和bytes可以声明为constant，但不能为immutable， 因为string 和 bytes的长度是动态变化的，与immutable设计初衷不符。
   * constant 类似于vite 的import.env.meta ， immutable全新概念，表示只能在contructor中初始化完毕后就不能再改的变量，constant这种没有gas费，而immutable有但是比普通变量少
   *  `call{value:_value}()` 和 `new Contract{value:_value}()` 这种写法叫附加参数，{}里面的key不能乱写，目前可知的是：
     * value eth金额
     * gas 最大支持的gas费
     * salt CREATE2指令的盐值
     * code CREATE2的指定合约字节码
   * 合约自己可以删除自己 `selfdestruct(_addr)`  _addr 是一个接收本合约余额的地址变量  selfdestruct需要看EIP版本，EIP-6780减少了SELFDESTRUCT操作码的功能，再调用会将合约余额转走，合约不删除
   * try-catch只能被用于external函数或创建合约时constructor（被视为external函数）的调用,格式 `try 合约名.函数名() returns?(returnType val) { // 成功操作 } catch {}`
   * 
 * 合约调合约(不是合约调library合约)
   * 知道合约地址，用合约实例化后调用 `OtherContract(_Address).setX(x);`
   * 可以import 一个合约，但是使用的时候需要 `ContractA a = new ContractA()`
   * 函数参数接收一个合约变量，直接调用合约变量的方法 `function callGetX(OtherContract _Address) external view returns(uint x) {  x = _Address.getX(); }`
   * 调用payable函数：`OtherContract(otherContract).setX{value: msg.value}(x);` x随便填
   * 用call 调合约方法： `合约地址.call{value: amount}(abi.encodeWithSignature(函数签名, 传给函数的参数));` 如果不涉及转账，就没有花括号及其内容；
   * 用delegateCall 调合约方法： `合约地址.call(abi.encodeWithSignature(函数签名, 传给函数的参数));` 不支持传递eth, 所谓delegateCall就是把函数签名表示函数的逻辑拿过来，在执行逻辑时，读写的状态和上下文都是当前合约而不是代理合约的
   * 合约工厂可以通过 `new 合约名(参数)`的方式创建新的合约，在拿到实例化的合约地址后保存起来，供对外取用，这种模式为合约工厂
   * 直接new 合约的方式导致产生的合约地址是不可预测的，通过 `new Pair{salt:salt}(param)` 的方式用CREATE2指令创建合约，地址由salt决定。 所以只要salt固定，产生的地址就是不变的、可以通过计算公式预测出来的
   * staticcall 也可以调用合约方法，与call不同，staticcall 用于调不改变合约状态的函数，如果发生状态改变，会导致报错；
 * 转账相关
   * 在转账相关操作中solidity提供了两个**合约接收到eth**的钩子函数 receive 和 fallback， 当msg对象有data的时候触发 fallback，没有触发receive ， 如果没定义receive 也会尝试触发fallback
   * 在ethers.js 里面用wallet.sendTransaction 单纯转账会触发 receive, 其余调合约中自己定义的转账方法或者直接调用fallback，都会触发fallback
   * 这两个钩子函数里面可以执行一些逻辑，包括打印日志、记账等，也可以不定义他俩，顶多没有钩子函数转而用合约函数做上面的事有点膈应
   * 给constructor定义成 payable的，能够支持在部署时转账 `constructor() payable{}`
   * revert关键字  会回滚整个交易的状态，并且撤销所有的操作。无论是成功还是失败的交易，revert 都会让以太币和任何状态变化恢复到交易前的状态。但是消耗过的gas费如果没有退还机制，默认就不退了
   * payable的修饰范围
     * 修饰函数，表示函数可以接收eth
     * 修饰构造函数，表示在合约部署时可以接收eth
     * 修饰address类型的函数参数或变量，表示合约可以给这个地址转账
   * 三个从**合约转账给外部**的api: [transfer send call](https://github.com/JhoneLee/WTF-Solidity/blob/main/20_SendETH/readme.md) , 推荐用call
     * transfer: 调用方式 `接收eth地址.transfer(金额)`  gas限制为2300，如果对方的receive和fallback太复杂导致消耗的gas超过2300，交易就失败了， 交易失败会自动回滚，但是仍然扣gas费，所以要预交易
     * send: 调用方式 `接收方地址.send(发送ETH数额)` gas限制为2300 交易失败不回滚，需要自己抛出异常
       ```solidity
       error SendFailed(); // 用send发送ETH失败error
       // send()发送ETH
       function sendETH(address payable _to, uint256 amount) external payable{
           // 处理下send的返回值，如果失败，revert交易并发送error
           bool success = _to.send(amount);
           if(!success){
               revert SendFailed();
           }
       }
       ```
     * call: 当用于转账的时候调用方式是 `接收方地址.call{value: 发送ETH数额}(data)` data一般是空串， 只有用于调其他合约时才是其他合约的abi字符串，无gas限制，交易失败不回滚，需要自己手动回滚
       ```solidity
       error CallFailed(); // 用call发送ETH失败error
       // call()发送ETH
       function callETH(address payable _to, uint256 amount) external payable{
           // 处理下call的返回值，如果失败，revert交易并发送error
           (bool success,) = _to.call{value: amount}("");
           if(!success){
               revert CallFailed();
           }
       }
       ``` 
