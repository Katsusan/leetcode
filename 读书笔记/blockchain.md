1. 源码
    - https://github.com/bitcoin/bitcoin    比特币
    - https://github.com/ethereum/go-ethereum   以太坊


2. 区块链开发指南
    - https://blog.csdn.net/Blockchain_lemon/article/details/81880183
    - https://cryptozombies.io/zh/lesson/2/chapter/7    Solidity

3. 区块链分类
    - 公共链(Publick Blockchain):利用密码学和经济激励建立共识从而形成去中心化的信用机制。共识机制一般为工作量证明(PoW)或权益证明(PoS).如比特币/以太坊
    - 联盟链(Consortium Blockchain):区块读写与参与记账权限按联盟规则制定，典型如40多家银行的区块链联盟R3与linux基金会支持的超级账本Hyperledger.
    - 私有链(Private Blockchain):读写与几张权限由私有组织控制，一般用于企业内部应用如数据库管理/审计，典型如MultiChain多链。

4. 加密算法
    椭圆曲线加密算法(非对称加密):ECC

    私钥(256位)：操作系统底层产生一个密码学安全的256位随机数
    
    ↓ Secp256K1椭圆曲线算法

    公钥(65字节)

    ↓ SHA256 + RIPEMD160

    公钥哈希(20字节)=body, 前面加上版本信息0x00, 后面加4字节的地址校验码(对公钥哈希进行2次SHA256取前4字节)

    ↓ BASE58

    比特币钱包地址(58字节)

5. 共识机制
    - PoW: 工作量证明, 即矿工把网络中尚未记录的交易打包到一个区块中，不断遍历找到一个随机数使得新区块加上随机数的哈希满足一定的难度条件，
        比如前十位为0，这样就确定了区块链网络的最新的一个区块，也就是获得了本轮的记账权。然后把该区块广播出去，其它节点验证Ok后加入自己版本
        的区块链上。
        + 优点： 完全去中心化，只要破坏者算力不达到全网一半以上就必定会达成一致。
        + 缺点： 造成大量的资源浪费，并且达成的周期较长，每秒至多做7笔交易，不适合商业应用。
    - PoS： 权益证明，要求节点提供一定数量的代币来获取区块链的记账权。
        为了避免富有者一直胜出，可加入其它因素，比如链龄。
        + 优点： 缩短共识达成时间，降低PoW机制的资源浪费。
        + 缺点： 攻击成本低，并且共识会倾向于富有账户，降低公正性。
    - DPoS： 股份授权证明，类似董事会投票。如bitshares比特股采用的是用PoS机制选出一定的见证人，每人有两秒时间生成区块，未能生成则交由下个区块。
        + 优点： 大幅缩小了验证/记账节点的数量，可达到秒级共识。
        + 缺点： 选固定数量的见证人不太符合完全去中心化的理念。

    - 分布式一致性算法：PBFT拜占庭容错算法，Paxos，Raft等

6. 每笔交易的输出实际是指向一个脚本而非地址，主要分为以下两种。
    - P2PKH(Pay-to-Public-Key-Hash)：支付给公钥的哈希地址，接收方只要用地址对应的私钥对该输出进行签名即可花掉这笔输出。
    - P2SH(Pay-to-Script-Hash)：支付给脚本的哈希地址，以多重签名为例，需要N把私钥中的M把同时签名才能花掉该输出，类似多个钥匙才能打开的保险柜。


7. 图灵完备
    当一组数据操作的规则(一组指令集/编程语言/元胞自动机)满足任意数据按一定的顺序可以计算出结果，则称为满足图灵完备性。

8. 以太坊相关概念
    - 叔区块(Uncle): 符合难度条件但区块里的交易不被确认的区块，或者叫“废块”(Stale)。由于以太坊的生成区块速度比较快，网络繁忙的时候会更容易出现废块。
    - 贪婪最重观察子树(Greedy Heaviest Observed Subtree/GHOST)：以太坊用类似于一棵树(包含叔区块)的结构来构建区块链，而不是一颗链条式。
    - Gas花销(Gas cost)：Gas花销是静态的，对于每一种操作可以视为不变，目的在于保证每个操作对应的计算资源保持一定。
    - Gas价格(Gas price): 花费每个gas所需的以太币数量，可由用户自行调整，基准价格随以太币市值浮动。
    - Gas费用(Gas fee): Gas价格乘以Gas花销，即智能合约所需的真实费用，单位为以太币。
    - 外部账户：
        + 可以存储以太币
        + 可以发起交易，包括交易以太币和部署智能合约
        + 由用户创建密钥，并管理账户
        + 不支持智能合约代码
    - 合约账户：
        + 可以存储以太币
        + 可以支持智能合约代码
        + 可以响应别的用户或合约执行此智能合约的请求，并返回结果
        + 可以调用别的智能合约

9. 密码学技术
    - 承诺方案(Commitment Scheme)：一类重要的密码学基本模型，承诺模型可以看作为一个密封信件的数字等价体。
        + 承诺值计算：输入消息m和随机值r，返回承诺值c=commit(m,r)
        + 承诺值验证: 输入承诺值c，消息m，随机值r，若c==commit(m,r)则返回真，否则返回假
        因此如果令commit(m,r) = Hash(r||m)，利用哈希函数的抗碰撞和不可逆可满足承诺的隐藏性和绑定性。
    - Merkle树：一类基于哈希值的二叉树或多叉树。 叶子节点存储数据的哈希值，而父节点存储各个子节点的组合哈希值。
    - 椭圆曲线密码算法(Elliptic Curve Cryptography/ECC):
        + secp256k1椭圆曲线形如 y^2=x^3+ax+b,，由六元组D=(p,a,b,G,n,h)定义，P+P+...+P=kP=Q, P，Q为椭圆曲线上的两点。
        + 以下为椭圆曲线签名生成步骤，k为私钥，Q为公钥
        + 步骤1：产生一个随机数d, 1<=d<=n-1,
        + 步骤2：计算dG = (x1, y1), 并将x1取整为x1',
        + 步骤3：令r=x1' mod n, 若r为0则转向1.
        + 步骤4：计算d^(-1) mod n,
        + 步骤5：计算Hash(m)，并转化为整数e
        + 步骤6：计算s=d^(-1)*(e+kr) mod n ⭐
        + 步骤7：(r,s)即为消息m的签名

        + 以下为椭圆曲线签名验证步骤，Q为公钥
        + 步骤1：验证r,s为区间[1,n-1]上的值
        + 步骤2：计算Hash(m)，转化为整数e
        + 步骤3：计算w=s^(-1) mod n
        + 步骤4：计算u1=ew mod n, u2=rw mod n
        + 步骤5：计算X=u1G+u2Q
        + 步骤6：若X==O，则拒绝签名。否则将X的x坐标x1转化为整数x1',计算v=x1' mod n
        + 步骤7：当且仅当v==r时，签名验证通过
        

10. 




