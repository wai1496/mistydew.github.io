---
layout: post
title:  "比特币 RPC 命令剖析 \"setmocktime\""
date:   2018-06-14 11:02:08 +0800
author: mistydew
categories: Blockchain
hidden: true
---
![bitcoin](/images/20180504/bitcoin.svg)

## 读在前面
比特币相关的解读目前均采用 [bitcoin v0.12.1](https://github.com/bitcoin/bitcoin/tree/v0.12.1)，此版本为官方内置挖矿算法的最后一版。<br>
目前比特币的最新版本为 bitcoin v0.16.0，离区块链 1.0 落地还有些距离。

## 提示说明

{% highlight shell %}
setmocktime timestamp # 设置本地时间为给定时间戳（仅限回归测试模式）
{% endhighlight %}

参数：<br>
1. `timestamp` （整型，必备）Unix 从 1970-01-01 00:00:00 UTC 开始的以秒为单位的时间戳通过 0 返回到使用系统时间。

结果：无返回值。

## 用法示例

{% highlight shell %}
$ bitcoin-cli setmocktime 1528946331
{% endhighlight %}

## 源码剖析
`setmocktime` 对应的函数在“rpcserver.h”文件中被引用。

{% highlight C++ %}
extern UniValue setmocktime(const UniValue& params, bool fHelp); // 设置 mocktime
{% endhighlight %}

实现在“rpcmisc.cpp”文件中。

{% highlight C++ %}
UniValue setmocktime(const UniValue& params, bool fHelp)
{
    if (fHelp || params.size() != 1) // 参数必须为 1 个
        throw runtime_error( // 命令帮助反馈
            "setmocktime timestamp\n"
            "\nSet the local time to given timestamp (-regtest only)\n"
            "\nArguments:\n"
            "1. timestamp  (integer, required) Unix seconds-since-epoch timestamp\n"
            "   Pass 0 to go back to using the system time."
        );

    if (!Params().MineBlocksOnDemand()) // 网络必须为 regtest 回归测试模式
        throw runtime_error("setmocktime for regression testing (-regtest mode) only");

    // cs_vNodes is locked and node send/receive times are updated
    // atomically with the time change to prevent peers from being
    // disconnected because we think we haven't communicated with them
    // in a long time.
    LOCK2(cs_main, cs_vNodes); // 节点列表上锁

    RPCTypeCheck(params, boost::assign::list_of(UniValue::VNUM)); // 参数类型检查，整型
    SetMockTime(params[0].get_int64()); // 获取指定时间戳并设置本地 mock 时间

    uint64_t t = GetTime(); // 获取当前时间
    BOOST_FOREACH(CNode* pnode, vNodes) { // 遍历已建立链接的节点列表
        pnode->nLastSend = pnode->nLastRecv = t; // 设置节点最后一次发送和接收的时间
    }

    return NullUniValue; // 返回空值
}
{% endhighlight %}

基本流程：<br>
1.处理命令帮助和参数个数。<br>
2.检查网络模式，必须为回归测试模式。<br>
3.已建立连接的节点上锁。<br>
4.检查参数类型，获取该参数并设置本地 Mock 时间。<br>
5.获取当前时间，并遍历已建立连接的节点列表，设置最后一次发送和接收的时间为当前时间。<br>
6.返回空值。

第四步，调用 SetMockTime(params[0].get_int64()) 设置 MockTime，该函数声明在“utiltime.h”文件中。

{% highlight C++ %}
void SetMockTime(int64_t nMockTimeIn); // 设置 Mock 时间
{% endhighlight %}

实现在“utiltime.cpp”文件中。

{% highlight C++ %}
static int64_t nMockTime = 0;  //! For unit testing // 用于单元测试
...
void SetMockTime(int64_t nMockTimeIn)
{
    nMockTime = nMockTimeIn; // 设置指定的 mocktime
}
{% endhighlight %}

Thanks for your time.

## 参照
* [Developer Documentation - Bitcoin](https://bitcoin.org/en/developer-documentation)
* [Bitcoin Developer Reference - Bitcoin](https://bitcoin.org/en/developer-reference#setmocktime)
* [精通比特币（第二版） \| 巴比特图书](http://book.8btc.com/masterbitcoin2cn)
* [...](https://github.com/mistydew/blockchain)