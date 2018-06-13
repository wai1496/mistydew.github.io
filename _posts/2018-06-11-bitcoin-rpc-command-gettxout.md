---
layout: post
title:  "比特币 RPC 命令剖析 \"gettxout\""
date:   2018-06-11 09:26:01 +0800
author: mistydew
categories: Blockchain
---
![bitcoin](/images/20180504/bitcoin.svg)

## 读在前面
比特币相关的解读目前均采用 [bitcoin v0.12.1](https://github.com/bitcoin/bitcoin/tree/v0.12.1)，此版本为官方内置挖矿算法的最后一版。<br>
目前比特币的最新版本为 bitcoin v0.16.0，离区块链 1.0 落地还有些距离。

## 提示说明

{% highlight shell %}
gettxout "txid" n ( includemempool ) # 获取关于一笔未花费交易输出的细节
{% endhighlight %}

参数：<br>
1. `txid` （字符串，必备）交易索引。<br>
2. `n` （数字型，必备）输出值（索引）。<br>
3. `includemempool` （布尔型，可选）是否在交易内存池中。

结果：<br>
{% highlight shell %}
{
  "bestblock" : "hash",    (string) the block hash
  "confirmations" : n,       (numeric) The number of confirmations
  "value" : x.xxx,           (numeric) The transaction value in BTC
  "scriptPubKey" : {         (json object)
     "asm" : "code",       (string) 
     "hex" : "hex",        (string) 
     "reqSigs" : n,          (numeric) Number of required signatures
     "type" : "pubkeyhash", (string) The type, eg pubkeyhash
     "addresses" : [          (array of string) array of bitcoin addresses
        "bitcoinaddress"     (string) bitcoin address
        ,...
     ]
  },
  "version" : n,            (numeric) The version
  "coinbase" : true|false   (boolean) Coinbase or not
}
{% endhighlight %}

## 用法示例

先使用 [`listunspent`](/2018/06/05/bitcoin-rpc-command-listunspent) 命令列出未花费交易输出，
再通过交易索引和交易输出索引获取该交易输出的详细信息。

{% highlight shell %}
$ bitcoin-cli listunspent
[
  ...
  {
    "txid": "5d306125b2fbfc5855b1b7729ceac1b3010e0ddaa7b03f7abeb225f7b13677ff",
    "vout": 0,
    "address": "1kjTv8TKSsbpGEBVZqLTcx1MeA4G8JkCnk",
    "account": "",
    "scriptPubKey": "76a914a4d938a6461a0d6f24946b9bfcda0862a1db6f7488ac",
    "amount": 0.10000000,
    "confirmations": 48523,
    "ps_rounds": -2,
    "spendable": true,
    "solvable": true
  }
]
$ bitcoin-cli gettxout 5d306125b2fbfc5855b1b7729ceac1b3010e0ddaa7b03f7abeb225f7b13677ff 0
{
  "bestblock": "000057a00444aefd85a8c9207c2daf292c86cf3dac05158baa6079a1668e7981",
  "confirmations": 48528,
  "value": 0.10000000,
  "scriptPubKey": {
    "asm": "OP_DUP OP_HASH160 a4d938a6461a0d6f24946b9bfcda0862a1db6f74 OP_EQUALVERIFY OP_CHECKSIG",
    "hex": "76a914a4d938a6461a0d6f24946b9bfcda0862a1db6f7488ac",
    "reqSigs": 1,
    "type": "pubkeyhash",
    "addresses": [
      "1kjTv8TKSsbpGEBVZqLTcx1MeA4G8JkCnk"
    ]
  },
  "coinbase": false
}
{% endhighlight %}

## 源码剖析
`gettxout` 对应的函数在“rpcserver.h”文件中被引用。

{% highlight C++ %}
extern UniValue gettxout(const UniValue& params, bool fHelp); // 获取一笔交易输出（链上或内存池中）的细节
{% endhighlight %}

实现在“rpcblockchain.cpp”文件中。

{% highlight C++ %}
UniValue gettxout(const UniValue& params, bool fHelp)
{
    if (fHelp || params.size() < 2 || params.size() > 3) // 参数为 2 个或 3 个
        throw runtime_error( // 命令帮助反馈
            "gettxout \"txid\" n ( includemempool )\n"
            "\nReturns details about an unspent transaction output.\n"
            "\nArguments:\n"
            "1. \"txid\"       (string, required) The transaction id\n"
            "2. n              (numeric, required) vout value\n"
            "3. includemempool  (boolean, optional) Whether to included the mem pool\n"
            "\nResult:\n"
            "{\n"
            "  \"bestblock\" : \"hash\",    (string) the block hash\n"
            "  \"confirmations\" : n,       (numeric) The number of confirmations\n"
            "  \"value\" : x.xxx,           (numeric) The transaction value in " + CURRENCY_UNIT + "\n"
            "  \"scriptPubKey\" : {         (json object)\n"
            "     \"asm\" : \"code\",       (string) \n"
            "     \"hex\" : \"hex\",        (string) \n"
            "     \"reqSigs\" : n,          (numeric) Number of required signatures\n"
            "     \"type\" : \"pubkeyhash\", (string) The type, eg pubkeyhash\n"
            "     \"addresses\" : [          (array of string) array of bitcoin addresses\n"
            "        \"bitcoinaddress\"     (string) bitcoin address\n"
            "        ,...\n"
            "     ]\n"
            "  },\n"
            "  \"version\" : n,            (numeric) The version\n"
            "  \"coinbase\" : true|false   (boolean) Coinbase or not\n"
            "}\n"

            "\nExamples:\n"
            "\nGet unspent transactions\n"
            + HelpExampleCli("listunspent", "") +
            "\nView the details\n"
            + HelpExampleCli("gettxout", "\"txid\" 1") +
            "\nAs a json rpc call\n"
            + HelpExampleRpc("gettxout", "\"txid\", 1")
        );

    LOCK(cs_main);

    UniValue ret(UniValue::VOBJ); // 目标类型的结果对象

    std::string strHash = params[0].get_str(); // 获取交易索引
    uint256 hash(uint256S(strHash));
    int n = params[1].get_int(); // 获取交易输出索引
    bool fMempool = true; // 是否包含内存池内交易的标志，默认为 true
    if (params.size() > 2)
        fMempool = params[2].get_bool(); // 获取指定的内存池标志

    CCoins coins; // 创建一个被修剪的交易版本对象（只包含元数据和未花费的交易输出）
    if (fMempool) { // 若包含内存池中的交易
        LOCK(mempool.cs); // 内存池上锁
        CCoinsViewMemPool view(pcoinsTip, mempool); // 创建内存池引用查看对象
        if (!view.GetCoins(hash, coins)) // 获取修剪版交易
            return NullUniValue;
        mempool.pruneSpent(hash, coins); // TODO: this should be done by the CCoinsViewMemPool
    } else { // 若不包含内存池的交易
        if (!pcoinsTip->GetCoins(hash, coins)) // 直接获取真正缓存的币数据
            return NullUniValue;
    }
    if (n<0 || (unsigned int)n>=coins.vout.size() || coins.vout[n].IsNull()) // 输出索引范围检测，或该索引对应输出为空
        return NullUniValue;

    BlockMap::iterator it = mapBlockIndex.find(pcoinsTip->GetBestBlock()); // 获取最佳区块索引映射迭代器
    CBlockIndex *pindex = it->second; // 获取最佳区块索引
    ret.push_back(Pair("bestblock", pindex->GetBlockHash().GetHex())); // 最佳区块哈希
    if ((unsigned int)coins.nHeight == MEMPOOL_HEIGHT) // 若币的高度为 0x7FFFFFFF
        ret.push_back(Pair("confirmations", 0)); // 未上链，0 确认数
    else // 否则表示已上链
        ret.push_back(Pair("confirmations", pindex->nHeight - coins.nHeight + 1)); // 获取确认数
    ret.push_back(Pair("value", ValueFromAmount(coins.vout[n].nValue))); // 输出金额
    UniValue o(UniValue::VOBJ);
    ScriptPubKeyToJSON(coins.vout[n].scriptPubKey, o, true); // 公钥脚本转换为 JSON 格式
    ret.push_back(Pair("scriptPubKey", o)); // 公钥脚本
    ret.push_back(Pair("version", coins.nVersion)); // 版本号
    ret.push_back(Pair("coinbase", coins.fCoinBase)); // 是否为创币交易

    return ret;
}
{% endhighlight %}

基本流程：<br>
1.处理命令帮助和参数个数。<br>
2.上锁。<br>
3.获取相关参数：交易索引，交易输出索引，是否包含交易内存池标志。<br>
4.若包含交易内存池，则获取修剪版本交易，并标记该交易输出已花费。<br>
5.若不包含，直接获取缓存的币数据。<br>
6.交易输出索引检测。<br>
7.获取相关信息并追加到结果对象。<br>
8.返回结果。

Thanks for your time.

## 参照
* [Developer Documentation - Bitcoin](https://bitcoin.org/en/developer-documentation)
* [Bitcoin Developer Reference - Bitcoin](https://bitcoin.org/en/developer-reference#gettxout)
* [精通比特币（第二版） \| 巴比特图书](http://book.8btc.com/masterbitcoin2cn)
* [...](https://github.com/mistydew/blockchain)