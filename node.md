# 笔记

* 数据类型： 值类型 引用类型， 外加一个映射类型，类似于js里面的map
  * 值类型： bool address 整型(int uint uint256) enum 定长字节数组
  * 引用类型： 结构体 不定长数组
* 合约的状态变量存储在链上，gas费很贵，如果计算不改变链上状态，就可以不用付 gas
* 函数： function 函数名(参数类型 参数名,...) 可见性修饰符 功能性修饰符 returns?(返回值类型 返回值名称? **数组类型**返回值存储位置memory|storage?, ...) { 函数体 }
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
