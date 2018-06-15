---
layout: post
title:  "比特币 RPC 命令剖析 \"estimatesmartfee\""
date:   2018-06-15 14:25:15 +0800
author: mistydew
categories: Blockchain
---
![bitcoin](/images/20180504/bitcoin.svg)

## 读在前面
比特币相关的解读目前均采用 [bitcoin v0.12.1](https://github.com/bitcoin/bitcoin/tree/v0.12.1)，此版本为官方内置挖矿算法的最后一版。<br>
目前比特币的最新版本为 bitcoin v0.16.0，离区块链 1.0 落地还有些距离。

## 提示说明

{% highlight shell %}
estimatesmartfee nblocks # 估计交易在 nblocks 个区块开始确认的每千字节的大致交易费用，如果可能则返回估计有效的区块数
{% endhighlight %}

**警告：该接口不稳定且可能消失或改变！**

参数：<br>
1. `nblocks` （数字型）区块数量。

结果：<br>
{% highlight shell %}
{
  "feerate" : x.x,     (numeric) estimate fee-per-kilobyte (in BTC)
  "blocks" : n         (numeric) block number where estimate was found
}
{% endhighlight %}

**注：如果没有足够的交易和区块用来做估算任意数量的区块，则返回一个负值。<br>
但是不会返回低于交易内存池拒收费用的值。**

## 用法示例

{% highlight shell %}
$ bitcoin-cli estimatesmartfee 6
{
  "feerate": -1,
  "blocks": 25
}
{% endhighlight %}

## 源码剖析
`estimatesmartfee` 对应的函数在“rpcserver.h”文件中被引用。

{% highlight C++ %}
extern UniValue estimatesmartfee(const UniValue& params, bool fHelp); // 智能估计交易费
{% endhighlight %}

实现在“rpcmining.cpp”文件中。

{% highlight C++ %}
UniValue estimatesmartfee(const UniValue& params, bool fHelp)
{
    if (fHelp || params.size() != 1) // 参数必须为 1 个
        throw runtime_error( // 命令帮助反馈
            "estimatesmartfee nblocks\n"
            "\nWARNING: This interface is unstable and may disappear or change!\n"
            "\nEstimates the approximate fee per kilobyte needed for a transaction to begin\n"
            "confirmation within nblocks blocks if possible and return the number of blocks\n"
            "for which the estimate is valid.\n"
            "\nArguments:\n"
            "1. nblocks     (numeric)\n"
            "\nResult:\n"
            "{\n"
            "  \"feerate\" : x.x,     (numeric) estimate fee-per-kilobyte (in BTC)\n"
            "  \"blocks\" : n         (numeric) block number where estimate was found\n"
            "}\n"
            "\n"
            "A negative value is returned if not enough transactions and blocks\n"
            "have been observed to make an estimate for any number of blocks.\n"
            "However it will not return a value below the mempool reject fee.\n"
            "\nExample:\n"
            + HelpExampleCli("estimatesmartfee", "6")
            );

    RPCTypeCheck(params, boost::assign::list_of(UniValue::VNUM)); // 参数类型检查

    int nBlocks = params[0].get_int(); // 获取指定的区块数

    UniValue result(UniValue::VOBJ);
    int answerFound; // 保存估计有效的块数
    CFeeRate feeRate = mempool.estimateSmartFee(nBlocks, &answerFound); // 智能估算交易费
    result.push_back(Pair("feerate", feeRate == CFeeRate(0) ? -1.0 : ValueFromAmount(feeRate.GetFeePerK()))); // 交易费
    result.push_back(Pair("blocks", answerFound)); // 有效的区块数
    return result; // 返回结果
}
{% endhighlight %}

基本流程：<br>
1.处理命令帮助和参数个数。<br>
2.参数类型检查。<br>
3.获取指定的块数。<br>
4.智能估算交易费，同时获取估算有效的区块数。<br>
5.把交易费和有效区块数加入结果对象并返回。

Thanks for your time.

## 参照
* [Developer Documentation - Bitcoin](https://bitcoin.org/en/developer-documentation)
* [Bitcoin Developer Reference - Bitcoin](https://bitcoin.org/en/developer-reference#estimatesmartfee)
* [精通比特币（第二版） \| 巴比特图书](http://book.8btc.com/masterbitcoin2cn)
* [...](https://github.com/mistydew/blockchain)