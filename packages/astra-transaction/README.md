# @astra-js/transaction

This package provides a collection of apis to create, sign/send transaction, and receive confirm/receipt.

## Installation

```
npm install @astra-js/transaction
```

## Usage

Create a Astra instance connecting to testnet

```javascript
const { Astra } = require('@astra-js/core');
const {
  ChainID,
  ChainType,
  hexToNumber,
  numberToHex,
  fromWei,
  Units,
  Unit,
} = require('@astra-js/utils');

const astra = new Astra(
    'https://rpc.s0.t.astranetwork.com/',
    {
        chainType: ChainType.Astra,
        chainId: ChainID.AstraTestnet,
    },
);
```

Creating a new transaction using parameters
```javascript
const txn = astra.transactions.newTx({
  to: '0xd6ba69DA5b45eC98b53e3258d7DE756a567B6763',
  value: new Unit(1).asOne().toWei(),
  // gas limit, you can use string
  gasLimit: '21000',
  // send token from shardID
  shardID: 0,
  // send token to toShardID
  toShardID: 0,
  // gas Price, you can use Unit class, and use Gwei, then remember to use toWei(), which will be transformed to BN
  gasPrice: new astra.utils.Unit('1').asGwei().toWei(),
});
```

Recovering transaction from raw transaction hash
```javascript
const raw = '0xf86d21843b9aca00825208808094d6ba69da5b45ec98b53e3258d7de756a567b6763880de0b6b3a76400008028a0da8887719f377401963407fc1d82d2ab52404600cf7bea37c27bd2dfd7c86aaaa03c405b0843394442b303256a804bde835821a8a77bd88a2ced9ffdc8b0a409e9';
const tx = astra.transactions.recover(raw);
```

Getting the RLP encoding of a transaction (rawTransaction), along with raw transaction field values that were encoded
```javascript
const [encoded, raw] = txn.getRLPUnsigned()
```

Sign the transaction using a wallet and send the transaction, wait for confirmation and print receipt
```javascript
// key corresponds to 0xFF9a025FdC0fd4b68Ba4e9Bff46476de15ED8f4D, only has testnet balance
astra.wallet.addByPrivateKey('45e497bd45a9049bcb649016594489ac67b9f052a6cdf5cb74ee2427a60bf25e');

astra.wallet.signTransaction(txn).then(signedTxn => {
  signedTxn.sendTransaction().then(([tx, hash]) => {
    console.log('tx hash: ' + hash);
    signedTxn.confirm(hash).then(response => {
      console.log(response.receipt);
    });
  });
});
```

Asynchronous transaction sign, send, and confirm
```javascript
async function transfer() {
  astra.wallet.addByPrivateKey('45e497bd45a9049bcb649016594489ac67b9f052a6cdf5cb74ee2427a60bf25e');

  const signedTxn = await astra.wallet.signTransaction(txn);
  signedTxn
    .observed()
    .on('transactionHash', (txnHash) => {
      console.log('');
      console.log('--- hash ---');
      console.log('');
      console.log(txnHash);
      console.log('');
    })
    .on('receipt', (receipt) => {
      console.log('');
      console.log('--- receipt ---');
      console.log('');
      console.log(receipt);
      console.log('');
    })
    .on('cxReceipt', (receipt) => {
      console.log('');
      console.log('--- cxReceipt ---');
      console.log('');
      console.log(receipt);
      console.log('');
    })
    .on('error', (error) => {
      console.log('');
      console.log('--- error ---');
      console.log('');
      console.log(error);
      console.log('');
    });

  const [sentTxn, txnHash] = await signedTxn.sendTransaction();

  const confiremdTxn = await sentTxn.confirm(txnHash);

  // if the transactino is cross-shard transaction
  if (!confiremdTxn.isCrossShard()) {
    if (confiremdTxn.isConfirmed()) {
      console.log('--- Result ---');
      console.log('');
      console.log('Normal transaction');
      console.log(`${txnHash} is confirmed`);
      console.log('');
      console.log('please see detail in explorer:');
      console.log('');
      console.log('https://explorer.testnet.astra.one/#/tx/' + txnHash);
      console.log('');
      process.exit();
    }
  }
  if (confiremdTxn.isConfirmed() && confiremdTxn.isCxConfirmed()) {
    console.log('--- Result ---');
    console.log('');
    console.log('Cross-Shard transaction');
    console.log(`${txnHash} is confirmed`);
    console.log('');
    console.log('please see detail in explorer:');
    console.log('');
    console.log('https://explorer.testnet.astra.one/#/tx/' + txnHash);
    console.log('');
    process.exit();
  }
}
transfer();
```



