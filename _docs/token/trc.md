---
title: TRC10 and TRC20
subtitle:
tags: [tokens]
category: Tokens
toc: true
author:
layout: doc
---



## How to request a token addition

### 1. Is my TRC10 or TRC20 token listed?

First, make sure your token isn’t already listed by checking this table of the [supported crypto-assets](https://github.com/LedgerHQ/ledger-live-desktop/blob/develop/cryptoassets.md).

<!--  -->
{% include alert.html style="tip" text="The list of supported assets on the <a href='https://www.ledger.com/supported-crypto-assets/'>website</a> is managed through an internal Ledger process. For the moment it’s difficult to guarantee the exact mapping between tokens available on Ledger Live and listed on the website." %}
<!--  -->

### 2. How to get my token listed?


|     TRC10       |     TRC20     |
|-----------------|---------------|
| 1. Update the [ledgerjs package](https://github.com/LedgerHQ/ledgerjs/blob/master/packages/cryptoassets/data/trc10.js).| 1. Update the [ledgerjs package](https://github.com/LedgerHQ/ledgerjs/blob/master/packages/cryptoassets/data/trc20.js).|
| 2. A specific signature is required on the [Tron Nano App](https://github.com/LedgerHQ/app-tron). | 2. Update both `src/tokens.c` and `src/tokens.h` in the [Tron Nano App](https://github.com/LedgerHQ/app-tron). |

<!--  -->
{% include alert.html style="important" text="To ensure both these topics are handled properly, we invite you to contact our engineering team through the <a href='https://discord.gg/Ledger'>Ledger's Discord server</a>" %}
<!--  -->

### What next?

This is all you need to do. We will review the list of tokens on a bi-montly basis and will apply our own internal signature process. Once signed, the new tokens will become available after a Ledger Live update. Ledger reserves the right to decide which token will be listed.
