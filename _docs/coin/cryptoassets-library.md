---
title: Adding *MyCoin* to CryptoAssets library
subtitle:
tags: [define explorer, explorers, currency model, unit]
category: Blockchain Support
author:
toc: true
layout: doc
---

The [@ledgerhq/cryptoassets](https://github.com/LedgerHQ/ledgerjs/tree/master/packages/cryptoassets) package contains all definitions of cryptoassets that could be useful to Ledger Live.

Ledger Live deals with different kinds of currencies and assets: fiat currencies, cryptocurrencies, and tokens.
They all share similar concepts but have specifics, which are defined as a Currency model.

All types can be found in [@ledgerhq/cryptoassets/src/types.ts](https://github.com/LedgerHQ/ledgerjs/tree/master/packages/cryptoassets/src/types.ts)

## The Currency model

Essentially all currencies share this common ground:

- `name: string`: the display name of a currency. For instance `"Bitcoin"` or `"Euro"`.
- `ticker: string`: the ticker name in exchanges and used for rates (for instance BTC or EUR).
- `countervalueTicker?: string`: an alias for the ticker (or alternative ticker)
- `units: Unit[]`: all available `Unit` of the currency (see below what is a Unit). by convention, [0] is the default and has the "highest" magnitude.
- `disableCountervalue?: boolean`: if available, this field expresses that we should assume there is not valid counter value for this coin. (because either we can't trade it or it's not available enough to assume enabling its value)
- `symbol?: string`: if available, it's a short symbol for the coin. <i>In practice it's currently not used and might be dropped.</i>


### Unit

```js
type Unit = {
  // display name of a given unit (example: satoshi)
  name: string,
  // string to use when formatting the unit. like 'BTC' or 'USD'
  code: string,
  // number of digits after the '.' in context of this unit.
  magnitude: number,
  // should it always print all digits even if they are 0 (usually: true for fiats, false for cryptos)
  showAllDigits?: boolean
};
```

A unit describes a given representation of a currency for humans. A currency can have many units, for instance, we can assume Euro have euros and cents. We can define Bitcoin to have: bitcoin, mBTC, bit, satoshi (but that's up to us really).

There are however two essential properties we must respect:

- the first unit (`[0]`) in an array of units is the **highest unit**, typically the default and commonly used unit (euro, bitcoin)
- the last unit must have a **magnitude of 0**: it is the smallest unit which in fine determines the magnitude of the coin (the most atomic unit representable in this currency) and drives integers representation.

Here is an example of Bitcoin units:

```js
[
  {
    name: "bitcoin",
    code: "BTC",
    magnitude: 8
  },
  {
    name: "mBTC",
    code: "mBTC",
    magnitude: 5
  },
  {
    name: "bit",
    code: "bit",
    magnitude: 2
  },
  {
    name: "satoshi",
    code: "sat",
    magnitude: 0
  }
];
```

The field `showAllDigits` is generally unset for all cryptocurrencies and set for fiat currencies to true (but there are exceptions). It forces a unit to display all digits even if it's all zeros.

We don't want showAllDigits for currencies because they generally have a lot of magnitudes, for instance, Ethereum has 18 which would turn `ETH 42.100000000000000000`. Instead, we want `ETH 42.1`.

At the opposite, it is commonly wanted that for popular fiats like EUR we will always display `EUR 42.10` and never `EUR 42.1`.

In live-common, our formatter is implemented by [`formatCurrencyUnit`](https://github.com/LedgerHQ/ledger-live-common/blob/master/src/currencies/formatCurrencyUnit.js) which takes a BigNumber value and a Unit (and many other options available).

### CryptoCurrency specific fields

The crypto currency level introduces many fields that are exclusively specific to crypto currency.

```js
type CryptoCurrency = CurrencyCommon & {
  type: "CryptoCurrency",
  // unique internal id of a crypto currency
  id: string,
  // define if a crypto is a fork from another coin. helps dealing with split/unsplit
  forkedFrom?: string,
  // name of the app as shown in the Manager
  managerAppName: string,
  // coin type according to slip44. THIS IS NOT GUARANTEED UNIQUE across currencies (e.g testnets,..)
  coinType: number,
  // the scheme name to use when formatting an URI (without the ':')
  scheme: string,
  // used for UI
  color: string,
  // a name for implementation (some coins share a common implementation like bitcoin or ethereum) - corresponds to live-common family folder name
  family: string,
  // the expected time of a block
  blockAvgTime?: number, // in seconds
  supportsSegwit?: boolean,
  supportsNativeSegwit?: boolean,
  // if defined this coin is a testnet for another crypto (id)};
  isTestnetFor?: string,
  // should be defined for bitcoin family
  bitcoinLikeInfo?: {
    P2PKH: number,
    P2SH: number
  },
  // should be defined for ethereum family
  ethereumLikeInfo?: {
    chainId: number
  },
  explorerViews: ExplorerView[],
  terminated?: {
    link: string
  }
};
```
<!--  -->
{% include alert.html style="tip" text="CoinType is generally found in <a href='https://github.com/satoshilabs/slips/blob/master/slip-0044.md'>github.com/satoshilabs/slips/blob/master/slip-0044.md</a>, although unicity is not quaranteed across networks (testnets, etc...)." %}
<!--  -->

### ExplorerView

You can define one or many explorers with urls, which will be used for external links in Ledger Live.

```js
type ExplorerView = {
  // url for transaction with $hash param
  tx?: string,
  // url for address with $address param
  address?: string,
  // url for a token with $contractAddress and $address params
  token?: string
};
```

<!--  -->
{% include alert.html style="tip" text="Find a reliable explorer. If you find more than one explorer, list them from most to less popular. Ledger Live will likely use the first explorer as default." %}
<!--  -->

### TokenCurrency specific fields

Tokens are also available through the cryptoassets package. ERC20, TRC10 and TRC20 tokens are automatically maintained on a regular basis. But if your blockchain supports tokens and you would like to add them, feel free to contact us.

```js
type TokenCurrency = CurrencyCommon & {
  type: "TokenCurrency",
  id: string,
  ledgerSignature?: string,
  contractAddress: string,
  // the currency it belongs to. e.g. 'ethereum'
  parentCurrency: CryptoCurrency,
  // the type of token in the blockchain it belongs. e.g. 'erc20'
  tokenType: string
};
```

## Adding MyCoin

Ensure that before integrating your coin to Ledger Live, your coin is well defined in [`@ledgerhq/cryptoassets/src/currencies.js`](https://github.com/LedgerHQ/ledgerjs/blob/master/packages/cryptoassets/src/currencies.js)


```js
 mycoin: {
   type: "CryptoCurrency",
   id: "mycoin",
   coinType: 8008, // The slip-0044 coin type if registered
   name: "MyCoin",
   managerAppName: "MyCoin", // name of the nano app in manager case-sensitive
   ticker: "MYC",
   countervalueTicker: "MYC", // depending on the counter value api
   scheme: "mycoin",
   color: "#E6007A", // color to be display on live-desktop and mobile
   family: "mycoin", // folder name in the live-common / desktop and mobile
   units: [
    {
       name: "MYC",
       code: "MYC",
       magnitude: 8,
    },
    {
       name: "SmallestUnit",
       code: "SMALLESTUNIT",
       magnitude: 0,
    },
  ],
   explorerViews: [
    {
       address: "https://mycoinexplorer.com/account/$address", // url for exploring an address
       tx: "https://mycoinexplorer.com/transaction/$hash", // url for exploring a transaction
       token: "https://mycoinexplorer.com/token/$contractAddress/?a=$address", // url for exploring a token address
    },
  ],
},
```
