# 笔记

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
  * 映射类型： mapping
* 内存分配
  * 一个存储槽为一个32字节的存储空间，为以太坊虚拟机(EVM)的最小存储空间 
  * bytes 属于动态字节数组，能让多个字符共享一个存储槽，而bytes1~32 每个都占用一个存储槽，即使bytes1非常短也占用一个，造成存储空间浪费
  * **storage 的变量必须指向一个已有的引用地址，而不能是新建的**
* 合约的状态变量存储在链上，gas费很贵，如果计算不改变链上状态，就可以不用付 gas
* 函数： function 函数名(参数类型 存储位置? 参数名,...) 可见性修饰符 功能性修饰符 returns?(返回值类型 存储位置? 返回值名称?, ...) { 函数体 }
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
