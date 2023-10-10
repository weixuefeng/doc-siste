[!TOC]
# GateChain 普通钱包文档

## 1. 钱包地址生成

### 由 seed 生成扩展私钥

```dart
static Uint8List generateXprvFromSeed(Uint8List seed) {
    int iter = 1;
    Uint8List xprv = Uint8List(96);

    while (true) {
      var key = Hmac(sha512, seed);
      var digest = key
          .convert(Uint8List.fromList(utf8.encode("Root Seed Chain $iter")))
          .bytes;
      Uint8List secretKey =
          Uint8List.fromList(sha512.convert(digest.sublist(0, 32)).bytes);
      Uint8List right = Uint8List.fromList(digest.sublist(32));

      secretKey[0] &= 248;
      secretKey[31] &= 63;
      secretKey[31] |= 64;

      if ((secretKey[31] & 0x20) == 0) {
        xprv.setRange(0, 64, secretKey, 0);
        xprv.setRange(64, 96, right, 0);
        break;
      }

      iter++;
    }

    return xprv;
  }
```

### 由扩展私钥 + 衍生路径生成 derivedXprv, derivedXprv 中包含公私钥内容
```dart
static Uint8List derivePrivateKeyFromPath(Uint8List xprv, String path) {
    final parts = path.split('/');
    for (var i = 0; i < parts.length; i++) {
        final part = parts[i];
        if (i == 0 && part == 'm') {
        continue;
        }

        // Check if the part ends with an apostrophe
        final harden = part.endsWith("'");

        // Remove the apostrophe if it exists
        if (harden) {
        parts[i] = part.substring(0, part.length - 1);
        }

        var index = int.tryParse(parts[i]);
        if (index == null) {
        throw Exception("Invalid BIP 32 path: not a valid integer");
        }

        if (index < 0 || index >= HARD_INDEX) {
        throw Exception("Invalid BIP 32 path: index is negative or too large");
        }

        if (harden) {
        index |= HARD_INDEX;
        }

        xprv = derivePrivate(xprv, index);
    }
    return xprv;
}

static Uint8List derivePrivate(Uint8List xprv, int index) {
    final ekey = xprv.sublist(0, 64);
    final chaincode = xprv.sublist(64, 96);

    final kl = xprv.sublist(0, 32);
    final kr = xprv.sublist(32, 64);

    final seri = Uint8List(4);
    seri.buffer.asByteData().setUint32(0, index, Endian.little);

    final zmac = Hmac(sha512, chaincode);
    final imac = Hmac(sha512, chaincode);
    List<int> zMacArray = [];
    List<int> iMacArray = [];
    if (index >= HARD_INDEX) {
      zMacArray.addAll(Uint8List.fromList([0]) + ekey + seri);
      iMacArray.addAll(Uint8List.fromList([1]) + ekey + seri);
    } else {
      var pubkey = edPubKey(xprv);
      zMacArray.addAll(Uint8List.fromList([2]) + pubkey + seri);
      iMacArray.addAll(Uint8List.fromList([3]) + pubkey + seri);
    }

    final zout = Uint8List.fromList(zmac.convert(zMacArray).bytes);
    final iout = Uint8List.fromList(imac.convert(iMacArray).bytes);
    final zl = zout.sublist(0, 32);
    final zr = zout.sublist(32, 64);

    final result = Uint8List(XPRV_SIZE);
    result.setRange(0, 32, add28Mul8(kl, zl));
    result.setRange(32, 64, add256Bits(kr, zr));
    result.setRange(64, 96, iout.sublist(32));
    return result;
  }
```


### 从 derivedXprv 中获取公私钥信息

```dart
var privateKey = edPrivKey(derived);
var publicKey = edPubKey(derived);

  static Uint8List edPubKey(Uint8List xprv) {
    var hBytes = xprv.sublist(0, 32);
    var A = ed.ExtendedGroupElement();
    ed.GeScalarMultBase(A, hBytes);
    var publicKeyBytes = Uint8List(32);
    A.ToBytes(publicKeyBytes);
    return publicKeyBytes;
  }

  static Uint8List edPrivKey(Uint8List priv) {
    Uint8List hBytes = Uint8List(64);
    hBytes.setRange(0, 32, priv.sublist(0, 32));
    Uint8List pubBytes = edPubKey(priv);
    hBytes.setRange(32, 64, pubBytes.sublist(0, 32));
    return hBytes;
  }
```

### 根据公钥计算 gt 地址

- 地址前缀配置,[账户类型](https://www.gatechain.io/docs/developers/cli/account/)

```dart
static const NORMAL_SINGLE_SIG_ADDRESS_PREFIX = 'gt1';
static const NORMAL_MULTI_SIG_ADDRESS_PREFIX = 'gt2';
static const VAULT_SINGLE_SIG_ADDRESS_PREFIX = 'vault1';
static const VAULT_MULTI_SIG_ADDRESS_PREFIX = 'vault2';
```

- 具体算法

```dart
  static String genSingleSigAddress(Uint8List publicKey, String prefix) {
    prefix = prefix;
    var sha512res = sha512.convert(publicKey);
    var ripemd320res = ripemd320(Uint8List.fromList(sha512res.bytes));
    var words = bech32.toWords(ripemd320res);
    return bech32.encode(Decoded(prefix: prefix, words: words));
  }
```

## 2. [地址余额查询](https://www.gatechain.io/docs/developers/api/account/#%E6%9F%A5%E8%AF%A2%E8%B4%A6%E6%88%B7%E4%BD%99%E9%A2%9D-%E5%91%BD%E4%BB%A4%E8%A1%8C)

## 3. [查询账户信息](https://www.gatechain.io/docs/developers/api/account/#%E6%9F%A5%E8%AF%A2%E8%B4%A6%E6%88%B7%E4%BF%A1%E6%81%AF-%E5%91%BD%E4%BB%A4%E8%A1%8C)

## 4. [交易结构构造](https://www.gatechain.io/docs/developers/api/tx/#%E6%99%AE%E9%80%9A%E4%BA%A4%E6%98%93)

- 测试网 chainId: `meteora`
- valid_height: [节点信息](https://www.gatechain.io/docs/developers/api/#%E6%9F%A5%E8%AF%A2%E8%8A%82%E7%82%B9%E7%8A%B6%E6%80%81%E4%BF%A1%E6%81%AF-%E5%91%BD%E4%BB%A4%E8%A1%8C),数组[当前区块高度，当前区块高度 + 200]， 即 `data['node_status']['lastHeight']`
- demon 单位: `NANOGT`

```dart
Future<dynamic> generateTransaction(
      String fromAddress, String toAddress, BigInt amount) async {
    var path = "$baseUrl/${PYGateApiConfig.generateTx(toAddress)}";
    var height = await getStatus();
    var params = {
      "base_req": {
        "from": fromAddress, //发送者账户
        "memo": "", //交易留言，留言长度限制为中文最多85个字符/英文最多256个字符
        "chain_id": "meteora", //链ID
        "gas": "200000", //交易消耗的gas数量
        "fees": [
          {
            "denom": "NANOGT", //单位
            "amount": "100000" //手续费
          }
        ],
        "simulate": false, //是否模拟计算gas
        "valid_height": [
          //交易生效高度
          height,
          (int.parse(height) + 200).toString()
        ]
      },
      "amount": [
        {
          "denom": "NANOGT", //单位
          "amount": amount.toInt().toString() //转移代币数量
        }
      ]
    };
    var account = await YLCosmosNetworkRequest.post(url: path, params: params);
    return account.data;
  }
```

## 5. 签署交易

- 5.1 根据上一步(交易结构构造)的返回结果，生成签名内容: `txBytes`

```dart
  Uint8List getSignBytes(dynamic tx) {
    var message = {
      "chain_id": "meteora",
      "fee": tx['value']['fee'],
      "memo": tx['value']['memo'],
      "msgs": tx['value']['msg'],
      "nonces": tx['value']['nonces'],
      "valid_height": tx['value']['valid_height'],
    };
    var sortedMessage = sortObject(message);
    return Hash.hashSHA256(jsonEncode(sortedMessage).utf8Encode());
  }

  dynamic sortObject(dynamic obj) {
    if (obj == null) return null;
    if (obj is! Map<String, dynamic> && obj is! List<dynamic>) return obj;

    if (obj is List<dynamic>) {
      return obj.map((item) => sortObject(item)).toList();
    }

    if (obj is Map<String, dynamic>) {
      List<String> sortedKeys = obj.keys.toList()..sort();
      Map<String, dynamic> result = {};
      for (String key in sortedKeys) {
        result[key] = sortObject(obj[key]);
      }
      return result;
    }

    return obj;
  }
```

- 5.2 使用账户私钥，对 `txBytes` 进行签名

使用 `ed25519` 算法变种进行签名，`sign` 函数各个语言类似，demo 为 `dart` 语言

```dart
static Uint8List signTx(String privateKeyHex, Uint8List data) {
    return sign(ed25519.PrivateKey(privateKeyHex.toUint8List()), data);
}
```

```dart
/// Sign signs the message with privateKey and returns a signature. It will
  /// throw ArumentError if privateKey.bytes.length is not PrivateKeySize.
  static Uint8List sign(ed25519.PrivateKey privateKey, Uint8List message) {
    if (privateKey.bytes.length != PrivateKeySize) {
      throw ArgumentError(
          'ed25519: bad privateKey length ${privateKey.bytes.length}');
    }
    var h = sha512.convert(privateKey.bytes.sublist(0, 32));
    var digest1 = h.bytes;
    // 改动 1, 
    // 算法原型:   var expandedSecretKey = digest1.sublist(0, 32);
    var expandedSecretKey = privateKey.bytes.sublist(0, 32);
    // 改动 2, 原先做了如下处理
    // expandedSecretKey[0] &= 248;
    // expandedSecretKey[31] &= 63;
    // expandedSecretKey[31] |= 64;

    var output = AccumulatorSink<Digest>();
    var input = sha512.startChunkedConversion(output);
    input.add(digest1.sublist(32));
    input.add(message);
    input.close();
    var messageDigest = output.events.single.bytes;

    var messageDigestReduced = Uint8List(32);
    ed.ScReduce(messageDigestReduced, messageDigest as Uint8List);
    var R = ed.ExtendedGroupElement();
    ed.GeScalarMultBase(R, messageDigestReduced);

    var encodedR = Uint8List(32);
    R.ToBytes(encodedR);

    output = AccumulatorSink<Digest>();
    input = sha512.startChunkedConversion(output);
    input.add(encodedR);
    input.add(privateKey.bytes.sublist(32));
    input.add(message);
    input.close();
    var hramDigest = output.events.single.bytes;
    var hramDigestReduced = Uint8List(32);
    ed.ScReduce(hramDigestReduced, hramDigest as Uint8List);

    var s = Uint8List(32);
    ed.ScMulAdd(s, hramDigestReduced, expandedSecretKey as Uint8List,
        messageDigestReduced);

    var signature = Uint8List(SignatureSize);
    arrayCopy(encodedR, 0, signature, 0, 32);
    arrayCopy(s, 0, signature, 32, 32);

    return signature;
  }
```

## 6. 广播交易

将 **交易结构构造** 中的结果，加上上一步 **签署交易**的结果，按照 base64 编码，拼接到广播交易结构中。

```dart
  Future<String> signAndSendTransaction(String privateKeyHex,
      String fromAddress, String toAddress, BigInt amount) async {
    var tx = await generateTransaction(fromAddress, toAddress, amount);
    var txBytes = getSignBytes(tx);
    var signed = PYGateMethod.signTx(privateKeyHex, txBytes);
    var public = PYGateMethod.getPublicKey(privateKeyHex);
    var signature = [];
    var sig = {
      "pub_key": {
        "type": "gatechain/PubKeyEd25519",
        "value": base64.encode(public),
      },
      "signature": base64.encode(signed),
    };
    signature.add(sig);
    tx['value']['signatures'] = signature;
    var res = await sendTransaction(tx);
    return res;
  }
```

## 7.示例数据

- 基础钱包测试数据
```sh
mnemonic: orchard wine fine pluck reflect script dish squirrel better book fold glide then rely rule name defense erode push stool oil under dog piano

seed hex: 982006830177ed97cb5f42f6d46afe88cbc743a8ad983f348d56570d95c80a79a8dd2fbb7f3b95d4d3fcf88c996852e0611969469eb0a84cdd7dce7c6bf50fe5

ExtendedPrivateKey: 7831adcf6404c6b1ebd6fbc139739da34128d9ad3b243a0f1123729f90d71643144c7ee02a90cb34500b449eb15a3c8a858d8d9b6618c7db340fd45a0f3907e9972fa480fbcf0c6d6df1dfe8281d34c1d9e1bc5e6dd4e389576a8329aeacce71

derived privateKey: a037bcd8df4c21a24c4c59b5400923c1b640b6f2c427fe17d7e4ec6da3d716438ac6a455568a99fce9682b1fa32bd46047cff8cbfadc1cded6c335f186ca4b5205dc6765e521590ae3558f67b5016b1cd2b0d68cf40ec3459bf8f63973479be0

 accountPrivateKey: a037bcd8df4c21a24c4c59b5400923c1b640b6f2c427fe17d7e4ec6da3d71643a9268a8d046a76392586a6c4c3008c2ed2865081803dd8bbbf3f3d807f9bc02f

 accountPublicKey: a9268a8d046a76392586a6c4c3008c2ed2865081803dd8bbbf3f3d807f9bc02f

 accountAddress: gt11gn5n8ge6e4p75wd6kgkl223986y90nvpdmtztlaxzjr33nj2y7y3wjr4z2ytyzwyvjvag9
```

- 发送交易数据

1. 根据 `status` 获取 `valid_height`

```
{node_status: {lastHeight: 2873342, lastConsensusVersion: v15, nextConsensusVersion: v15, nextConsensusVersionRound: 2873343, nextConsensusVersionSupported: true, timeSinceLastRound: 23, catchupTime: 0, hasSyncedSinceStartup: false, step: 2, period: 0, zeroTimeStamp: 2023-10-10T13:00:14.69881317Z, deadline: 19, fastRecoveryDeadline: 525, minimumTxFee: 100000, minimumBlockTxFee: 10}, application_version: {name: gate, server_name: gated, client_name: gatecli, version: 1.1.4-4-gc0729f58, commit: c0729f58d16fd4ba0e3dfbd62068aec6b9ff10f5/020182bf46f159a631de71078c28a5040bc4e097, build_tags: netgo,ledger, go: go version go1.13.15 linux/amd64}}
```

2. 通过接口获取普通交易数据

- url: `http://139.162.15.16:1317/v1/tx/send/gt11t60zetuu8edfrygh49qmxltdvf0tqw3cmt3szq6eexpq4hn8sml7u7avezmz73zug2kzvt`

- request params: `{base_req: {from: gt11gn5n8ge6e4p75wd6kgkl223986y90nvpdmtztlaxzjr33nj2y7y3wjr4z2ytyzwyvjvag9, memo: , chain_id: meteora, gas: 200000, fees: [{denom: NANOGT, amount: 100000}], simulate: false, valid_height: [2873342, 2873542]}, amount: [{denom: NANOGT, amount: 100}]}`

- response data: `{type: StdTx, value: {msg: [{type: MsgSend, value: {from_address: gt11gn5n8ge6e4p75wd6kgkl223986y90nvpdmtztlaxzjr33nj2y7y3wjr4z2ytyzwyvjvag9, to_address: gt11t60zetuu8edfrygh49qmxltdvf0tqw3cmt3szq6eexpq4hn8sml7u7avezmz73zug2kzvt, amount: [{denom: NANOGT, amount: 100}]}}], fee: {amount: [{denom: NANOGT, amount: 100000}], gas: 200000}, nonces: [null], signatures: null, memo: , valid_height: [2873342, 2873542]}}`

3. 构造签名的交易数据结构

```json
{"type":"StdTx","value":{"msg":[{"type":"MsgSend","value":{"from_address":"gt11gn5n8ge6e4p75wd6kgkl223986y90nvpdmtztlaxzjr33nj2y7y3wjr4z2ytyzwyvjvag9","to_address":"gt11t60zetuu8edfrygh49qmxltdvf0tqw3cmt3szq6eexpq4hn8sml7u7avezmz73zug2kzvt","amount":[{"denom":"NANOGT","amount":"100"}]}}],"fee":{"amount":[{"denom":"NANOGT","amount":"100000"}],"gas":"200000"},"nonces":[null],"signatures":null,"memo":"","valid_height":["2873342","2873542"]}}
```

4. 根据上述内容，计算签名用的 `txBytes`
`txbytes hex: 1ae28d5471f1179f1b140b3184401b7c484e24bd45840cbb6e557cd8c516c477`

5. 使用私钥的签名结果: 
`signature hex: 3c72de2106be459e7140e7d05b48929b0a1d0fffdb2d943c4b4faa57161aa7bafc6b2fd02205e5764e25f55fb744bd6a86cc327e497ac644da99cdaa83227203`

6. 广播交易使用的数据结构:
```json
{"type":"StdTx","value":{"msg":[{"type":"MsgSend","value":{"from_address":"gt11gn5n8ge6e4p75wd6kgkl223986y90nvpdmtztlaxzjr33nj2y7y3wjr4z2ytyzwyvjvag9","to_address":"gt11t60zetuu8edfrygh49qmxltdvf0tqw3cmt3szq6eexpq4hn8sml7u7avezmz73zug2kzvt","amount":[{"denom":"NANOGT","amount":"100"}]}}],"fee":{"amount":[{"denom":"NANOGT","amount":"100000"}],"gas":"200000"},"nonces":[null],"signatures":[{"pub_key":{"type":"gatechain/PubKeyEd25519","value":"qSaKjQRqdjklhqbEwwCMLtKGUIGAPdi7vz89gH+bwC8="},"signature":"PHLeIQa+RZ5xQOfQW0iSmwodD//bLZQ8S0+qVxYap7r8ay/QIgXldk4l9V+3RL1qhswyfkl6xkTamc2qgyJyAw=="}],"memo":"","valid_height":["2873342","2873542"]}}
```

7. 上述交易的`txHash`: `IRREVOCABLEPAY-78E89D70A5344483C6D5493CDB460E6590EDBC720D65117AD3C42CA3B17AB6B205822C33B818A57D3DFF98372007FECE`

