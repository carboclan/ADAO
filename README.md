# ADAO

A new SRML-based Substrate node, ready for hacking.

With developing.....

# 

## 团队名称：Apollo

## 项目名称：ADAO

ADAO是基于Substrate Runtime实现的一个社区驱动、DAO治理的广告系统。采用哈博格税模型实现广告牌的价值发现和定价。赞助商或广告主需要购买广告牌的使用权，并向DAO支付哈博格税，通过市场博弈来实现广告牌的合理定价。而DAO组织可以使用税收收入，通过智能合约提案治理的方式对广告系统的条款（如广告内容筛选等）进行管理。

已实现了一个在线广告牌，更多功能开发中。。。

目前可以将页面上的多个展示位卖出，功能包括：

- 创建广告位（banner）
- 设定拍卖状态（24*600个块后结束，最终状态：有效成交和流拍需要一笔交易触发）
- 出价竞拍
- 竞拍成功后，可以上传展示图片url到广告位

公开函数

- create_banner （创建广告位）
- set_image_url （广告位所有者修改广告位图片）
- auction_banner（广告位所有者创建拍卖）
- bid（竞拍）

数据结构

```rust
* Banner
  * id
  * name
  * image_url
  * desc
  * current_price
  * current_bidder
  * can_bid
  * bid_end_height
```

模块Storage：

```rust
Banners get(banner): map T::Hash => Banner<T::Hash, T::Balance, T::AccountId, T::BlockNumber>;
BannerOwner get(owner_of): map T::Hash => Option<T::AccountId>;

AllBannersArray get(banner_by_index): map u64 => T::Hash;
AllBannersCount get(all_banners_count): u64;
AllBannersIndex: map T::Hash => u64;

OwnedBannersArray get(banner_of_owner_by_index): map (T::AccountId, u64) => T::Hash;
OwnedBannersCount get(owned_banner_count): map T::AccountId => u64;
OwnedBannersIndex: map T::Hash => u64;
```

用map模拟全部banners数组，某地址所有banners数组的实现。

模块Event：

```rust
* CreateBanner(AccountId, Hash)
* StartAuction(AccountId, Hash, Balance)
* Bid(AccountId, Hash, Balance)
* Transferred(AccountId, AccountId, Hash)
* Deal(AccountId, Hash, Balance)
* Abort(AccountId, Hash)
```

根据交易类型及上述event类型可以重现某个banner的所有拍卖流程

# Building

Install Rust:

```bash
curl https://sh.rustup.rs -sSf | sh
```

Install required tools:

```bash
./scripts/init.sh
```

Build the WebAssembly binary:

```bash
./scripts/build.sh
```

Build all native code:

```bash
cargo build
```

# Run

You can start a development chain with:

```bash
cargo run -- --dev
```

Detailed logs may be shown by running the node with the following environment variables set: `RUST_LOG=debug RUST_BACKTRACE=1 cargo run -- --dev`.

If you want to see the multi-node consensus algorithm in action locally, then you can create a local testnet with two validator nodes for Alice and Bob, who are the initial authorities of the genesis chain that have been endowed with testnet units. Give each node a name and expose them so they are listed on the Polkadot [telemetry site](https://telemetry.polkadot.io/#/Local%20Testnet). You'll need two terminal windows open.

We'll start Alice's substrate node first on default TCP port 30333 with her chain database stored locally at `/tmp/alice`. The bootnode ID of her node is `QmQZ8TjTqeDj3ciwr93EJ95hxfDsb9pEYDizUAbWpigtQN`, which is generated from the `--node-key` value that we specify below:

```bash
cargo run -- \
  --base-path /tmp/alice \
  --chain=local \
  --alice \
  --node-key 0000000000000000000000000000000000000000000000000000000000000001 \
  --telemetry-url ws://telemetry.polkadot.io:1024 \
  --validator
```

In the second terminal, we'll start Bob's substrate node on a different TCP port of 30334, and with his chain database stored locally at `/tmp/bob`. We'll specify a value for the `--bootnodes` option that will connect his node to Alice's bootnode ID on TCP port 30333:

```bash
cargo run -- \
  --base-path /tmp/bob \
  --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/QmQZ8TjTqeDj3ciwr93EJ95hxfDsb9pEYDizUAbWpigtQN \
  --chain=local \
  --bob \
  --port 30334 \
  --telemetry-url ws://telemetry.polkadot.io:1024 \
  --validator
```

Additional CLI usage options are available and may be shown by running `cargo run -- --help`.
