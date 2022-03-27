# Logging

Last updated: September 12, 2021

## How to configure bitcoind debug logging

### On node startup

Logging can be configured on bitcoind startup with the `-debug=` option.

For example, `bitcoind -debug=lock -debug=i2p` on the command line will turn on
logging of lock contentions and I2P information in the debug log.

The same options can be used in the `bitcoin.conf` file without the leading
hyphen and with one command per line:

    debug=lock
    debug=i2p

### On a running node

Logging can be read and set on a running instance of bitcoind using the
logging RPC:

    bitcoin-cli logging
    bitcoin-cli help logging

#### Examples

To turn on debug=net:

    bitcoin-cli logging '["net"]'

To turn off debug=net:

    bitcoin-cli logging [] '["net"]'

To turn on debug=all:

    bitcoin-cli logging '["all"]'

To turn off all debug logging:

    bitcoin-cli logging [] '["all"]'


## Log categories

Here are the log categories and a description of each. This is an updated
version of this [Bitcoin Stack Exchange
answer](https://bitcoin.stackexchange.com/questions/66892/what-are-the-debug-categories/66895#66895)
in 2017 by Andrew Chow.

          addrman: Address Manager. Messages about the status of the address
          manager and when addresses are added or removed from the address
          manager database.

          bench: Messages about the benchmark performance of various parts of
          the software that can have performance issues.

          cmpctblock: Messages about the Compact Blocks relay protocol,
          including when blocks are partially downloaded or reconstructed.

          coindb: Coin Database. Messages about the coin database which contains
          the UTXO set, including messages about database flushes and writes.

          estimatefee: Messages about the fee estimation algorithm, including
          messages about when fee estimates are requested and information about
          the status of the fee estimator.

          http: Messages related to the HTTP server that is used to handle the
          RPC requests. These messages will typically be for the startup and
          shutdown of the server as well as received requests.

          i2p: All messages related to using the I2P privacy network.

          ipc: All requests and responses between processes. See
          [doc/multiprocess.md](https://github.com/bitcoin/bitcoin/blob/master/doc/multiprocess.md).

          libevent: Messages from the libevent library used for the HTTP server.

          lock: All lock contentions and their duration.

          mempool: Messages related to actions done in the memory pool, most
          frequently transaction acceptance (`AcceptToMemoryPool`). Also
          includes transaction removals.

          mempoolrej: Messages about transactions rejected from the memory pool.

          net: All messages related to communicating with other nodes on the
          network, including what P2P messages were sent and received and to
          whom and other information about the network messages.

          proxy: Messages about using a SOCKS5 proxy and its authentication.

          prune: Messages about local blockchain pruning, including the result
          of a pruning operation.

          qt: Messages about Qt, the GUI framework.

          rand: Messages for when randomness is needed by any function.

          reindex: Messages about the reindexing process, in particular errors
          about out-of-order blocks and repeated blocks.

          rpc: Messages about the RPC server, including its startup and shutdown
          as well as when commands are issued.

          selectcoins: Coin Selection. Messages about the UTXOs that are
          selected when sending money.

          tor: All messages related to using a TOR SOCKS5 proxy and TOR hidden
          service (used for receiving incoming connections over TOR). This
          includes messages about the creation and shutdown of the TOR hidden
          service and messages about the connection to the TOR proxy.

          validation: All messages related to the validation interface, most
          frequently `TransactionAddedToMempool` and `stored orphan tx`.

          walletdb (db in v.0.19 and earlier): Messages about the status of the
          Berkeley Database engine used for the wallet database, including
          messages about database flushes.

          zmq: Messages about the ZeroMQ notification system, including the
          startup and shutdown of the service as well as when notifications are
          issued and new clients connected.

This is not an exhaustive list of the types of messages you will see for each
category. Also, some categories have many possible messages whilst others have
very few.
