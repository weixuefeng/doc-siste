# VARA 接入

- [官网](https://vara-network.io/)
- [wiki](https://wiki.vara-network.io/)
- [ss58 repo](https://github.com/paritytech/ss58-registry/blob/main/ss58-registry.json#L814)
- [slip44,913,VARA](https://github.com/satoshilabs/slips/blob/master/slip-0044.md)
- [ss58Prefix,137](https://github.com/gear-tech/gear/blob/a721177f3a1761dcba90b98a703619521c50e78d/runtime/common/src/constants.rs#L22)
- [decimal,12](https://github.com/gear-tech/gear/blob/a721177f3a1761dcba90b98a703619521c50e78d/runtime/common/src/constants.rs#L25)
- [explorer](https://explorer.vara-network.io/)
- [subscan](https://vara.subscan.io/)
- [idea 测试工具](https://idea.gear-tech.io/)


## wallet-core 接入 polkadot 系列链
- 更新链信息到 `registry.json`
- 生成新链信息 `./codegen/newcoin vara`
- 删除无用文件 `比如 varaNetwork 目录`，因为所有逻辑都走 `polkadot`。
- 编写 `c++`, `kotlin`, `swift` 测试用例

- 构建 `walletconsole`

```
cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=Debug
make -Cbuild
```

- 测试

```
./build/tests/tests tests --gtest_filter=*
./tools/build-and-test
```

- 生成proto文件
```
./tools/generate-files
```

- 构建iOS
```
./tools/ios-build && ./tools-xcframework
```

- 构建android aar
```
./build
```

## 具体方案
- 采用 [pokladart](https://github.com/leonardocustodio/polkadart/) 构造交易，交易结构 wallet-core 支持的不好，算法采用 `ed25519`
- 采用 wallet-core 进行签名，方便快捷
- 采用 chainCode+key 进行衍生
- derivattionPath: `m/44'/913'/0'/0'/0'`
- 衍生路径: `m/${position}'/0'/0'`
- 助记词到私钥采用 bip39, bip44，没有使用 [`substrateBip39`](https://github.com/paritytech/substrate-bip39), 因此地址和插件钱包不一致。还有一个原因，我认为当前的助记词也可以满足 substratebip39 的愿景，只是如何体现到产品上。

## 注意问题参考链接
- [账户存在金额](https://support.polkadot.network/support/solutions/articles/65000180045-error-balances-existentialdeposit-)

## 当前问题

Polkadot 地址和交易支持多种算法 `ed25519`, `sr25519`,`secp256k1`, 官方建议钱包 `subwallet`和官方工具 `subKey`  支持的算法为`sr25519`
但是 [`wallet-core`](https://github.com/trustwallet/wallet-core/issues/3512) 和 flutter 的 lib  [`polkadart`](https://github.com/leonardocustodio/polkadart/blob/86309e1ffedfacd30283681a1cf7d313cc0cdecc/packages/polkadart/lib/extrinsic/extrinsic_payload.dart#L67C1-L67C1) 目前仅支持 `ed25519` 算法，导致地址和上述两个工具不一致
