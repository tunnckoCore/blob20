# blobscriptions

> A command-line interface, a library, and a set of tools to manage and manipulate blobscriptions.

Ethscriptions ESIP-8 implementation in JavaScript/TypeScript, also called Blobscriptions which is
using Ethereum Blob Transactions to create Ethscriptions. The package includes a CLI to create and
manage blobscriptions, as well as a library to track/index blobscriptions and BLOB-20 tokens. It is
built with plugins architecture for easier development.

For the BLOB-20 specification, see the [blob20](./blob20.md) file.

## Install and use

To use as a library, you can install it via npm:

```
npm i blobscriptions
```

And then import it in your project:

```typescript
import { blobscriptions, type TxPayload } from 'blobscriptions';

const options = { logging: true, blob20: true };

blobscriptions((payload: TxPayload) => {
  console.log('payload:', payload);
  // do whatever you want with the payload, like storing it in a database, etc.

  // it includes the block, the transaction, the attachment, and the blob20 (if it is a blob20 token transaction)
  console.log('block:', payload.block.number, payload.block.hash);
  console.log('tx:', payload.transaction.hash, payload.transaction.from, payload.transaction.to);
  console.log('attachment:', payload.attachment.contentType, payload.attachment.sha);

  // if it is a blob20 token transaction and blob20 is enabled (it is by default)
  console.log('blob20:', payload.blob20?.protocol, payload.blob20?.token.operation);
}, options).catch((e) => {
  console.error('Failure in blobscriptions handling...', e);
});
```

## Payload

The payload is an object with the following structure:

```typescript
import type { Block, Chain, TransactionBase, TransactionEIP4844, Transport } from 'viem';

type TxPayload = {
  transaction: TransactionBase & Omit<TransactionEIP4844, 'accessList'>;
  block: Pick<
    Block,
    | 'timestamp'
    | 'miner'
    | 'blobGasUsed'
    | 'gasUsed'
    | 'gasLimit'
    | 'baseFeePerGas'
    | 'hash'
    | 'number'
  >;
  attachment: {
    contentType: Uint8Array | string;
    content: Uint8Array | string;
    sha: `0x${string}`;
  };
  blob20?: {
    protocol: 'blob20';
    token: {
      operation: 'deploy' | 'mint' | 'transfer';
      // operation-specific fields
      // ...
    };
  };
};

type TrackBlobsOptions = {
  chain?: Chain; // mainnet
  transport?: Transport; // http to 1rpc.io/eth
  onlyBlobscriptions?: boolean; // true
  blob20?: boolean; // true
  trimInput?: boolean; // true
  logging?: boolean; // false
};
```

## Webhooks

In case you want to receive the payload as a webhook, you can use pass a URL string instead of a
function, or you can provide both a function and a URL string to send the payload to the URL as well
as to the function.

```typescript
blobscriptions('https://your-webhook-url.com', (payload: TxPayload) => {
  console.log('payload:', payload);
});
```

The webhook data that this POST request handler will receive is a JSON object with the following
structure:

```typescript
type WebhookPayload = {
  type: 'BLOBSCRIPTIONS';
  timestamp: number; // timestamp, javascript Date.now()
  payload: TxPayload;
};
```

## CLI

to be documented and implemented
