---
title: 'How to deploy a custom gas token chain using the Orbit SDK'
sidebar_label: 'Deploy a custom gas token chain'
description: 'How to deploy a custom gas token chain using the Orbit SDK'
author: GreatSoshiant
sme: GreatSoshiant
target_audience: 'Developers deploying and maintaining Orbit chains.'
sidebar_position: 3
---

:::caution UNDER CONSTRUCTION

This document is under construction and may change significantly as we incorporate [style guidance](/for-devs/contribute#document-type-conventions) and feedback from readers. Feel free to request specific clarifications by clicking the `Request an update` button at the top of this document.

:::

:::info

See the ["create a rollup custom fee token" example](https://github.com/OffchainLabs/arbitrum-orbit-sdk/blob/main/examples/create-rollup-custom-fee-token/index.ts) in the Orbit SDK repository for additional guidance.

:::

Deploying a Custom Gas Token Orbit chain introduces a unique aspect to the standard Orbit chain setup: the ability to pay transaction fees using a specific `ERC-20` token instead of `ETH`. While the setup process largely mirrors that of a standard <a data-quicklook-from="arbitrum-rollup-chain">Rollup Orbit chain</a>, as detailed in the [introduction](../orbit-sdk-introduction.md), there are key differences to account for when configuring a Custom Gas Token Orbit chain.

:::important

Custom gas tokens are only supported on Orbit AnyTrust chains, currently. This feature is **not available yet** on Rollup Orbit chains.

:::

## Key Differences for Custom Gas Token Orbit Chain Deployment

### 1. Custom gas token specification

The most significant difference is the specification of the `ERC-20` token on the parent chain to be used as the gas fee token. This requires selecting an existing `ERC-20` token or deploying a new one to be used specifically for transaction fees on your Orbit chain.

**Note:** Currently, only `ERC-20` tokens with 18 decimals are acceptable as gas tokens on Orbit chains.

### 2. Chain Configuration

You can configure your Orbit chain using the `prepareChainConfig` method and assigning it to a `chainConfig` variable.

Example:

```js
import { prepareChainConfig } from '@arbitrum/orbit-sdk';

const chainConfig = prepareChainConfig({
    chainId: Some_Chain_ID,
    nativeToken: yourERC-20TokenAddress,
    DataAvailabilityCommittee: true,
});
```

To use the `prepareChainConfig` method as shown in the example above, some inputs need to be defined:

| Parameter                   | Type      | Description                                                                                                                                                     |
| --------------------------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `chainId`                   | `number`  | Your Orbit chain's `chainId`.                                                                                                                                   |
| `nativeToken`               | `Address` | The contract address on the parent chain of the `ERC-20` token your chain will use for `gas` fees. It needs to have 18 decimals to be accepted on Orbit chains. |
| `DataAvailabilityCommittee` | `boolean` | Should be set to `true` since only AnyTrust chains can accept `ERC-20` tokens.                                                                                  |

### 3. Token Approval Before Deployment Process

In Custom gas token Orbit chains, the owner needs to give allowance to the `rollupCreator` contract before starting the deployment process so that `RollupCreator` can spend enough tokens for the deployment process. For this purpose, we defined two APIs on the Orbit SDK:

#### A. createRollupEnoughCustomFeeTokenAllowance

This API gets related inputs and checks if the `rollupCreator` contract has enough Allowance on the token from the owner:

```js
import {createRollupEnoughCustomFeeTokenAllowance} from '@arbitrum/orbit-sdk';

const allowanceParams = {
nativeToken,
account: deployer.address,
publicClient: parentChainPublicClient,
};

const enough Allowance = createRollupEnoughCustomFeeTokenAllowance(allowanceParams)
```

To build the `allowanceParams` object as shown in the example above, you need to provide with the following:

| Parameter      | Type           | Description                                                                                           |
| -------------- | -------------- | ----------------------------------------------------------------------------------------------------- |
| `nativeToken`  | `Address`      | The contract address on the parent chain of the `ERC-20` token your chain will use for `gas` fees.    |
| `account`      | `Address`      | The address Orbit chain's                                                                             |
| `publicClient` | `PublicClient` | The `PublicClient` object [as defined by the Viem library](https://viem.sh/docs/clients/public.html). |

#### B. createRollupPrepareCustomFeeTokenApprovalTransactionRequest

This API gets related inputs and creates the transaction request to secure enough Allowance from the owner to the `RollupCreator` to spend `nativeToken` on the deployment process.

Example:

```js
import { createRollupEnoughCustomFeeTokenAllowance } from '@arbitrum/orbit-sdk';

const allowanceParams = {
  nativeToken,
  account: deployer.address,
  publicClient: parentChainPublicClient,
};

const approvalTxRequest = await createRollupPrepareCustomFeeTokenApprovalTransactionRequest(
  allowanceParams,
);
```

### 4. Deployment Process

The overall deployment process, including the use of APIs like `createRollupPrepareConfig` and `createRollupPrepareTransactionRequest`, remains similar to the [Rollup deployment](orbit-sdk-deploying-anytrust-chain.md) process. However, attention must be given to incorporating the `ERC-20` token details into these configurations.

:::note

When using the API, you also need to specify `nativeToken` as a param.

:::

Example:

```js
const txRequest = await createRollupPrepareTransactionRequest({
  params: {
    config,
    batchPoster,
    validators: [validator],
    nativeToken,
  },
  account: deployer.address,
  publicClient: parentChainPublicClient,
});
```

All other parts would be the same as explained in the [Rollup Orbit chain deployment page](orbit-sdk-deploying-rollup-chain.md).
