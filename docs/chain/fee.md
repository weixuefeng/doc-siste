# 钱包中手续费计算

## sei(cosmos) 系列
通过构造 Fake 签名交易模拟正式签名交易，发送到链上进行预估，部分代码展示:

```dart
  static String simulate = "cosmos/tx/v1beta1/simulate";

  // https://docs.cosmos.network/swagger/#/Service/Simulate
  // https://github.com/AufWallet/alan3.dart/tree/main
  Future<PYSeiGasModel> simulate(String txBytes) async {
    var res = await YLCosmosNetworkRequest.post(
        url: "$mRestUrl/${CosmosApiConfig.simulate}",
        params: {"tx_bytes": txBytes});
    var model = PYSeiGasModel.fromMap(res.data['gas_info']);
    return model;
  }

  Future<String> calculateFee(
      String fromAddress, String toAddress, String amount,
      {String denom = "usei",
      String? memo,
      int coinType = TWCoinType.TWCoinTypeSei}) async {
    var fee = "120000";
    var chainId = await getChianId();
    var txAmount = Cosmos.Amount(denom: denom, amount: amount);
    var sendCoinMsg = Cosmos.Message_Send(
      amounts: [txAmount],
      fromAddress: fromAddress,
      toAddress: toAddress,
    );
    var message = Cosmos.Message(sendCoinsMessage: sendCoinMsg);

    var feeAmount = Cosmos.Amount(amount: fee, denom: denom);

    var seiFee = Cosmos.Fee(amounts: [feeAmount], gas: $fixnum.Int64(200000));

    var accountInfo = await queryAccountInfo(fromAddress);
    var key = HDWallet().getKeyForCoin(coinType).data();
    var signingInput = Cosmos.SigningInput(
        accountNumber: $fixnum.Int64(accountInfo.accountNumber),
        signingMode: Cosmos.SigningMode.Protobuf,
        fee: seiFee,
        chainId: chainId,
        sequence: $fixnum.Int64(accountInfo.sequence),
        privateKey: key,
        messages: [message]);
    var signed = AnySigner.sign(signingInput.writeToBuffer(), coinType);
    var output = Cosmos.SigningOutput.fromBuffer(signed);
    var map = jsonDecode(output.serialized);
    var txBytes = map['tx_bytes'].toString();
    var res = await simulate(txBytes);
    return res.gas_used;
  }
```

## vara, polkadot 系列
通过设置 fake 签名，构造交易，由链上进行手续费计算

```dart
Future<BigInt> calculateTransferFee(
      String fromAddress, String toAddress, BigInt amount,
      {bool transferAll = false, bool keepAlive = true}) async {
    var blockNumber = await getBlockNumber();
    var methodCall = "";
    if (transferAll) {
      methodCall = transferAllCall(toAddress, keepAlive);
    } else {
      methodCall = transferCall(toAddress, amount);
    }
    var nonce = await getNonce(fromAddress);
    var extrinsic = Extrinsic(
        signer: Uint8List.fromList(Address.decode(fromAddress).pubkey)
            .toHex()
            .split0x(),
        method: methodCall,
        signature:
            "d19e03fc1a4ec115ec52d29e53676ddaeae0467134f9513b29ed3cd6fd6cd551a96c35b92b867dfd08ba37417e5733620acc4ad17c1d7c65909d6edaaffd4d0e", // fake signature
        eraPeriod: 64,
        blockNumber: blockNumber,
        nonce: nonce,
        tip: 0);
    // signing payload encode
    var raw = extrinsic.encode(getRegistry());
    var feeInfo = await provider.send("payment_queryFeeDetails", [raw.toHex()]);
    if (feeInfo.result != null) {
      var info = feeInfo.result['inclusionFee'];
      var baseFee =
          BigInt.parse(info['baseFee'].toString().split0x(), radix: 16);
      var lenFee = BigInt.parse(info['lenFee'].toString().split0x(), radix: 16);
      var weightFee = BigInt.parse(
          info['adjustedWeightFee'].toString().split0x(),
          radix: 16);
      var result = baseFee + weightFee + lenFee;
      return result;
    } else {
      Log.e(feeInfo.error.toString());
      return BigInt.zero;
    }
  }
```

## trx 波场系列

- 合约手续费估算，类似 evm

```dart

  static String gasPrice = "wallet/getenergyprices";
/// get energy price, unit is sun.
  Future<int> getEnergyPrice() async {
    var url = "$restUrl/${TronApiConfig.gasPrice}";
    var res = await YLCosmosNetworkRequest.get(url: url);
    var prices = res.data['prices'];
    if (prices != null) {
      var priceArray = prices.toString().split(",");
      var currentPrice =
          priceArray[priceArray.length - 1].toString().split(":")[1];
      return int.parse(currentPrice);
    } else {
      throw "Get Energy Price Error";
    }
  }

  /// calculate Contract Fee
  /// [contractAddress] contract address
  /// [ownerAddress] wallet from address
  /// [function] function selector info (https://developers.tron.network/docs/parameter-encoding-and-decoding#function-selector)
  /// [param] function parameter, info(https://developers.tron.network/docs/parameter-encoding-and-decoding#argument-encoding)
  Future<int> calculateContractFee(String contractAddress, String ownerAddress,
      String function, String param) async {
    var price = await getEnergyPrice();
    var gas =
        await estimateGas(contractAddress, ownerAddress, function, param) * 1.1;
    var res = gas * price;
    return res.toInt();
  }


static String estimate = "wallet/triggerconstantcontract";
  /// estimate gas
  Future<int> estimateGas(String contractAddress, String ownerAddress,
      String function, String param) async {
    var url = "$restUrl/${TronApiConfig.estimate}";
    var res = await YLCosmosNetworkRequest.post(url: url, params: {
      "owner_address": ownerAddress,
      "contract_address": contractAddress,
      "function_selector": function,
      "parameter": param,
      "fee_limit": 1000000000,
      "visible": true,
    });
    var energyUsed = int.parse(res.data['energy_used'].toString());
    return energyUsed;
  }
```

普通交易手续费预估:

```dart
  Future<int> calculateNormalFee(
      String ownerAddress, String toAddress, int amount) async {
    var transaction = await createTransaction(ownerAddress, toAddress, amount); // 走 rest 接口
    var rawTransaction = transaction['raw_data_hex'].toString();
    var rawLength = rawTransaction.toUint8List().length;
    var res = await signTransaction(    // 构造交易体，当前使用 wallet-core，可以考虑使用其他 sdk 进行处理
        ownerAddress,  
        toAddress,
        bytesToHex(HDWallet().getKeyForCoin(TWCoinType.TWCoinTypeTron).data()), // fake private key
        amount,
        TWCoinType.TWCoinTypeTron);
    var sig = jsonDecode(res)['signature'][0].toString().toUint8List().length;
    // https://github.com/tronprotocol/wallet-cli/issues/292
    var bytes = rawLength + sig + 64 + 5;
    return bytes;
  }

    /// sign noraml transaction. send trx.
  /// [ownerAddress] wallet address
  /// [toAddress] to address
  /// [privateKeyHex] wallet private key hex string
  /// [amount] unit is sun
  /// [coinType] coinType value about [TWCoinType]
  Future<String> signTransaction(String ownerAddress, String toAddress,
      String privateKeyHex, int amount, int coinType) async {
    var blockInfo = await getBlock();
    var block = blockInfo.block_header!.raw_data!;
    Tron.SigningInput input = Tron.SigningInput(
        privateKey: privateKeyHex.toUint8List().toList(),
        transaction: Tron.Transaction(
            blockHeader: Tron.BlockHeader(
                txTrieRoot: block.txTrieRoot.toString().toUint8List(),
                parentHash: block.parentHash.toString().toUint8List(),
                number: $fixnum.Int64(block.number),
                witnessAddress: block.witness_address.toString().toUint8List(),
                timestamp: $fixnum.Int64(block.timestamp),
                version: block.version),
            transfer: Tron.TransferContract(
                ownerAddress: ownerAddress,
                toAddress: toAddress,
                amount: $fixnum.Int64(amount))));
    var signed = AnySigner.sign(input.writeToBuffer(), coinType);
    Tron.SigningOutput output = Tron.SigningOutput.fromBuffer(signed);
    return output.json;
  }
```

## sui 系列
sui 通过构造转账交易，由区块链进行计算手续费预估

```dart
  Future<BigInt> calculateFee(
      String targetAddress, String signerAddress, String amount) async {
    var suiCoinInfo = await getCoins(signerAddress);
    var objectId = calculateObject(amount, suiCoinInfo);  // 这里手动选择 object 进行处理，可以参考 js sdk splitCoin 进行处理会更准确一点，当前 demo 仅供参考
    var balance = await getBalance(signerAddress); 
    // https://docs.sui.io/testnet/learn/tokenomics/gas-in-sui#gas-budget-examples
    var gas = complexNetGas;
    var txBytes = await unSafePaySui( // 最终调用 dryRunTransactionBlock 进行处理
        targetAddress, signerAddress, amount, objectId!, gas.toString());
    return dryRunTransactionBlock(txBytes);
  }

  Future<BigInt> dryRunTransactionBlock(String txBytes) {
    return _makeRPCCall<Map<String, dynamic>>('sui_dryRunTransactionBlock', [
      txBytes,
    ]).then((value) => value["effects"]["gasUsed"]).then((value) =>
        calculateFee(
            int.parse(value["computationCost"]),
            int.parse(value["storageCost"]),
            int.parse(value["storageRebate"])));
  }

 Future<String> unSafePaySui(String targetAddress, String signerAddress,
      String amount, List<String> objectId, String gasBudget) {
    return _makeRPCCall<Map<String, dynamic>>('unsafe_paySui', [
      signerAddress,
      objectId,
      [targetAddress],
      [amount],
      gasBudget
    ]).then((value) => value['txBytes']);
  }
```

## sol 系列
sol 系列的手续费需要用到私钥，链上会验证签名信息，可以多看看文档深入研究一下，有bug随时联系 `weixuefeng1018@gmail.com`.

```dart
  /// calculate transaction fee
  /// [targetAddress] receiver address
  /// [privateKeyHex] singer private key hex string
  /// [amount] send amount, uint is lamport, 1 lamport = 10^-9 SOL.
  /// [memo] memo string.
  Future<String> calculateFee(String targetAddress, String privateKeyHex,
      String amount, String? memo) async {
    Solana.SigningOutput output =
        await signTransaction(targetAddress, privateKeyHex, amount, memo);
    var tx = output.unsignedTx;
    var decode = Base58.base58DecodeNoCheck(tx);
    tx = base64.encode(decode!.toList());
    return getFeeForMessage(tx);
  }


  Future<String> getFeeForMessage(String message) {
    return _makeRPCCall<Map<String, dynamic>>('getFeeForMessage', [message])
        .then((value) => jsonEncode(value['value']));
  }
```

## btc 系列
btc 手续费和input，output 相关，和交易地址类型有关，可[参考链接](https://github.com/jlopp/bitcoin-transaction-size-calculator/blob/master/index.html)
[trustwalletcore](https://github.com/trustwallet/wallet-core/blob/c94d86028abfb8ca2c22746983b493f0fa78f024/src/Bitcoin/TransactionBuilder.cpp#L35)


## aptos
构造交易体，和 fake 签名，由区块链进行手续费计算
```dart
/// publicKey: signer public key, for simulate transation.
  /// fromAddress: signer address
  /// toAddress: receiver address
  /// amount: transfer native token amount
  /// transferType: [PYAptTransferType]
  /// preGasPrice: pre set gas price. default = 0;
  /// accountAddress: for nft transfer, coin transfer, it is token creator.
  /// module: only for transfer coin, coin module
  /// name: only for transfer coin, coin name
  /// nftSenderAddress: only for claim nft, nftSenderAddress
  /// collectionName: only for transfer nft, collection name
  /// tokenName: only for transfer nft, token name
  /// propertyVersion: only for transfer nft, propertyVersion, if set or null.
  /// nftAmounbt: only for transfer nft, default = 1
  Future<PYAptoGasAmount> simulateTransaction(
      String publicKey,
      String fromAddress,
      String? toAddress,
      BigInt amount,
      PYAptTransferType transferType,
      int coinType,
      {String? accountAddress,
      String? module,
      String? name,
      String? nftSenderAddress,
      String? collectionName,
      String? tokenName,
      int? propertyVersion,
      int nftAmount = 1}) async {
    var wallet = HDWallet();
    var expiration =
        DateTime.now().add(Duration(minutes: 30)).millisecondsSinceEpoch;
    var sequenceNumber = await querySequenceNumber(fromAddress);
    // query chain id
    var chainId = await getChainId();
    // gasPrice
    var gasPrice = await getGasPrice();
    // calcualte transfer type, hide for 1.7.0
    Aptos.TransferMessage? transfer = null;
    Aptos.TokenTransferCoinsMessage? coinTransfer = null;
    Aptos.OfferNftMessage? offerNftMessage = null;
    Aptos.ClaimNftMessage? claimNftMessage = null;

    if (transferType == PYAptTransferType.normalTransfer) {
      transfer = Aptos.TransferMessage(
          to: toAddress, amount: $fixnum.Int64(amount.toInt()));
    } else if (transferType == PYAptTransferType.coinTransfer) {
      coinTransfer = Aptos.TokenTransferCoinsMessage(
          amount: $fixnum.Int64(amount.toInt()),
          to: toAddress,
          function: Aptos.StructTag(
              accountAddress: accountAddress, module: module, name: name));
    } else if (transferType == PYAptTransferType.offerNFT) {
      offerNftMessage = Aptos.OfferNftMessage(
          receiver: toAddress,
          creator: accountAddress,
          collectionName: collectionName,
          name: tokenName,
          propertyVersion:
              propertyVersion == null ? null : $fixnum.Int64(propertyVersion),
          amount: $fixnum.Int64(nftAmount));
    } else if (transferType == PYAptTransferType.claimNFT) {
      claimNftMessage = Aptos.ClaimNftMessage(
          sender: nftSenderAddress,
          creator: accountAddress,
          collectionName: collectionName,
          name: tokenName,
          propertyVersion:
              propertyVersion == null ? null : $fixnum.Int64(propertyVersion));
    }

    // sign transaction
    Aptos.SigningInput input = Aptos.SigningInput(
        sender: fromAddress,
        transfer: transfer,
        nftMessage: (offerNftMessage == null && claimNftMessage == null)
            ? null
            : Aptos.NftMessage(
                offerNft: offerNftMessage, claimNft: claimNftMessage),
        tokenTransferCoins: coinTransfer,
        privateKey: wallet.getKeyForCoin(coinType).data(),
        gasUnitPrice: $fixnum.Int64(gasPrice.gas_estimate),
        expirationTimestampSecs: $fixnum.Int64(expiration),
        sequenceNumber: $fixnum.Int64(int.parse(sequenceNumber)),
        chainId: chainId);
    Aptos.SigningOutput output = Aptos.SigningOutput.fromBuffer(
        AnySigner.sign(input.writeToBuffer(), coinType));
    var path = "$_mRestUrl/${AptosApiConfig.simulateTransaction}";
    var paramsMap = jsonDecode(output.json);
    paramsMap['signature']['public_key'] = publicKey;
    paramsMap['signature']['signature'] =
        "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000";
    var res = await YLCosmosNetworkRequest.post(url: path, params: paramsMap);
    var gas = PYAptoGasAmount(
        max_gas_amount: int.parse(res.data[0]['max_gas_amount'].toString()),
        gas_used: int.parse(res.data[0]['gas_used'].toString()));
    return gas;
  }
```