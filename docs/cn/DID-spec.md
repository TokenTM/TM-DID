#### TokenTM DID方法
以下文档定义了TokenTM公司的DID方法。

以下规范将来可能会有所变化，但它们必须符合W3C凭证社区组指定的最新版通用DID规范。

此方法的功能DID由此存储库中的智能合约提供。

#### DID格式
该方法应使用名称标识selfkey。SelfKey DID具有以下格式：

```
did:tm:<32 byte hexadecimal string>
```

例如：

```
did:tm: 0xe32df42865e97135acfb65f3bae71bdc86f4d49150ad6a440b6f15878109880a
```

它<32 byte hexadecimal string>对应于keccak256以及在DID合约中生成的随机数连接的以太坊地址的哈希值。

DID在合约中注册，由单个以太坊地址控制，该地址默认设置为最初调用该createDID方法的地址。该地址稍后可以将控制转移到不同的地址，或者更新/删除合约中的相应DID。

#### ID字符串生成
标识符字符串在DID合约的以下行中生成：

```
bytes32 _hash = keccak256(abi.encodePacked(msg.sender, nonce));
```

凡nonce在每次调用增加，从而使结果被认为是随机的，地址是能够创建和控制多个的DID。

#### DID结构定义
分类帐上的每条DID记录都显示以下结构：

```
struct DID {
        address owner;
        uint256 created;
        uint256 updated;
        bool    revoked;
}
```

当DID被撤销，revoked为true。
#### DID操作
以下部分定义了管理DID所支持的操作。
##### 创建
通过TM_DID合同提交调用以下方法的事务来完成DID创建：

```
createDID()
```

记录DID的msg.sender为owner，记录创建时间，revoked置为true。
##### 读
通过TM_DID合同提交调用以下方法的事务来读取DID的记录：

```
getDID(address didAddress) public view returns(address, uint256, uint256, bool)
```

返回owner，创建时间，更新时间，是否撤销
##### 更新
通过调用

```
setAttribute(address didAddress, bytes32 name, bytes memory value, uint validity)
```
来更新DID的attribute
```
revokeAttribute(address didAddress, bytes32 name, bytes memory value)
```

撤销attrubute
##### 撤销

```
revokeDID(address didAddress)
```
置revoked为true

#### 代理
##### 判断代理是否有效

```
validDelegate(address didAddress, bytes32 delegateType, address delegate)  returns(bool)
```

##### 添加代理

```
addDelegate(address didAddress, bytes32 delegateType, address delegate, uint validity)
```

##### 撤销代理

```
revokeDelegate(address didAddress, bytes32 delegateType, address delegate)
```


#### 方法的扩展
DID分类帐实现为以太坊网络上的链上持久性身份注册的简单层，同时允许扩展它以包括其他数据和功能。通过使用身份合同作为分类帐上DID的控制器来实现可扩展性。特别是，ERC725结合密钥管理合同，例如ERC734预期是添加此附加功能的常见情况（例如，定义服务端点，密钥轮换，委派和许可等），同时允许使用其他标准和甚至允许DID所有者从一个合同实现转换到另一个合同实现而不会丢失其标识符。
#### 安全考虑
应考虑以下几点，并且社群可以就一般安全问题进行讨论：
- 由于DID文档未显式存储但是动态生成，因此无法对其进行签名，因此依赖方需要信任解析器代码才能正确执行。预计依赖方使用TokenTM提供的解析器库代码的验证（例如校验和验证）版本或社区的其他可信来源。
- 一旦控制器地址将DID的控制转移到新地址，它就失去了对该DID执行操作的所有能力。因此，必须小心地执行该操作以避免错误（将DID控制转移到错误的地址或不在用户控制下的地址）。
- 此方法在分类帐级别未定义委派或恢复机制。必须通过密钥管理和代理身份智能合同（例如ERC725 / ERC734）实施适当的可恢复性。预计这将成为在此方法下管理的DID的常见做法。
#### 补充说明
已经提出了其他方法来在以太坊平台上提供分散的身份，然而，TokenTM DID方法是基于以下原则设计的：
- 理想情况下，DID应该是“加时间戳”和可撤销的。一般性的以太坊地址不满足此要求。
- 与DID相关联的公共数据和此数据的表示结构应根据通用规范或此特定方法中的更改进行升级。这是通过提供简单的标识符< - >地址映射以及解析器可以解释为以不同方式解析的灵活元数据属性来实现的。预计此方法下的DID将与聚合必要数据的代理身份智能合约相关联，同时允许简单帐户控制DID以涵盖某些简单情况。
- 应该能够针对任何任意实体（以太网网络中是否存在）提出公开声明。在这方面，使用bytes32可信声明类型可以指代TokenTM DID或表示链上或链外的任何其他任意事物的散列。虽然可信声明的性质超出了本方法的范围，但也正在开发公共索赔登记合同，以便与本文件中定义的DID一起使用。