---
layout: post
title:  "比特币“靓号”地址"
date:   2018-05-16 18:56:51 +0800
author: mistydew
categories: Blockchain
---
![bitcoin](/images/20180504/bitcoin.svg)

## 读在前面
比特币相关的解读目前均采用 [bitcoin v0.12.1](https://github.com/bitcoin/bitcoin/tree/v0.12.1)，此版本为官方内置挖矿算法的最后一版。<br>
目前比特币的最新版本为 bitcoin v0.16.0，离区块链 1.0 落地还有些距离。

## 概要
比特币“靓号”地址是拥有个性化（自定义）前缀的公钥地址。<br>
例如以 `1kid` 开头的公钥地址 `1kidyp7EFY3xUdMGSTWpkEmLcfKu9yvoq`。<br>
这里后 3 位 `kid` 就是我们自定义的地址前缀了，当然地址本身的生成方式并没有改变，
只是不断地通过筛选得到拥有该前缀的地址，在你拿到这个地址前，或许已经生成了 n 个地址，这个过程有点像挖矿。
毫无疑问，随着前缀长度的增加，这个筛选过程所耗费的时间会呈指数级增长。

## 安全性
这里的安全指的是黑客可能会偷换博客或论坛等页面上作者留下的比特币捐赠地址，
利用的是人们用肉眼做地址验证时，通常只对比前几位地址，所以黑客往往会生成具有一定长度的和目标地址前缀相同的地址来欺骗捐赠者。
但若是指定前缀的“靓号”地址，黑客若想欺骗捐赠者，必须生成比该前缀长度具有更长前缀的“新靓号”地址，然而这个代价可能是巨大的。<br>
**注：指定的前缀必须符合 [Base58 编码](/2018/05/12/base58-encoding)的结果，即不能包含 0（零），O（大写字母 o），I（大写字母 i）和 l（小写字母 L）。**

## 源码剖析
“源码之前，了无秘密” — 侯捷<br>
要想让比特币启动时就生成指定地址前缀（长度大于 1）的公钥地址，就需要修改源码了。<br>
共需要修改 3 个函数：<br>
1.`GenerateNewKey` 函数，位于“wallet/wallet.cpp”文件中。<br>
该函数的作用是生成一个私钥，并返回对应的公钥。<br>
实现：生成私钥，获取公钥地址，对比地址前缀，不满足则重复以上过程，直至找到指定前缀的地址。

{% highlight C++ %}
CPubKey CWallet::GenerateNewKey()
{
    AssertLockHeld(cs_wallet); // mapKeyMetadata
    bool fCompressed = CanSupportFeature(FEATURE_COMPRPUBKEY); // default to compressed public keys if we want 0.6.0 wallets

+   while (true)
+   {
        CKey secret; // 创建一个私钥
        secret.MakeNewKey(fCompressed); // 随机生成一个数来初始化私钥，注意边界，下界为 1
    
        // Compressed public keys were introduced in version 0.6.0
        if (fCompressed) // 是否压缩公钥，0.6.0 版引入
            SetMinVersion(FEATURE_COMPRPUBKEY);
    
        CPubKey pubkey = secret.GetPubKey(); // 获取与私钥对应的公钥（椭圆曲线加密算法）
        assert(secret.VerifyPubKey(pubkey)); // 验证私钥公钥对是否匹配

+       CKeyID keyID = pubkey.GetID();
+       if (CBitcoinAddress(keyID).ToString().compare(0, 5, "1kids") == 0) { // 只有满足前缀为 "1kids" 的公钥地址才能返回
            // Create new metadata // 创建新元数据/中继数据
            int64_t nCreationTime = GetTime(); // 获取当前时间
            mapKeyMetadata[pubkey.GetID()] = CKeyMetadata(nCreationTime);
            if (!nTimeFirstKey || nCreationTime < nTimeFirstKey)
                nTimeFirstKey = nCreationTime;
    
            if (!AddKeyPubKey(secret, pubkey))
                throw std::runtime_error("CWallet::GenerateNewKey(): AddKey failed");
            return pubkey; // 返回对应的公钥
+       }
+   }
}
{% endhighlight %}

2.RPC 命令 `getnewaddress` 函数，位于“wallet/rpcwallet.cpp”文件中。<br>
该函数的作用是通过比特币核心客户端该调用 RPC 命令，获取一个新的公钥地址。<br>
这里只需要修改第 6 步，实现同上。

{% highlight C++ %}
UniValue getnewaddress(const UniValue& params, bool fHelp) // 在指定账户下新建一个地址，若不指定账户，默认添加到""空账户下
{
    if (!EnsureWalletIsAvailable(fHelp)) // 1.确保钱包可用，即钱包已创建成功
        return NullUniValue;
    
    if (fHelp || params.size() > 1) // 参数个数为 0 或 1，即要么使用默认账户，要么指定账户
        throw runtime_error( // 2.查看该命令的帮助或命令参数个数超过 1 个均返回该命令的帮助
            "getnewaddress ( \"account\" )\n"
            "\nReturns a new Bitcoin address for receiving payments.\n"
            "If 'account' is specified (DEPRECATED), it is added to the address book \n"
            "so payments received with the address will be credited to 'account'.\n"
            "\nArguments:\n"
            "1. \"account\"        (string, optional) DEPRECATED. The account name for the address to be linked to. If not provided, the default account \"\" is used. It can also be set to the empty string \"\" to represent the default account. The account does not need to exist, it will be created if there is no account by the given name.\n"
            "\nResult:\n"
            "\"bitcoinaddress\"    (string) The new bitcoin address\n"
            "\nExamples:\n"
            + HelpExampleCli("getnewaddress", "")
            + HelpExampleRpc("getnewaddress", "")
        );

    LOCK2(cs_main, pwalletMain->cs_wallet); // 3.对钱包加锁

    // Parse the account first so we don't generate a key if there's an error
    string strAccount; // 用于保存帐户名
    if (params.size() > 0) // 有 1 个参数的情况
        strAccount = AccountFromValue(params[0]); // 4.解析第一个参数并将其作为账户名

    if (!pwalletMain->IsLocked()) // 检查钱包是否上锁（被用户加密）
        pwalletMain->TopUpKeyPool(); // 5.填充密钥池

    // Generate a new key that is added to wallet
    CPubKey newKey; // 6.生成一个新密钥并添加到钱包，返回一个对应的比特币地址
+   while (true)
+   {
        if (!pwalletMain->GetKeyFromPool(newKey)) // 获取一个公钥
            throw JSONRPCError(RPC_WALLET_KEYPOOL_RAN_OUT, "Error: Keypool ran out, please call keypoolrefill first");
        CKeyID keyID = newKey.GetID(); // 对 65 bytes 的公钥调用 hash160(即先 sha256, 再 ripemd160)
    
+       if (CBitcoinAddress(keyID).ToString().compare(0, 5, "1kids") == 0) { // 只有满足前缀为 "1kids" 的公钥地址才能返回
            pwalletMain->SetAddressBook(keyID, strAccount, "receive");
    
            return CBitcoinAddress(keyID).ToString(); // 160 位的公钥转化为公钥地址：Base58(1 + 20 + 4 bytes)
+       }
+   }
}
{% endhighlight %}

3.RPC 命令 `validateaddress` 函数，位于“rpcmisc.cpp”文件中。<br>
该函数的作用是通过比特币核心客户端该调用 RPC 命令，验证一个公钥地址是否有效。

{% highlight C++ %}
UniValue validateaddress(const UniValue& params, bool fHelp)
{
    if (fHelp || params.size() != 1) // 参数必须为 1 个
        throw runtime_error( // 命令帮助反馈
            "validateaddress \"bitcoinaddress\"\n"
            "\nReturn information about the given bitcoin address.\n"
            "\nArguments:\n"
            "1. \"bitcoinaddress\"     (string, required) The bitcoin address to validate\n"
            "\nResult:\n"
            "{\n"
            "  \"isvalid\" : true|false,       (boolean) If the address is valid or not. If not, this is the only property returned.\n"
            "  \"address\" : \"bitcoinaddress\", (string) The bitcoin address validated\n"
            "  \"scriptPubKey\" : \"hex\",       (string) The hex encoded scriptPubKey generated by the address\n"
            "  \"ismine\" : true|false,        (boolean) If the address is yours or not\n"
            "  \"iswatchonly\" : true|false,   (boolean) If the address is watchonly\n"
            "  \"isscript\" : true|false,      (boolean) If the key is a script\n"
            "  \"pubkey\" : \"publickeyhex\",    (string) The hex value of the raw public key\n"
            "  \"iscompressed\" : true|false,  (boolean) If the address is compressed\n"
            "  \"account\" : \"account\"         (string) DEPRECATED. The account associated with the address, \"\" is the default account\n"
            "}\n"
            "\nExamples:\n"
            + HelpExampleCli("validateaddress", "\"1PSSGeFHDnKNxiEyFrD1wcEaHr9hrQDDWc\"")
            + HelpExampleRpc("validateaddress", "\"1PSSGeFHDnKNxiEyFrD1wcEaHr9hrQDDWc\"")
        );

#ifdef ENABLE_WALLET
    LOCK2(cs_main, pwalletMain ? &pwalletMain->cs_wallet : NULL); // 钱包上锁
#else
    LOCK(cs_main);
#endif

    CBitcoinAddress address(params[0].get_str()); // 获取指定的比特币地址
    bool isValid = address.IsValid(); // 判断地址是否有效
+   isValid = address.ToString().compare(0, 5, "1kids") == 0; // 若前缀非 "1kids"，则该地址无效

    UniValue ret(UniValue::VOBJ); // 对象类型的返回结果
    ret.push_back(Pair("isvalid", isValid)); // 有效性
    if (isValid) // 若有效
    {
        CTxDestination dest = address.Get(); // 从比特币地址获取交易目的地址
        string currentAddress = address.ToString();
        ret.push_back(Pair("address", currentAddress)); // 当前比特币地址

        CScript scriptPubKey = GetScriptForDestination(dest); // 从交易目的地址获取脚本公钥
        ret.push_back(Pair("scriptPubKey", HexStr(scriptPubKey.begin(), scriptPubKey.end()))); // 脚本公钥

#ifdef ENABLE_WALLET
        isminetype mine = pwalletMain ? IsMine(*pwalletMain, dest) : ISMINE_NO;
        ret.push_back(Pair("ismine", (mine & ISMINE_SPENDABLE) ? true : false)); // 是否属于我的
        ret.push_back(Pair("iswatchonly", (mine & ISMINE_WATCH_ONLY) ? true: false)); // 是否为 watch-only 地址
        UniValue detail = boost::apply_visitor(DescribeAddressVisitor(), dest); // 排序
        ret.pushKVs(detail); // 地址细节
        if (pwalletMain && pwalletMain->mapAddressBook.count(dest)) // 若钱包可用 且 该目的地址在地址簿中
            ret.push_back(Pair("account", pwalletMain->mapAddressBook[dest].name)); // 获取该地址关联的账户名
#endif
    }
    return ret; // 返回结果
}
{% endhighlight %}

关于比特币地址前缀 `'1'` 的修改，参考[如何制作一枚山寨数字货币](/2018/05/13/how-to-make-an-altcoin)。

## 参照
* [Address - Bitcoin Wiki](https://en.bitcoin.it/wiki/Address)
* [List of address prefixes - Bitcoin Wiki](https://en.bitcoin.it/wiki/List_of_address_prefixes)
* [Technical background of version 1 Bitcoin addresses - Bitcoin Wiki](https://en.bitcoin.it/wiki/Technical_background_of_version_1_Bitcoin_addresses)
* [精通比特币（第二版）第四章 密钥和地址 · 巴比特图书](http://book.8btc.com/books/6/masterbitcoin2cn/_book/ch04.html)
* [samr7/vanitygen](https://github.com/samr7/vanitygen) 比特币靓号地址生成器
* [手把手教你拥有个性化的BTC地址 \| 巴比特](http://www.8btc.com/get-a-vanitygen-address)
* [jonathanfoster/vanity-miner: Bitcoin vanity address miner](https://github.com/jonathanfoster/vanity-miner) C++ 版比特币靓号矿工（需要依赖）
* [BitcoinVanityGen.com - Bitcoin Vanity Address Generator Online, Free Bicoin Vanity Address Generation](http://bitcoinvanitygen.com) 在线比特币靓号地址生成器，最多定制 9 位，6 位（含 6 位）以内免费
* [...](https://github.com/mistydew/blockchain)
