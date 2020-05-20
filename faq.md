# FAQ

## What is EtherCore?

EtherCore is a decentralized open source smart contract platform with its own cryptocurrency \(ERE\) that is based on ProgPoW algorithm and Ethereum protocol. The main goal of EtherCore is to research and develop a reference implementation of a secure, scalable, and decentralized blockchain ecosystem and application development environment.

## Is EtherCore \(ERE\) a token or a coin?

EtherCore is a mineable cryptocurrency with ERE ticker. EtherCore is distributed and developed as a self-governed main network.

## How to claim the airdrop

EtherCore coin \(ERE\) was initially distributed as an airdrop to the holders of Ethereum. A snapshot was taken at block X and distributed in 1:1 ratio to the new blockchain of EtherCore Network.

In order to claim EtherCore, the funds have to be on the supported wallets or exchanges.

## Which wallets support EtherCore?

EtherCore mainnet is supported by several wallets:

* [EtherCore Wallet](https://wallet.ethercore.io): The native web-based wallet of EtherCore
* [MyEtherWallet](https://myetherwallet.com): A free, client-side open-source web-based wallet for Ethereum-like blockchains
* [MyCrypto](https://mycrypto.com): Web, Windows, OSX and Linux app wallet
* [Trezor](https://trezor.io): Hardware based cryptocurrency wallet for storing coins
* [Ledger](https://ledger.com): State-of-the-art Hardware Wallet for crypto assets

## How do I run EtherCore node?

You can run EtherCore node with the following supported node implementations:

* [Go-EtherCore](https://github.com/ethercore/go-ethercore/releases/latest): Golang based native EtherCore mainnet node implementation
* [Parity-Ethereum](https://github.com/paritytech/parity-ethereum/releases/latest): Rust based node implementation for Ethereum-like networks

For go-ethercore, start the node with `geth` command, for parity-ethereum, start the node with `parity --chain=ethercore` command

## Can I use ERC-20 and ERC-721 contracts on EtherCore mainnet?

Yes. EtherCore supports Solidity as a smart contract language. [ERC-20](https://eips.ethereum.org/EIPS/eip-20) and [ERC-721](https://eips.ethereum.org/EIPS/eip-721) token written in Solidity for Ethereum can be deployed and executed on EtherCore.

## Can I use Truffle for the smart contract development on EtherCore?

Yes. Truffle can be used in developing smart contracts on EtherCore with the node running on local or using our public remote node endpoint [https://rpc.ethercore.io](https://rpc.ethercore.io).

## What is a port of EtherCore node?

EtherCore uses 30303 \(both tcp and udp\) for p2p communication between nodes and 8545 / 8546 for RPC / Web socket connection.

## Is EtherCore mineable?

Yes, EtherCore coins can be mined via Solo Mining with local EtherCore node or a Mining Pool supporting EtherCore mainnet.

## What algorithm does EtherCore use?

EtherCore uses ProgPoW, an ASIC resistant, GPU friendly programmatic Proof-of-Work mining consensus algorithm for mainnet and Clique / Istanbul BFT Proof-of-Authority consensus algorithm for sidechain implementation.

## How do I mine EtherCore?

You can mine EtherCore with a GPU that has at least 2GB of memory using the following miner implementations:

* [EthCoreMiner](https://github.com/ethercore/ethcoreminer/releases/latest): GPU desktop miner for EtherCore ProgPoW mining, supports the latest ProgPoW 0.9.3 implementation.
* [TT-Miner](https://bitcointalk.org/index.php?topic=5025783.0): Windows and linux miner for multiple GPU mineable coins.

## How many EtherCore are rewarded per block?

At this moment on phrase 1 of regular halving schedule, EtherCore has 1 ERE of block reward.

## How will the block reward evaluate?

EtherCore has implemented a regular halving schedule which is called [ECIP-1017](https://github.com/ethereumproject/ECIPs/blob/master/ECIPs/ECIP-1017.md), every 100 million blocks the mining reward of EtherCore will be decreased up to 30 percent.

## Is EtherCore ASIC-resistant?

With the help of ProgPoW consensus algorithm EtherCore is considered as an ASIC resistant GPU mineable coin.

## What is the block time?

EtherCore mainnet block time is ~ 14 seconds and for sidechain implementation 1 second.

## I have a mining pool. How do I get featured on your website?

Please contact us via our official email address [bd@ethercore.io](mailto:bd@ethercore.io) or our discord / telegram community to get your pool featured on our website.

