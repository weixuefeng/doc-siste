# VARA 接入

- [官网](https://vara-network.io/)
- [wiki](https://wiki.vara-network.io/)
- [ss58 repo](https://github.com/paritytech/ss58-registry/blob/main/ss58-registry.json#L814)
- [slip44,913,VARA](https://github.com/satoshilabs/slips/blob/master/slip-0044.md)
- [ss58Prefix,137](https://github.com/gear-tech/gear/blob/a721177f3a1761dcba90b98a703619521c50e78d/runtime/common/src/constants.rs#L22)
- [decimal,12](https://github.com/gear-tech/gear/blob/a721177f3a1761dcba90b98a703619521c50e78d/runtime/common/src/constants.rs#L25)
- [explorer](https://explorer.vara-network.io/)
- [subscan](https://vara.subscan.io/)


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

