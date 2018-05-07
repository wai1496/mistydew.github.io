---
layout: post
title:  "启动比特币核心 bitcoind"
date:   2018-05-04 03:08:22 +0800
categories: jekyll update
---
原始可用的比特币程序有两个版本；
一个带有图形化用户界面（通常被称为“比特币”），和一个“无头”版本（被称为 bitcoind，这里的“无头”指的是没有图形化界面，只有命令行）。
它们完全互相兼容，并使用相同的命令行参数，读取相同的配置文件，且读写相同的数据文件。
你可以在你的系统上运行比特币或 bitcoind 的一个副本一次（如果你不小心启动来另一个，该副本将让你知道：比特币或 bitcoind 已经启动并该程序将要退出）。

![bitcoin](/images/20180504/bitcoin.svg)

## 读在前面
比特币相关的解读目前均采用 [bitcoin v0.12.1](https://github.com/bitcoin/bitcoin/tree/v0.12.1)，此版本为官方内置挖矿算法的最后一版。<br>
目前比特币的最新版本为 bitcoin v0.16.0，离区块链 1.0 落地还有些距离。

## Linux 快速启动
使用命令行客户端启动（从头开始）的最简单方式，自动同步区块并创建一个钱包，只要从包含你的 bitcoind 二进制程序的目录运行以下命令（不带参数）：

> `./bitcoind`

运行标准的图形化界面：

> `./bitcoin-qt`

## 命令行参数
以下命令是准确的来自于比特币核心版本 v0.12.1.

> `./bitcoind --help -help-debug` 得到的。
{% highlight C++ %}
Usage:
  bitcoind [options]                     Start Bitcoin Core Daemon

Options:

  -?
       This help message

  -version
       Print version and exit

  -alerts
       Receive and display P2P network alerts (default: 0)

  -alertnotify=<cmd>
       Execute command when a relevant alert is received or we see a really
       long fork (%s in cmd is replaced by message)

  -blocknotify=<cmd>
       Execute command when the best block changes (%s in cmd is replaced by
       block hash)

  -blocksonly
       Whether to operate in a blocks only mode (default: 0)

  -checkblocks=<n>
       How many blocks to check at startup (default: 288, 0 = all)

  -checklevel=<n>
       How thorough the block verification of -checkblocks is (0-4, default: 3)

  -conf=<file>
       Specify configuration file (default: bitcoin.conf)

  -daemon
       Run in the background as a daemon and accept commands

  -datadir=<dir>
       Specify data directory

  -dbcache=<n>
       Set database cache size in megabytes (4 to 16384, default: 100)

  -loadblock=<file>
       Imports blocks from external blk000??.dat file on startup

  -maxorphantx=<n>
       Keep at most <n> unconnectable transactions in memory (default: 100)

  -maxmempool=<n>
       Keep the transaction memory pool below <n> megabytes (default: 300)

  -mempoolexpiry=<n>
       Do not keep transactions in the mempool longer than <n> hours (default:
       72)

  -par=<n>
       Set the number of script verification threads (-1 to 16, 0 = auto, <0 =
       leave that many cores free, default: 0)

  -pid=<file>
       Specify pid file (default: bitcoind.pid)

  -prune=<n>
       Reduce storage requirements by pruning (deleting) old blocks. This mode
       is incompatible with -txindex and -rescan. Warning: Reverting this
       setting requires re-downloading the entire blockchain. (default: 0 =
       disable pruning blocks, >550 = target size in MiB to use for block
       files)

  -reindex
       Rebuild block chain index from current blk000??.dat files on startup

  -sysperms
       Create new files with system default permissions, instead of umask 077
       (only effective with disabled wallet functionality)

  -txindex
       Maintain a full transaction index, used by the getrawtransaction rpc
       call (default: 0)

Connection options:

  -addnode=<ip>
       Add a node to connect to and attempt to keep the connection open

  -banscore=<n>
       Threshold for disconnecting misbehaving peers (default: 100)

  -bantime=<n>
       Number of seconds to keep misbehaving peers from reconnecting (default:
       86400)

  -bind=<addr>
       Bind to given address and always listen on it. Use [host]:port notation
       for IPv6

  -connect=<ip>
       Connect only to the specified node(s)

  -discover
       Discover own IP addresses (default: 1 when listening and no -externalip
       or -proxy)

  -dns
       Allow DNS lookups for -addnode, -seednode and -connect (default: 1)

  -dnsseed
       Query for peer addresses via DNS lookup, if low on addresses (default: 1
       unless -connect)

  -externalip=<ip>
       Specify your own public address

  -forcednsseed
       Always query for peer addresses via DNS lookup (default: 0)

  -listen
       Accept connections from outside (default: 1 if no -proxy or -connect)

  -listenonion
       Automatically create Tor hidden service (default: 1)

  -maxconnections=<n>
       Maintain at most <n> connections to peers (default: 125)

  -maxreceivebuffer=<n>
       Maximum per-connection receive buffer, <n>*1000 bytes (default: 5000)

  -maxsendbuffer=<n>
       Maximum per-connection send buffer, <n>*1000 bytes (default: 1000)

  -onion=<ip:port>
       Use separate SOCKS5 proxy to reach peers via Tor hidden services
       (default: -proxy)

  -onlynet=<net>
       Only connect to nodes in network <net> (ipv4, ipv6 or onion)

  -permitbaremultisig
       Relay non-P2SH multisig (default: 1)

  -peerbloomfilters
       Support filtering of blocks and transaction with bloom filters (default:
       1)

  -enforcenodebloom
       Enforce minimum protocol version to limit use of bloom filters (default:
       0)

  -port=<port>
       Listen for connections on <port> (default: 8222 or testnet: 18222)

  -proxy=<ip:port>
       Connect through SOCKS5 proxy

  -proxyrandomize
       Randomize credentials for every proxy connection. This enables Tor
       stream isolation (default: 1)

  -seednode=<ip>
       Connect to a node to retrieve peer addresses, and disconnect

  -timeout=<n>
       Specify connection timeout in milliseconds (minimum: 1, default: 5000)

  -torcontrol=<ip>:<port>
       Tor control port to use if onion listening enabled (default:
       127.0.0.1:9051)

  -torpassword=<pass>
       Tor control port password (default: empty)

  -whitebind=<addr>
       Bind to given address and whitelist peers connecting to it. Use
       [host]:port notation for IPv6

  -whitelist=<netmask>
       Whitelist peers connecting from the given netmask or IP address. Can be
       specified multiple times. Whitelisted peers cannot be DoS banned and
       their transactions are always relayed, even if they are already in the
       mempool, useful e.g. for a gateway

  -whitelistrelay
       Accept relayed transactions received from whitelisted peers even when
       not relaying transactions (default: 1)

  -whitelistforcerelay
       Force relay of transactions from whitelisted peers even they violate
       local relay policy (default: 1)

  -maxuploadtarget=<n>
       Tries to keep outbound traffic under the given target (in MiB per 24h),
       0 = no limit (default: 0)

Wallet options:

  -disablewallet
       Do not load the wallet and disable wallet RPC calls

  -keypool=<n>
       Set key pool size to <n> (default: 100)

  -fallbackfee=<amt>
       A fee rate (in BTC/kB) that will be used when fee estimation has
       insufficient data (default: 0.0002)

  -mintxfee=<amt>
       Fees (in BTC/kB) smaller than this are considered zero fee for
       transaction creation (default: 0.00001)

  -paytxfee=<amt>
       Fee (in BTC/kB) to add to transactions you send (default: 0.00)

  -rescan
       Rescan the block chain for missing wallet transactions on startup

  -salvagewallet
       Attempt to recover private keys from a corrupt wallet.dat on startup

  -sendfreetransactions
       Send transactions as zero-fee transactions if possible (default: 0)

  -spendzeroconfchange
       Spend unconfirmed change when sending transactions (default: 1)

  -txconfirmtarget=<n>
       If paytxfee is not set, include enough fee so transactions begin
       confirmation on average within n blocks (default: 2)

  -maxtxfee=<amt>
       Maximum total fees (in BTC) to use in a single wallet transaction;
       setting this too low may abort large transactions (default: 0.10)

  -upgradewallet
       Upgrade wallet to latest format on startup

  -wallet=<file>
       Specify wallet file (within data directory) (default: wallet.dat)

  -walletbroadcast
       Make the wallet broadcast transactions (default: 1)

  -walletnotify=<cmd>
       Execute command when a wallet transaction changes (%s in cmd is replaced
       by TxID)

  -zapwallettxes=<mode>
       Delete all wallet transactions and only recover those parts of the
       blockchain through -rescan on startup (1 = keep tx meta data e.g.
       account owner and payment request information, 2 = drop tx meta data)

Debugging/Testing options:

  -uacomment=<cmt>
       Append comment to the user agent string

  -checkblockindex
       Do a full consistency check for mapBlockIndex, setBlockIndexCandidates,
       chainActive and mapBlocksUnlinked occasionally. Also sets -checkmempool
       (default: 0)

  -checkmempool=<n>
       Run checks every <n> transactions (default: 0)

  -checkpoints
       Disable expensive verification for known chain history (default: 1)

  -dblogsize=<n>
       Flush wallet database activity from memory to disk log every <n>
       megabytes (default: 100)

  -disablesafemode
       Disable safemode, override a real safe mode event (default: 0)

  -testsafemode
       Force safe mode (default: 0)

  -dropmessagestest=<n>
       Randomly drop 1 of every <n> network messages

  -fuzzmessagestest=<n>
       Randomly fuzz 1 of every <n> network messages

  -flushwallet
       Run a thread to flush wallet periodically (default: 1)

  -stopafterblockimport
       Stop running after importing blocks from disk (default: 0)

  -limitancestorcount=<n>
       Do not accept transactions if number of in-mempool ancestors is <n> or
       more (default: 25)

  -limitancestorsize=<n>
       Do not accept transactions whose size with all in-mempool ancestors
       exceeds <n> kilobytes (default: 101)

  -limitdescendantcount=<n>
       Do not accept transactions if any ancestor would have <n> or more
       in-mempool descendants (default: 25)

  -limitdescendantsize=<n>
       Do not accept transactions if any ancestor would have more than <n>
       kilobytes of in-mempool descendants (default: 101).

  -debug=<category>
       Output debugging information (default: 0, supplying <category> is
       optional). If <category> is not supplied or if <category> = 1, output
       all debugging information.<category> can be: addrman, alert, bench,
       coindb, db, lock, rand, rpc, selectcoins, mempool, mempoolrej, net,
       proxy, prune, http, libevent, tor, zmq.

  -nodebug
       Turn off debugging messages, same as -debug=0

  -gen
       Generate coins (default: 0)

  -genproclimit=<n>
       Set the number of threads for coin generation if enabled (-1 = all
       cores, default: 1)

  -help-debug
       Show all debugging options (usage: --help -help-debug)

  -logips
       Include IP addresses in debug output (default: 0)

  -logtimestamps
       Prepend debug output with timestamp (default: 1)

  -logtimemicros
       Add microsecond precision to debug timestamps (default: 0)

  -mocktime=<n>
       Replace actual time with <n> seconds since epoch (default: 0)

  -limitfreerelay=<n>
       Continuously rate-limit free transactions to <n>*1000 bytes per minute
       (default: 15)

  -relaypriority
       Require high priority for relaying free or low-fee transactions
       (default: 1)

  -maxsigcachesize=<n>
       Limit size of signature cache to <n> MiB (default: 40)

  -minrelaytxfee=<amt>
       Fees (in BTC/kB) smaller than this are considered zero fee for relaying,
       mining and transaction creation (default: 0.00001)

  -printtoconsole
       Send trace/debug info to console instead of debug.log file

  -printpriority
       Log transaction priority and fee per kB when mining blocks (default: 0)

  -privdb
       Sets the DB_PRIVATE flag in the wallet db environment (default: 1)

  -shrinkdebugfile
       Shrink debug.log file on client startup (default: 1 when no -debug)

Chain selection options:

  -testnet
       Use the test chain

  -regtest
       Enter regression test mode, which uses a special chain in which blocks
       can be solved instantly. This is intended for regression testing tools
       and app development.

Node relay options:

  -acceptnonstdtxn
       Relay and mine "non-standard" transactions (testnet/regtest only;
       default: 1)

  -bytespersigop
       Minimum bytes per sigop in transactions we relay and mine (default: 20)

  -datacarrier
       Relay and mine data carrier transactions (default: 1)

  -datacarriersize
       Maximum size of data in data carrier transactions we relay and mine
       (default: 83)

  -mempoolreplacement
       Enable transaction replacement in the memory pool (default: 1)

Block creation options:

  -blockminsize=<n>
       Set minimum block size in bytes (default: 0)

  -blockmaxsize=<n>
       Set maximum block size in bytes (default: 750000)

  -blockprioritysize=<n>
       Set maximum size of high-priority/low-fee transactions in bytes
       (default: 0)

  -blockversion=<n>
       Override block version to test forking scenarios

RPC server options:

  -server
       Accept command line and JSON-RPC commands

  -rest
       Accept public REST requests (default: 0)

  -rpcbind=<addr>
       Bind to given address to listen for JSON-RPC connections. Use
       [host]:port notation for IPv6. This option can be specified multiple
       times (default: bind to all interfaces)

  -rpccookiefile=<loc>
       Location of the auth cookie (default: data dir)

  -rpcuser=<user>
       Username for JSON-RPC connections

  -rpcpassword=<pw>
       Password for JSON-RPC connections

  -rpcauth=<userpw>
       Username and hashed password for JSON-RPC connections. The field
       <userpw> comes in the format: <USERNAME>:<SALT>$<HASH>. A canonical
       python script is included in share/rpcuser. This option can be specified
       multiple times

  -rpcport=<port>
       Listen for JSON-RPC connections on <port> (default: 8221 or testnet:
       18221)

  -rpcallowip=<ip>
       Allow JSON-RPC connections from specified source. Valid for <ip> are a
       single IP (e.g. 1.2.3.4), a network/netmask (e.g. 1.2.3.4/255.255.255.0)
       or a network/CIDR (e.g. 1.2.3.4/24). This option can be specified
       multiple times

  -rpcthreads=<n>
       Set the number of threads to service RPC calls (default: 4)

  -rpcworkqueue=<n>
       Set the depth of the work queue to service RPC calls (default: 16)

  -rpcservertimeout=<n>
       Timeout during HTTP requests (default: 30)
{% endhighlight %}

## 比特币配置文件样例
来自 [https://github.com/mistydew/blockchain/blob/master/bitcoin.conf](https://github.com/mistydew/blockchain/blob/master/bitcoin.conf):

## 参照
* [Running Bitcoin - Bitcoin Wiki](https://en.bitcoin.it/wiki/Running_Bitcoin)
* [bitcoin.conf](https://github.com/mistydew/blockchain/blob/master/bitcoin.conf)
* [精通比特币（第二版） \| 巴比特图书](http://book.8btc.com/masterbitcoin2cn)
* [...](https://github.com/mistydew)