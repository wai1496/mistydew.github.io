---
layout: post
title:  "比特币 RPC 命令剖析 \"listbanned\""
date:   2018-05-29 08:52:09 +0800
author: mistydew
categories: Blockchain
---
![bitcoin](/images/20180504/bitcoin.svg)

## 读在前面
比特币相关的解读目前均采用 [bitcoin v0.12.1](https://github.com/bitcoin/bitcoin/tree/v0.12.1)，此版本为官方内置挖矿算法的最后一版。<br>
目前比特币的最新版本为 bitcoin v0.16.0，离区块链 1.0 落地还有些距离。

## 提示说明

{% highlight shell %}
listbanned # 列出所有禁止的 IP/子网
{% endhighlight %}

结果：以 JSON 数组的形式返回所有被禁止的 IP。

## 用法示例

### 比特币核心客户端

显示服务器黑名单（被禁止的 IP 合集）。

{% highlight shell %}
$ bitcoin-cli listbanned
[
  {
    "address": "192.168.0.2/32",
    "banned_until": 1530079566,
    "ban_created": 1529993166,
    "ban_reason": "manually added"
  }
]
{% endhighlight %}

### cURL

{% highlight shell %}
$ curl --user myusername:mypassword --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "listbanned", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:8332/
{"result":[{"address":"192.168.0.2/32","banned_until":1530079566,"ban_created":1529993166,"ban_reason":"manually added"}],"error":null,"id":"curltest"}
{% endhighlight %}

## 源码剖析
`listbanned` 对应的函数在“rpcserver.h”文件中被引用。

{% highlight C++ %}
extern UniValue listbanned(const UniValue& params, bool fHelp); // 列出黑名单
{% endhighlight %}

实现在“rpcnet.cpp”文件中。

{% highlight C++ %}
UniValue listbanned(const UniValue& params, bool fHelp)
{
    if (fHelp || params.size() != 0) // 没有参数
        throw runtime_error( // 命令帮助反馈
                            "listbanned\n"
                            "\nList all banned IPs/Subnets.\n"
                            "\nExamples:\n"
                            + HelpExampleCli("listbanned", "")
                            + HelpExampleRpc("listbanned", "")
                            );

    banmap_t banMap;
    CNode::GetBanned(banMap); // 获取禁止列表

    UniValue bannedAddresses(UniValue::VARR); // 创建数组类型的禁止地址
    for (banmap_t::iterator it = banMap.begin(); it != banMap.end(); it++)
    { // 遍历禁止列表
        CBanEntry banEntry = (*it).second; // 获取禁止条目
        UniValue rec(UniValue::VOBJ);
        rec.push_back(Pair("address", (*it).first.ToString())); // 子网地址
        rec.push_back(Pair("banned_until", banEntry.nBanUntil)); // 禁止结束时间
        rec.push_back(Pair("ban_created", banEntry.nCreateTime)); // 创建禁止时间
        rec.push_back(Pair("ban_reason", banEntry.banReasonToString())); // 禁止原因

        bannedAddresses.push_back(rec); // 加入结果集
    }

    return bannedAddresses; // 返回禁止列表
}
{% endhighlight %}

基本流程：<br>
1.处理命令帮助和参数个数。<br>
2.获取禁止列表。<br>
3.遍历该列表获取需要的信息。<br>
4.返回获取的信息。

类型 banmap_t 的定义在“net.h”文件中。

{% highlight C++ %}
typedef std::map<CSubNet, CBanEntry> banmap_t; // 禁止列表：子网与禁止条目的映射
{% endhighlight %}

函数 CNode::GetBanned(banMap) 声明在“net.h”文件的 CNode 类中。

{% highlight C++ %}
/** Information about a peer */
class CNode // 关于同辈的信息
{
    ...
    static void GetBanned(banmap_t &banmap);
    ...
};
{% endhighlight %}

实现在“net.cpp”文件中。

{% highlight C++ %}
void CNode::GetBanned(banmap_t &banMap)
{
    LOCK(cs_setBanned);
    banMap = setBanned; //create a thread safe copy
}
{% endhighlight %}

对象 setBanned 是静态的屏蔽地址列表，所有被禁止的地址会添加到该对象，它定义在“net.h”文件的 CNode 类中。

{% highlight C++ %}
class CNode // 关于同辈的信息
{
    ...
protected:

    // Denial-of-service detection/prevention
    // Key is IP address, value is banned-until-time
    static banmap_t setBanned;
    static CCriticalSection cs_setBanned;
    ...
};
{% endhighlight %}

初始化在“net.cpp”文件中。

{% highlight C++ %}
banmap_t CNode::setBanned; // 设置屏蔽地址列表
CCriticalSection CNode::cs_setBanned;
{% endhighlight %}

Thanks for your time.

## 参照
* [Developer Documentation - Bitcoin](https://bitcoin.org/en/developer-documentation)
* [Bitcoin Developer Reference - Bitcoin](https://bitcoin.org/en/developer-reference#listbanned)
* [JSON 数组 \| 菜鸟教程](http://www.runoob.com/json/js-json-arrays.html)
* [精通比特币（第二版） \| 巴比特图书](http://book.8btc.com/masterbitcoin2cn)
* [...](https://github.com/mistydew/blockchain)
