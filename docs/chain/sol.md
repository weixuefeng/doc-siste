# sol 相关文档

## SOL NonceAccount 使用

创建 Nonce 账户

```javascript
let account = Keypair.fromSecretKey(Uint8Array.from(a));
let nonceAccount = Keypair.fromSecretKey(Uint8Array.from(b));
//   // Get Minimum amount for rent exemption
let minimumAmount = await connection.getMinimumBalanceForRentExemption(
    NONCE_ACCOUNT_LENGTH,
);

// Form CreateNonceAccount transaction
let transaction = new Transaction().add(
SystemProgram.createNonceAccount({
    fromPubkey: account.publicKey,
    noncePubkey: nonceAccount.publicKey,
    authorizedPubkey: account.publicKey,
    lamports: minimumAmount,
}),
);
// Create Nonce Account
var sig = await sendAndConfirmTransactio(
    connection, 
    transaction, 
    [
        account,
        nonceAccount,
    ]
);
```

使用 nonce 账户发送交易

```javascript
let account = Keypair.fromSecretKey(Uint8Array.from(a));
let nonceAccount = Keypair.fromSecretKey(Uint8Array.from(b));

const ix = SystemProgram.transfer({
    fromPubkey: account.publicKey,
    toPubkey: account.publicKey,
    lamports: toLamports(0.01),
});
const advanceIX = SystemProgram.nonceAdvance({
    authorizedPubkey: account.publicKey,
    noncePubkey: nonceAccount.publicKey,
});

const tx = new Transaction();

tx.add(advanceIX);
tx.add(ix);

const accountInfo = await connection.getAccountInfo(nonceAccount.publicKey);
const nonceAccountInfo = NonceAccount.fromAccountData(accountInfo.data);

tx.recentBlockhash =  nonceAccountInfo.nonce;
tx.feePayer = account.publicKey;
tx.sign(account)
var sig = await sendAndConfirmTransaction(connection, tx, [
    nonceAccount,
    account,
]);
console.log("sig:",sig)
```

## 发送流程检查

- 发送 SOL:
    - 最低账户租金 `rent` = `0.00089088`
    - 发送金额(`amount`)检查：
        1. 获取钱包余额 `balance`
        2. 如果 `balance` < `amount`, 提示 *余额不足，补充 SOL* , 以下情况 `balance` > `amount`.
        3. 判断账户租金是否满足, `balance` < `amount` + `rent`, 提示 *请预留至少 0.00089088 SOL 以支付租金*
        3. 检查目标地址是否处于激活状态，如果是激活状态:
            1. 计算发送需要的手续费 `fee`, 如果 `balance` < `amount` + `fee` + `rent`, 提示用户 *扣除网络费用后，最高可用金额为 xxx SOL*
            2. 如果 `balance` > `amount` + `fee` + `rent`, 那么进入下一步
        4. 如果目标地址不是激活状态，则需要激活目标地址，激活金额 0.00089088：
            1. 判断, 如果`amount` < `rent`, 提示: *请发送至少 0.00089088 SOL 以支付租金*
            2. 如果 `amount` > `rent`, 根据 3.1 进行计算
    - 发送最大值计算
        1. 发送到最大值 = 当前余额 - 租金费用（0.00089088）- 手续费

- 发送 spl-token:
    - 发送金额 `amount` 检查:
        1. 获取 token 余额 `token-balance`, sol 余额 `balance`,
        2. 判断: 如果`amount` < `token-balance`, 提示 *余额不足，补充 xx token*
        3. 计算手续费 `fee`
        4. 判断账户是否激活，如果已经激活，判断如果 `balance` < `fee` + `rent`, 提示 *请预留至少 0.00089088 SOL 以支付租金* , 否则就正常下一步发送
        5. 如果没有激活账户，判断如果 `balance` - `rent` < `fee` + `rent`, 提示 *请预留至少 0.00089088 SOL 以支付租金* , 否则就正常下一步发送
        6. 直接发送


- sol 最低账户租期费用： https://solana.com/docs/more/exchange#minimum-deposit-withdrawal-amounts

