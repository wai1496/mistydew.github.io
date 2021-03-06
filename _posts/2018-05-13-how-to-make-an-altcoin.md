---
layout: post
title:  "如何制作一枚山寨数字货币"
date:   2018-05-13 16:02:28 +0800
categories: Blockchain
---
![bitcoin](/images/20180504/bitcoin.svg)

## 读在前面
比特币相关的解读目前均采用 [bitcoin v0.12.1](https://github.com/bitcoin/bitcoin/tree/v0.12.1)，此版本为官方内置挖矿算法的最后一版。<br>
目前比特币的最新版本为 bitcoin v0.16.0，离区块链 1.0 落地还有些距离。

## 概要
我认为基于比特币制作一枚山寨数字货币是了解比特币或者其底层区块链技术入门的最好方式。
了解各种相关概念后，终于要开始接触源码了，还记得侯捷的“源码之前，了无秘密”吗？
我们也来一探比特币的源码世界。

## 0. 下载比特币源码 v0.12.1

{% highlight shell %}
$ git clone https://github.com/bitcoin/bitcoin.git
$ cd bitcoin
$ git checkout v0.12.1
{% endhighlight %}

关于比特币源码的编译详见[编译比特币源码](/2018/05/03/compile-bitcoin)篇。

## 1. 修改币名

### 1.1. 修改源码文件币种名

{% highlight shell %}
$ mv bitcoin altcoin
$ cd altcoin
$ find . -exec rename 's/bitcoin/altcoin/' {} ";"
{% endhighlight %}

### 1.2. 修改源码中的币名

{% highlight shell %}
$ find . -type f -print0 | xargs -0 sed -i 's/bitcoin/altcoin/g'
$ find . -type f -print0 | xargs -0 sed -i 's/Bitcoin/Altcoin/g'
$ find . -type f -print0 | xargs -0 sed -i 's/BitCoin/AltCoin/g'
$ find . -type f -print0 | xargs -0 sed -i 's/BItCoin/AltCoin/g'
$ find . -type f -print0 | xargs -0 sed -i 's/BITCOIN/ALTCOIN/g'
{% endhighlight %}

### 1.3. 修改源码中币的单位

{% highlight shell %}
$ find . -type f -print0 | xargs -0 sed -i 's/btc/atc/g'
$ find . -type f -print0 | xargs -0 sed -i 's/BTC/ATC/g'
{% endhighlight %}

使用 `grep` 命令查看是否修改成功。

### 1.4. 修改错误拼写

{% highlight shell %}
$ grep -inr bitc
{% endhighlight %}

从结果中可以看到“src/qr/locale/altcoin_et.ts”文件的 `Bitconi`、
“src/qr/locale/altcoin_ar.ts”文件中的 `Bitcion` 和“src/qr/locale/altcoin_da.ts”文件中的 `bitcon`，
这些误拼可以忽略。

## 2. 修改默认端口
这里需要修改节点间通讯的端口以及服务端与客户端之间通讯的 RCP 端口。

**注：比特币的源码根目录为 `bitcoin/src`，之后文件的位置均以 `src` 为根目录。**

### 2.1. 修改节点间通讯的端口
节点间通讯的端口硬编在“chainparams.cpp”文件的 3 个网络类 CMainParams（主网）、CTestNetParams（测试网）和 CRegTestParams（回归测试网） 的默认构造函数中。<br>
主网和测试网均为公网，而回归测试网是用于给开发者提供测试的私网。<br>
下面均以**主网**为例进行修改。

{% highlight C++ %}
/**
 * Main network
 */
/**
 * What makes a good checkpoint block?
 * + Is surrounded by blocks with reasonable timestamps
 *   (no blocks before with a timestamp after, none after with
 *    timestamp before)
 * + Contains no strange transactions
 */

class CMainParams : public CChainParams {
public:
    CMainParams() {
        ...
        nDefaultPort = 8333; // 这个是默认端口，改为其它合适的端口，例："8331"
        ...
        };
    }
};
{% endhighlight %}

### 2.2. 修改服务端（d）与客户端（cli）通讯的 RCP 端口

RPC 端口硬编在“chainparamsbase.cpp”文件中。

{% highlight C++ %}
/**
 * Main network
 */
class CBaseMainParams : public CBaseChainParams
{
public:
    CBaseMainParams()
    {
        nRPCPort = 8332; // 改为其它合适的端口，例："8330"
    }
};
{% endhighlight %}

**注：nRPCPort 和 nDefaultPort 不能相同，否则会导致默认端口被占用而绑定失败。**

## 3. 修改网络协议魔数

所谓协议魔术就是网络中节点之间传递消息的头，类似于 TCP 协议 20 个字节的报头和 IP 20 个字节的报头，
这里是 4 个字节的消息头，使用一个特殊的不同于该魔数的消息头即可。

{% highlight C++ %}
class CMainParams : public CChainParams {
public:
    CMainParams() {
        ...
        /**
         * The message start string is designed to be unlikely to occur in normal data.
         * The characters are rarely used upper ASCII, not valid as UTF-8, and produce
         * a large 32-bit integer with any alignment.
         */ // 随意设置           例：0xcafecafe
        pchMessageStart[0] = 0xf9; // 0xca
        pchMessageStart[1] = 0xbe; // 0xfe
        pchMessageStart[2] = 0xb4; // 0xca
        pchMessageStart[3] = 0xd9; // 0xfe
        ...
        };
    }
};
{% endhighlight %}

可以取一个有意义的单词作为魔数，例如：`0xcafecafe`。<br>
或使用下面命令：

{% highlight shell %}
$ echo $RANDOM
{% endhighlight %}

获取随机数随机数的后三位（当 <= 255 时取值）转换为 16 进制，取 4 个作为魔数。

## 4. 修改创世区块信息

要修改的创世区块的信息有：创建时间（时间戳），随机数（挖到块时），难度（影响挖矿速度），版本（一般不变），奖励（会影响货币的发行量），还有文字版时间戳。<br>
关于区块的构造，详见[比特币源码剖析 - 区块]()。下面是在“chainparams.cpp”文件中硬编的创世区块：

{% highlight C++ %}
...
/**
 * Build the genesis block. Note that the output of its generation
 * transaction cannot be spent since it did not originally exist in the
 * database.
 *
 * CBlock(hash=000000000019d6, ver=1, hashPrevBlock=00000000000000, hashMerkleRoot=4a5e1e, nTime=1231006505, nBits=1d00ffff, nNonce=2083236893, vtx=1)
 *   CTransaction(hash=4a5e1e, ver=1, vin.size=1, vout.size=1, nLockTime=0)
 *     CTxIn(COutPoint(000000, -1), coinbase 04ffff001d0104455468652054696d65732030332f4a616e2f32303039204368616e63656c6c6f72206f6e206272696e6b206f66207365636f6e64206261696c6f757420666f722062616e6b73)
 *     CTxOut(nValue=50.00000000, scriptPubKey=0x5F1DF16B2B704C8A578D0B)
 *   vMerkleTree: 4a5e1e
 */
static CBlock CreateGenesisBlock(uint32_t nTime, uint32_t nNonce, uint32_t nBits, int32_t nVersion, const CAmount& genesisReward)
{
    const char* pszTimestamp = "The Times 03/Jan/2009 Chancellor on brink of second bailout for banks"; // 这里就是中本聪在创世区块中留下的泰晤士报的标题
    const CScript genesisOutputScript = CScript() << ParseHex("04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f") << OP_CHECKSIG;
    return CreateGenesisBlock(pszTimestamp, genesisOutputScript, nTime, nNonce, nBits, nVersion, genesisReward);
}
...
class CMainParams : public CChainParams {
public:
    CMainParams() {
        ... // 这里硬编的创世区块参数：创建时间，随机数，初始挖矿难度，版本号，奖励
        genesis = CreateGenesisBlock(1231006505, 2083236893, 0x1d00ffff, 1, 50 * COIN); // 这里创建创世区块
        consensus.hashGenesisBlock = genesis.GetHash(); // 并增加创世区块的哈希到共识对象
        assert(consensus.hashGenesisBlock == uint256S("0x000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f")); // 检验创世区块哈希
        assert(genesis.hashMerkleRoot == uint256S("0x4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b")); // 检验区块默尔克树根哈希
        ...
        };
    }
};
{% endhighlight %}

1.获取当前的 UNIX 时间戳使用如下命令：

{% highlight shell %}
$ date +%s
{% endhighlight %}

2.随机数置为 0，为挖创世区块做准备。<br>
3.难度可以设为回归测试网难度 0x207fffff（秒出块）。<br>
4.版本一般不变。<br>
5.奖励根据货币发行量来更改。<br>
6.文字版时间戳使用大多数人都知道可以成为历史（可以追溯）事件，例：较有名的报纸当日的新闻标题。

## 5. 修改公钥地址前缀

{% highlight C++ %}
class CMainParams : public CChainParams {
public:
    CMainParams() {
        ...
        base58Prefixes[PUBKEY_ADDRESS] = std::vector<unsigned char>(1,0); // 公钥地址前缀，10 进制的 0 对应 base58 编码的 1
        base58Prefixes[SCRIPT_ADDRESS] = std::vector<unsigned char>(1,5); // 脚本地址前缀，10 进制的 5 对应 base58 编码的 3
        base58Prefixes[SECRET_KEY] =     std::vector<unsigned char>(1,128); // 密（私）钥前缀，10 进制的 128 对用 base58 编码的 K 或 L（压缩的私钥）
        base58Prefixes[EXT_PUBLIC_KEY] = boost::assign::list_of(0x04)(0x88)(0xB2)(0x1E).convert_to_container<std::vector<unsigned char> >();
        base58Prefixes[EXT_SECRET_KEY] = boost::assign::list_of(0x04)(0x88)(0xAD)(0xE4).convert_to_container<std::vector<unsigned char> >();
        ...
        };
    }
};
{% endhighlight %}

PUBKEY_ADDRESS 前缀对应的 10 进制根据 [List of address prefixes - Bitcoin Wiki](https://en.bitcoin.it/wiki/List_of_address_prefixes) 进行修改。<br>
例：把比特币的公钥地址前缀 `1` 改为 `C`，通过查表得到 `C` 对应的 10 进制为 28，所以直接把 `std::vector<unsigned char>(1,0)` 改为 `std::vector<unsigned char>(1,28)`。

**注：<br>
1. 公钥地址和脚本地址以及私钥均采用 base58 编码后显示，方便人类使用，详见[Base58 编码](/2018/05/12/base58-encoding)。<br>
2. 这里的前缀只有一个字符，若是超过 1 个字符长度的前缀，可以参考[比特币“靓号”地址](/2018/05/16/bitcoin-vanity-address)。**

## 6. 修改 DNS 种子

比特币节点通过 DNS 种子来获取并连接到正在运行的节点（区块链网络），
连接之后节点交换它们已建立连接的节点地址。
如果没有该种子可以直接注释掉这部分。

{% highlight C++ %}
class CMainParams : public CChainParams {
public:
    CMainParams() {
        ...
        vSeeds.push_back(CDNSSeedData("bitcoin.sipa.be", "seed.bitcoin.sipa.be")); // Pieter Wuille
        vSeeds.push_back(CDNSSeedData("bluematt.me", "dnsseed.bluematt.me")); // Matt Corallo
        vSeeds.push_back(CDNSSeedData("dashjr.org", "dnsseed.bitcoin.dashjr.org")); // Luke Dashjr
        vSeeds.push_back(CDNSSeedData("bitcoinstats.com", "seed.bitcoinstats.com")); // Christian Decker
        vSeeds.push_back(CDNSSeedData("xf2.org", "bitseed.xf2.org")); // Jeff Garzik
        vSeeds.push_back(CDNSSeedData("bitcoin.jonasschnelli.ch", "seed.bitcoin.jonasschnelli.ch")); // Jonas Schnelli
        ... 
        vFixedSeeds = std::vector<SeedSpec6>(pnSeed6_main, pnSeed6_main + ARRAYLEN(pnSeed6_main));
        ...
        };
    }
};
{% endhighlight %}

## 7. 修改检测点

检测点用于验证区块链上的区块，是一个 kv 键值对，对应区块号（高度）和区块哈希。<br>
检测点数据对象包含检测点映射列表、最后一个检测点区块的时间戳、创世区块和最后一个检测点区块间的总交易数以及对以后每天交易量的估计值。
以下是主网中硬编的监测点对象：

{% highlight C++ %}
class CMainParams : public CChainParams {
public:
    CMainParams() {
        ...
        checkpointData = (CCheckpointData) {
            boost::assign::map_list_of
            ( 11111, uint256S("0x0000000069e244f73d78e8fd29ba2fd2ed618bd6fa2ee92559f542fdb26e7c1d"))
            ( 33333, uint256S("0x000000002dd5588a74784eaa7ab0507a18ad16a236e7b1ce69f00d7ddfb5d0a6"))
            ( 74000, uint256S("0x0000000000573993a3c9e41ce34471c079dcf5f52a0e824a81e7f953b8661a20"))
            (105000, uint256S("0x00000000000291ce28027faea320c8d2b054b2e0fe44a773f3eefb151d6bdc97"))
            (134444, uint256S("0x00000000000005b12ffd4cd315cd34ffd4a594f430ac814c91184a0d42d2b0fe"))
            (168000, uint256S("0x000000000000099e61ea72015e79632f216fe6cb33d7899acb35b75c8303b763"))
            (193000, uint256S("0x000000000000059f452a5f7340de6682a977387c17010ff6e6c3bd83ca8b1317"))
            (210000, uint256S("0x000000000000048b95347e83192f69cf0366076336c639f9b7228e9ba171342e"))
            (216116, uint256S("0x00000000000001b4f4b433e81ee46494af945cf96014816a4e2370f11b23df4e"))
            (225430, uint256S("0x00000000000001c108384350f74090433e7fcf79a606b8e797f065b130575932"))
            (250000, uint256S("0x000000000000003887df1f29024b06fc2200b55f8af8f35453d7be294df2d214"))
            (279000, uint256S("0x0000000000000001ae8c72a0b0c301f67e3afca10e819efa9041e458e9bd7e40"))
            (295000, uint256S("0x00000000000000004d9b4ef50f0f9d686fd69db2e03af35a100370c64632a983")),
            1397080064, // * UNIX timestamp of last checkpoint block
            36544669,   // * total number of transactions between genesis and last checkpoint
                        //   (the tx=... number in the SetBestChain debug.log lines)
            60000.0     // * estimated number of transactions per day after checkpoint
        };
        ...
        };
    }
};
{% endhighlight %}

1. 把检测点列表删除，增加创世区块的哈希到该列表。
2. 填入创世区块的创建时间。
3. 交易数为 0。
4. 估计交易数为 500（这个值随意填）。

**注：检测点的信息可随区块链的延伸不断更新。**

## 参照
* [How to make an altcoin \| Bear's Den](http://dillingers.com/blog/2015/04/18/how-to-make-an-altcoin)
* [如何仿照比特币创造自己的山寨币 \| Sunny's Blog](http://shusunny.github.io)
* [手动做一个自己的 COIN 客户端：附区块链核心代码解读](http://gitbook.cn/books/5a9792622afd2366fc629d01/index.html)
* [【比特币】自己动手制作山寨币 - 区块链知识库](http://lib.csdn.net/article/blockchain/45844)
* [从 0 到 1 建立自己的区块链 \| 巴比特](http://www.8btc.com/build-your-own-blockchain)
* [比特币源代码阅读笔记-创世块的产生_易安定_新浪博客](http://blog.sina.com.cn/s/blog_5922b3960101s3ap.html)
* [...](https://github.com/mistydew/blockchain)
