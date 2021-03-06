---
layout: post
title:  "比特币 RPC 命令剖析 \"getdifficulty\""
date:   2018-05-22 15:41:33 +0800
author: mistydew
categories: Blockchain
---
![bitcoin](/images/20180504/bitcoin.svg)

## 读在前面
比特币相关的解读目前均采用 [bitcoin v0.12.1](https://github.com/bitcoin/bitcoin/tree/v0.12.1)，此版本为官方内置挖矿算法的最后一版。<br>
目前比特币的最新版本为 bitcoin v0.16.0，离区块链 1.0 落地还有些距离。

## 提示说明

{% highlight shell %}
getdifficulty # 获取作为最低难度 `1` 倍数的工作量证明难度
{% endhighlight %}

结果：（数字）返回作为最低难度 `1` 倍数的工作量证明难度。

## 用法示例

### 比特币核心客户端

{% highlight shell %}
$ bitcoin-cli getdifficulty
0.001533333096242079
{% endhighlight %}

### cURL

{% highlight shell %}
$ curl --user myusername:mypassword --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getdifficulty", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:8332/
{"result":0.001532956637923291,"error":null,"id":"curltest"}
{% endhighlight %}

## 源码剖析
`getdifficulty` 对应的函数在“rpcserver.h”文件中被引用。

{% highlight C++ %}
extern UniValue getdifficulty(const UniValue& params, bool fHelp); // 获取当前挖矿难度
{% endhighlight %}

实现在“rpcblockchain.cpp”文件中。

{% highlight C++ %}
UniValue getdifficulty(const UniValue& params, bool fHelp)
{
    if (fHelp || params.size() != 0) // 没有参数
        throw runtime_error( // 命令帮助反馈
            "getdifficulty\n"
            "\nReturns the proof-of-work difficulty as a multiple of the minimum difficulty.\n"
            "\nResult:\n"
            "n.nnn       (numeric) the proof-of-work difficulty as a multiple of the minimum difficulty.\n"
            "\nExamples:\n"
            + HelpExampleCli("getdifficulty", "")
            + HelpExampleRpc("getdifficulty", "")
        );

    LOCK(cs_main); // 上锁
    return GetDifficulty(); // 返回获取的难度值
}
{% endhighlight %}

基本流程：<br>
1.处理命令帮助和参数个数。<br>
2.上锁。<br>
3.获取当前难度并返回。

第三步，调用 GetDifficulty() 函数获取当前难度，该函数实现在“rpcblockchain.cpp”文件中。

{% highlight C++ %}
double GetDifficulty(const CBlockIndex* blockindex)
{
    // Floating point number that is a multiple of the minimum difficulty, // 最小难度倍数的浮点数
    // minimum difficulty = 1.0. // 最小难度 = 1.0
    if (blockindex == NULL)
    {
        if (chainActive.Tip() == NULL) // 链尖为空
            return 1.0; // 返回最小难度
        else
            blockindex = chainActive.Tip(); // 获取链尖区块索引
    }

    int nShift = (blockindex->nBits >> 24) & 0xff; // 获取 nBits 的高 8 位 2 进制

    double dDiff = // main and testnet (0x1d00ffff) or regtest (0x207fffff) 0x1e0ffff0 (dash)
        (double)0x0000ffff / (double)(blockindex->nBits & 0x00ffffff); // 计算难度

    while (nShift < 29)
    {
        dDiff *= 256.0;
        nShift++;
    }
    while (nShift > 29) // main and testnet (0x1d, 29) or regtest (0x20, 32)
    {
        dDiff /= 256.0;
        nShift--;
    }

    return dDiff; // 返回难度
}
{% endhighlight %}

Thanks for your time.

## 参照
* [Developer Documentation - Bitcoin](https://bitcoin.org/en/developer-documentation)
* [Bitcoin Developer Reference - Bitcoin](https://bitcoin.org/en/developer-reference#getdifficulty)
* [精通比特币（第二版） \| 巴比特图书](http://book.8btc.com/masterbitcoin2cn)
* [...](https://github.com/mistydew/blockchain)
