# @astra-js/core

This package provides a collection of apis to interact with Astra blockchain.

## Installation

```
npm install @astra-js/core
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

Getting balance of account `0xFF9a025FdC0fd4b68Ba4e9Bff46476de15ED8f4D`
```javascript
astra.blockchain
  .getBalance({ address: '0xFF9a025FdC0fd4b68Ba4e9Bff46476de15ED8f4D' })
  .then((response) => {
    console.log('balance in ONEs: ' + fromWei(hexToNumber(response.result), Units.one));
  });
```

Getting the latest block number
```javascript
astra.blockchain.getBlockNumber().then((response) => {
  console.log('current block number: ' + hexToNumber(response.result));
});
```

Getting the block using block hash
```javascript
astra.blockchain
  .getBlockByHash({
    blockHash: '0x08c46ae7249362a7d1f602d44c5a81f33ebdab6a7dcb6068f99610b57911aafd',
  })
  .then((response) => {
    console.log(response.result);
  });
```

Getting the block using block number
```javascript
astra.blockchain
  .getBlockByNumber({
    blockNumber: numberToHex(422635),
  })
  .then((response) => {
    console.log(response.result);
  });
```

Getting the transaction using hash
```javascript
astra.blockchain
  .getTransactionByHash({
    txnHash: '0x56c73eb993b18dc04baacec5c2e9d1292a090f6a978a4a1c461db5255fcbc831',
  })
  .then((response) => {
    console.log(response.result);
  });
```

Getting the transaction receipt
```javascript
astra.blockchain
  .getTransactionReceipt({
    txnHash: '0x56c73eb993b18dc04baacec5c2e9d1292a090f6a978a4a1c461db5255fcbc831',
  })
  .then((response) => {
    console.log(response.result);
  });
```

Getting the cross-shard transaction receipt
```javascript
astra.blockchain
  .getCxReceiptByHash({
    txnHash: '0xcd36a90ff5d5373285c2896ba7bbcd3f5324263c0cb8ecfb7cad2f5fc2fbdbda',
    shardID: 1,
  })
  .then((value) => {
    console.log(value.result);
  });
```

Getting the deployed smart contract code
```javascript
astra.blockchain
  .getCode({
    address: '0x08AE1abFE01aEA60a47663bCe0794eCCD5763c19',
    blockNumber: 'latest',
  })
  .then((response) => {
    console.log(response.result);
  });
```

Getting the transaction count of an account
```javascript
astra.blockchain
  .getTransactionCount({
    address: '0x0B585F8DaEfBC68a311FbD4cB20d9174aD174016',
  })
  .then((response) => {
    console.log(hexToNumber(response.result));
  });
```

Getting the shard structure and details
```javascript
astra.blockchain.getShardingStructure().then((response) => {
  console.log(response.result);
});
```

Transferring funds using `sendTransaction`
```javascript
// key corresponds to 0xFF9a025FdC0fd4b68Ba4e9Bff46476de15ED8f4D, only has testnet balance
astra.wallet.addByPrivateKey('45e497bd45a9049bcb649016594489ac67b9f052a6cdf5cb74ee2427a60bf25e');

async function transfer() {
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

  // sign the transaction use wallet;
  const signedTxn = await astra.wallet.signTransaction(txn);
  const txnHash = await astra.blockchain.sendTransaction(signedTxn);
  console.log(txnHash.result);
}

transfer();
```