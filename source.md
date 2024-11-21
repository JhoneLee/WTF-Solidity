# 看源码得技巧

## 1. 基础语法

### 1.1 using
* `using SafeMath for *;` 表示将这个库应用于所有它支持的数据类型上

### 1.2 address
* solidity中地址分为两大类：
  * EOA: 外部地址，一版是用户钱包之类
  * 合约地址： 包括代币、智能合约等
* address(0): 在以太坊中表示以太币(eth) 的地址
* address(this)：表示本合约地址

### 1.3 合约与接口
* 通过自己编写Token 合约，里面写虚方法，就可以实例化来源五花八门的代币token， 类似于IERC20接口，但是自己还可以完全控制。 EtherDelta 就是这么做的。
* 在合约中各种入参和执行步骤的检查，可以用 require 包裹返回值为bool的操作， 中间一旦出现异常，直接回滚
* 存入金额操作强调链上状态优先，即先转账成功我再改账本。 转出操作强调合约内部状态优先，改完账本再转账
  * 转出这么搞是因为我们转出需要调用第三方代币的transfer方法，而第三方transfer是不可信的， 它可以反调我们的转账方法，在没有改账本前多次转出代币，而账本只改一次， 这种就叫**重入攻击**
  * 转入这么搞是可以理解和转出的操作相反，即可保证合约状态的正常。
* 过期标准：
  * 区块高度标准： 在智能合约里订单有效性可以用区块链高度标记，如果一个订单的expires 设置为65536， 表示只要当前区块链高度在 65537 之内均可交易，否则作废； `block.number <= orderSigned.expires`
  * 时间标准： 用户直观的还是想设置自己订单的过期时间，所以可以根据当前智能合约时间进行判断，时间单位是秒 `uint256 orderExpires = block.timestamp + 3600;  // 当前时间 + 1小时`

### 1.4 消息签名
#### 1.4.1 ecrecover 函数
>是一个在以太坊智能合约中常用的内置函数，用来从签名中恢复出签名者的地址。签名在以太坊中使用了 ECDSA（Elliptic Curve Digital Signature Algorithm）算法，而 ecrecover 就是用来解读该签名的工具。

接收4个参数：
* hash bytes32: 我希望验证的消息hash值
* v int8: 签名恢复的标志位，通常为27 或 28
* r bytes32: 签名第一部分
* s bytes32: 签名第二部分

```solidity
function decode(OrderSigned memory orderSigned) public  {
   // 自己对前端传递来的数据进行消息签名
   bytes32 hash = keccak256(
        abi.encodePacked(
            address(this),
            orderSigned.tokenGet,
            orderSigned.amountGet,
            orderSigned.tokenGive,
            orderSigned.amountGive,
            orderSigned.expires,
            orderSigned.nonce
        )
    );
    // 反解出消息的签名用户，这个用户就是前端签名时 wallet对象绑定的用户
    user = ecrecover(
        keccak256(
            // 手动不上消息自动前缀
            abi.encodePacked("\x19Ethereum Signed Message:\n32", hash)
        ),
        orderSigned.v, // v是位数27/28， r s是前端传过来的hash截取字符串
        orderSigned.r,
        orderSigned.s
    );
}
```

#### 1.4.2 ethers.js 对消息签名并适配EtherDelta
* ethers.solidityPackedKeccak256
* wallet.signMessage

```js
// 先求出hash， 传参就是 类型数组 实际取值数组
const hash = ethers.solidityPackedKeccak256(
    [
      'address',
      'address',
      'uint256',
      'address',
      'uint256',
      'uint256',
      'uint256',
    ],
    [
      contarctAddress,
      tokenGet,
      amountGet,
      tokenGive,
      amountGive,
      expires,
      nonce,
    ]
  );
// 像智能合约里加前缀那样，给hash补上前缀
const messageHashBytes = ethers.getBytes(hash);
const signature = await wallet.signMessage(messageHashBytes);
// 从签名中解析 r, s, 和 v
const r = '0x' + signature.slice(2, 66); // 签名的前 32 字节 从2开始是因为前面有固定的0x, 取32为何截66 是因为签名是用2个十六进制表示一个字节
const s = '0x' + signature.slice(66, 130); // 签名的中间 32 字节
const v = parseInt(signature.slice(130, 132), 16); // 签名的最后 1 字节，转换为整数
// 假设我的合约验签函数叫decode， 我把签名参数传递给智能合约
const tx = await contract.decode(orderSigned);
await tx.wait();
const userRes = await contract.user();
```

#### 1.4.2 DEX app EtherDelta 的数据结构设计
```solidity
// 订单信息入参struct
 struct OrderSigned {
     address tokenGet; // 用户想要交换的代币token
     uint256 amountGet; // 交换的金额
     address tokenGive; // 用户提供的代币token
     uint256 amountGive; // 提供的金额
     uint256 expires; // 订单过期时间
     uint256 nonce; // 订单唯一标志
     address user; // 发起订单的用户
     uint8 v; // 签名恢复标志， 27 或 28
     bytes32 r; // ECDSA 签名的一个部分（即椭圆曲线数字签名算法中的一个输出），它是用私钥对 hash 进行签名时生成的 需要前端用ethers.js 做
     bytes32 s; // 是 ECDSA 签名的另一部分，与 r 一起构成完整的签名。它用于保证签名的唯一性和防止篡改。需要前端用ethers.js 做
 }
```

### 1.5 数学计算
* 利用SafeMath进行基本运算，防止精度丢失
* 如果有小数计算，先将小数转换成整型，计算完毕后再除去换乘整型的单位， 比如： `feeRebateXfer = SafeMath.mul(amount, feeRebate) / (1 ether);`
* 避免除0





## 2. 实际开发

### 2.1 DEX App

#### 2.1.1 状态定义
* 管理员地址，当操作需要只有管理员才能完成时，管理员地址会提供给校验权限修饰器对msg.sender进行比对
* 收费账户， DEX app 不是做公益，要收手续费赚钱，放手续费的地方
* 费率状态：一般设计为两头吃，对卖方收费费率， 对买方收费费率
* 交易金记账本： 推荐数据结构为mapping ， key 为代币token,  value 为  mapping(用户地址， 金额)  记录某种币下各用户的金额
* 订单签名验证记账本： 数据结构为mapping， key 为用户地址，value 为 mapping(订单号(一种经过特定计算的哈希值), 是否签名验证过)
* 订单成交进度记账本： 数据结构为mapping, key 为用户地址， value 为 mapping(订单号， 订单当前金额) 记录当前用户订单中所需要的币兑换的进度，假设一个人要用chainLink换100个狗狗币，其他持有狗狗币的用户给他凑
* 一些引用的外部合约地址，比如 用户等级计算 合约等

#### 2.1.2 数据设计
* 订单消息签名数据：
  * 用户想要交换买入的币种及币值
  * 用户提供作为交换的币种及币值
  * 签名的用户地址
  * 订单过期时间
  * 订单编号
  

#### 2.1.3 事件定义
> 要突出业务上的关键操作， 用户的行为，包含足够的信息， 便于查询， 可以从以下角度去分析
* 关键操作
  * 订单的操作： 挂单 取消订单 成交
  * 资金流动： 充值 提现
  * 关键合约状态变化
  * 合约自身更新
* 用户行为(和关键操作有重合)
  * 挂单
  * 撤单
  * 买入
  * 充值：和游戏等充值不同，用户可以提现
  * 提现
  * 账户变化
  * 用户的等级： 普通用户 银卡用户 金卡用户等等，不同用户在费率上又优惠，每笔交易还有返佣
* 事件包含信息： 从前端和外部工具角度，去暴露对应的数据
* 巧妙利用最多3个indexed 事件索引， 提升查询效率


#### 2.1.4 函数功能
* 初始化
  * 设置管理员地址
  * 设置各种交易收费费率
  * 设置收取交易费的钱包
* 变更管理员： 便于管理权交接 防止单一地址泄密增加安全性
* 变更各种交易收费费率： 这里要加判断，etherDelta 规定费率只能越设越低，给用户交易信心。 其中还有一些特殊规定， 交易购买手续费率必须要比返点费率大，否则就是赔钱做交易；
* 充值eth功能， 充值函数通过payable控制，只收eth， 充值修改状态成功后触发 充值事件，告知前端/用户充值成功
* 充值代币功能：
  * 需要判断币不是eth
  * 然后判断转账是否成功
  * 修改记账本
  * 触发充值事件
* 提现eth：
  * 先检查用户在合约里面的余额: `require(tokens[address(0)][msg.sender] >= amount);`
  * 合约中关于金额存储的状态修改
  * 给用户EOA打款方法： `(bool success, ) = msg.sender.call{value: amount}("");`
  * 校验是否转账成功，不成功通过require触发状态回滚： `require(success, "withdraw failed.");`
  * 触发对应的提现事件
* 提现代币：
  * 确认用户传递的币token不是eth
  * 检查余额
  * 修改记账本
  * 调代币的transfer执行转出
  * 触发提现事件
* 查询某个币下面某用户的额度， 直接返回记账本的映射即可
* 创建订单（Maker调用）：
* 交易操作（Taker调用）：
  * 根据参数传入的用户签名对象，拿出特定的属性值进行hash操作，生成一个专属hash值作为订单的唯一标志
  * 拿到上面的hash后，校验这个消息是否和签名的用户一致， 这里前端用谁的钱包签名的用户就是谁，无法篡改
  * 校验活动：
    * 用户订单是否被记录在订单记账本上， 如果没有判断 当前订单签名信息是否和Maker 用户地址一致
    * 
    * 
  * 修改相关记账本
    * 判断用户的会员等级，根据等级不同计算不同的费率
  * 触发交易事件



## 3. 常用库及合约
* @openzeppelin
  * SafeMath 安全的数学运算，提供加减乘三种计算函数
