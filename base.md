# 基础
## 1.数据类型

### 1.1 前端与智能合约数据类型对应关系
> 以下是 Solidity 数据类型与前端常用工具（如 **Web3.js** 和 **Ethers.js**）中数据类型的对应关系，以及在前端如何构造这些数据类型的说明：

---

#### 1.1.1 **基本类型**

| Solidity 类型 | 前端对应类型 | 构造方式 (示例) |
|---------------|---------------|-----------------|
| `address`     | `string`      | `"0x..."` (40位十六进制字符串，必须以 `0x` 开头)<br>**示例**: `const addr = "0x1234567890abcdef1234567890abcdef12345678";` |
| `bool`        | `boolean`     | `true` 或 `false`<br>**示例**: `const flag = true;` |
| `uint` / `uint256` | `string` 或 `BigNumber` | 使用字符串表示大整数，或使用库如 **Ethers.js** 的 `BigNumber`<br>**示例**: `const amount = "1000000000000000000"; // 1 ether in Wei`<br>**Ethers.js**: `const amount = ethers.BigNumber.from("1000000000000000000");` |
| `int` / `int256` | `string` 或 `BigNumber` | 与 `uint` 类似，支持负数<br>**示例**: `const balance = "-500";` |
| `bytes32`     | `string`      | 64位十六进制字符串（32字节），以 `0x` 开头<br>**示例**: `const data = "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef";` |
| `bytes`       | `string` 或 `Uint8Array` | 动态字节数组，传入十六进制字符串或字节数组<br>**示例**: `const payload = "0xabcdef"; // 动态长度`<br>或者 `const payload = new Uint8Array([0xab, 0xcd, 0xef]);` |

---

#### 1.1.2 **数组类型**

| Solidity 类型            | 前端对应类型 | 构造方式 (示例) |
|--------------------------|--------------|-----------------|
| `uint[]` / `uint256[]`   | `Array<string>` 或 `Array<BigNumber>` | **示例**: `const arr = ["100", "200", "300"];`<br>**Ethers.js**: `const arr = [ethers.BigNumber.from("100"), ethers.BigNumber.from("200")];` |
| `address[]`              | `Array<string>` | **示例**: `const addrs = ["0x123...", "0xabc..."];` |
| `bytes32[]`              | `Array<string>` | **示例**: `const dataArr = ["0x123...", "0xabc..."];` |
| `bool[]`                 | `Array<boolean>` | **示例**: `const flags = [true, false, true];` |

---

#### 1.1.3 **结构体类型**

Solidity 的结构体在 ABI 中被表示为对象，前端需要以对象的形式构造数据，字段顺序和字段名必须与结构体一致。

| Solidity 结构体            | 前端对应类型 | 构造方式 (示例) |
|-----------------------------|--------------|-----------------|
| `struct Order { address user; uint256 amount; bool active; }` | `{ user: string, amount: string, active: boolean }` | **示例**: `const order = { user: "0x123...", amount: "1000", active: true };` |

---

#### 1.1.4. **映射 (Mapping)**

`mapping` 类型无法直接传递到合约，因为mapping 是 storage 变量，必须存储到链上，而函数的参数不能设置为 storage， 只能是memory
折中的方案是可以通过合约提供的对mapping数据的细化操作函数进行交互， 这种函数一般参数都是上面那种常见的数据类型。

---
