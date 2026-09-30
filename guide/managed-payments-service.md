# Managed Payments Service

dStorage Pro is a hosted signing/payment service run by the dStorage team. It covers the network
fees your app incurs across every supported network — Arweave storage costs and Midnight DUST
chain fees alike — billed to your dStorage Pro account instead of drawn from each end user's own
wallet, so your users never need a funded wallet on any of them. Access is authorized per-request
via a token you configure once. It also offers a `TEST` sandbox, which runs the same payment flow
against your account without spending real funds, so you can build and test your integration
before going live.

This guide takes the [Local & Simulator Adapters](/guide/local-simulator-adapters) script and
routes its Arweave storage payment through dStorage Pro's `TEST` (sandbox) service. You get the
full managed-payment round trip against your real dStorage Pro account, while uploads still go to
your local arlocal instance and no real funds are spent.

## Prerequisites

Everything from the [Local & Simulator Adapters](/guide/local-simulator-adapters) guide, plus a
dStorage Pro token:

- Node.js 22 or later
- [arlocal](https://github.com/textury/arlocal) running locally — see
  [Step 1 of the Local & Simulator Adapters guide](/guide/local-simulator-adapters#step-1-—-start-arlocal)
- A dStorage Pro account and API token from [portal.dstorage.pro](https://portal.dstorage.pro)
Fast track: clone [`starter-template`](https://github.com/dStorageTech/dstorage-docs/tree/main/starter-template) and wire up this guide's adapters in minutes.

## Step 1 — Get a dStorage Pro API token

Go to [portal.dstorage.pro](https://portal.dstorage.pro), sign in, and create a token from the
dashboard. Copy it — you'll paste it into the adapter config in Step 2.

::: tip Free coupons
Free coupons for the dStorage Pro service and its sandbox are regularly shared on our socials —
[X](https://x.com/dStorageTech), [Reddit](https://www.reddit.com/r/dStorage), and
[LinkedIn](https://www.linkedin.com/company/dstorage-tech) — like
[this one](https://x.com/dStorageTech/status/2105119589810217410?s=20). Redeem them from the
portal to top up your balance.
:::

See the [Managed Payments FAQ](/faq/managed-payments#managed-payments-dstorage-pro) for the
different token types and how to scope them.

## Step 2 — Add managed payment to the storage adapter

Starting from the exact Step 2 config of the Local & Simulator Adapters guide, the only change is
a `managedPayment` option passed to `ArweaveLocalStorageAdapter.createWithTestWallet()`:

```typescript
import {
  DStorage,
  ArweaveLocalStorageAdapter,
  MidnightSimulatorChainAdapter,
  PasswordEncryptionAdapter,
} from "@dstorage-tech/dstorage-sdk";

const signingServerUrl = "https://portal.dstorage.pro";
const authToken = "your_dstorage_pro_token_here"; // from portal.dstorage.pro

const { adapter: storageAdapter } = await ArweaveLocalStorageAdapter.createWithTestWallet({
  fundAr: 5, // amount of test AR to fund the generated wallet with
  managedPayment: {
    signingServerUrl,
    authToken,
    testMode: true,
  },
});

const sdk = new DStorage({
  storageAdapters: [storageAdapter],
  chainAdapters: [new MidnightSimulatorChainAdapter()],
  encryptionAdapters: [
    new PasswordEncryptionAdapter({
      password: "Correct-Horse-Battery!",
      salt: "myapp:v1",
    }),
  ],
});
```

With `testMode: true`, each upload's payment request is sent to dStorage Pro with the `TEST`
network identifier: dStorage Pro records it as a sandbox payment against your account, without
spending any real funds.

Your content never goes to dStorage Pro — not the raw data, and not even the encrypted bytes.
Only the transaction metadata and its Merkle root (`data_root`) are sent for payment; the
encrypted content is uploaded directly to arlocal afterwards.

## Step 3 — Init, store, retrieve

Same call pattern as every other guide in this series:

```typescript
await sdk.init();

const { chainRefId } = await sdk.store(
  new TextEncoder().encode("hello, dStorage"),
);

const { bytes } = await sdk.retrieveByRefId(chainRefId);
console.log(new TextDecoder().decode(bytes)); // "hello, dStorage"
```

What's different from the Local & Simulator Adapters guide:

- **The storage payment goes through dStorage Pro** — each upload is quoted and recorded as a
  `TEST` payment on your dStorage Pro account, so you can see it in the portal dashboard.
- **Everything else is unchanged** — uploads still land in your local arlocal instance, and the
  on-chain reference is still written by `MidnightSimulatorChainAdapter`.

## Learn More

- Browse the [FAQ](/faq/managed-payments#managed-payments-dstorage-pro) for the full
  managed-payments reference, including token scoping and security notes.
- Next: [Midnight Network Adapter](/guide/midnight-network-adapter) swaps the simulator for a real
  Midnight network, a real wallet, and a live proof server.
