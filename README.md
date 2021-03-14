# RAI_UMA_Challenge

## Challenge
https://gitcoin.co/issue/reflexer-labs/geb/97/100024834

To build a synthetic asset tracking the
[Kovan RAI](https://github.com/reflexer-labs/geb-changelog/tree/master/releases/kovan/1.4.0/median/fixed-discount)
redemption rate movements using [UMA](https://docs.umaproject.org/build-walkthrough/build-process).


## Overview
### RAI

* **OracleRelayer**:- [Source](https://github.com/reflexer-labs/geb/blob/master/src/OracleRelayer.sol),
[Docs](https://docs.reflexer.finance/system-contracts/oracle-module/oracle-relayer)

* **RateSetter**:- [Kovan](https://kovan.etherscan.io/address/0x0641C280B21A31daf1518a91A68Ad396EcC6f2f0#code), [Events](https://kovan.etherscan.io/address/0x0641C280B21A31daf1518a91A68Ad396EcC6f2f0#events)

### UMA

#### Setup and Minting a Synthetic token

* Setup https://docs.umaproject.org/developers/setup
* Local install https://docs.umaproject.org/build-walkthrough/mint-locally
* EMP on net https://docs.umaproject.org/developers/emp-deployment
* Kovan addresses https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/42.json


#### Bots

* Bot parametrization https://docs.umaproject.org/developers/bot-param

## Solution
### Pricing Model
Redemption Rate is the rate at which RAI is being devalued or revalued, it can therefore be negative as well. So 



#### PriceFeed Implementation



#### Code & Tests
The full implementation of price feed with unit tests is contained in UMA protocol repo's fork.
https://github.com/ashutoshvarma/protocol

RaiRedemptionPriceFeed - [here](https://github.com/ashutoshvarma/protocol/blob/master/packages/financial-templates-lib/src/price-feed/RAIRedemptionRatePriceFeed.js)

Unit tests - [here](https://github.com/ashutoshvarma/protocol/blob/master/packages/financial-templates-lib/test/truffle/RAIRedemptionRatePriceFeed.js)

UMA's `Networker` class does not support sending POST requests which was nesseary in order to query subgraphs. To add support for POST requests I made few small changes to it. Here is the PR https://github.com/UMAprotocol/protocol/pull/2691


### Deployment and Testing
1. `RaiRedemptionPriceFeed` code has been tested with unit tests in the fork repo.
2. EMP UMA Contract (along with Synth Token) has been deployed to Kovan Testnet
3. Default price-feed configuration for has been added for bots to work with minimal configuration - [eacb633](https://github.com/ashutoshvarma/protocol/commit/eacb6338ab598d28e0a30fcf4050154087b159cd)
4. Uniswap Swap Pool (RAI-Synth) has been made to trade synths - [here](https://app.uniswap.org/#/swap?outputCurrency=0xCaC5B5AC9F4af1A4b73a12CD007A64BA4DFa07C2)
5. Deployed a Dapp to interact with EMP, manage position, deposit collateral, redeem synth and view positions or liquidations. (Fork from [emp-tools](https://github.com/UMAprotocol/emp-tools/)) - [Here](https://emp-tools-2391flb9z-ashutoshvarma.vercel.app/?address=0x08eA186755Ad743897c00AAfaEF7Fb9A7EcE8cf3)

### Setup & Configuration
#### Collateral Currency - RAI
Added buy UMA team to `
AddressWhitelist`. through this [transaction](https://kovan.etherscan.io/tx/0x006f18d76ba32ae42e2ca73eea703c9c5574c835773b447342bd46e71964ae6f)

#### Price Indentifier Name - `RaiRedemptionRate`
Added by UMA Team to `IdentifierWhitelist` through this [transaction](https://kovan.etherscan.io/tx/0x5cc0ccb70a86480af46385105d1b3e6318554df7c503ee43c055de77a0fb2b9b)

#### Synth Token - `RR-RAI-APR21`
```
syntheticName: "RAI Redemption Rate [RR April 2021]", 
syntheticSymbol: 'RR-RAI-APR21', 
```
Token Deployed (by EMP) at 
https://kovan.etherscan.io/token/0xcac5b5ac9f4af1a4b73a12cd007a64ba4dfa07c2


#### EMP Parameteres
```
Expiry date: 30/04/2021, 16:30:00 UTC
Price identifier: RaiRedemptionRate
Collateral requirement: 1.25
Unique sponsors: 1
Minimum sponsor tokens: 100.0 RR-RAI-APR21
```

EMP Contract Deployed at
https://kovan.etherscan.io/address/0x08eA186755Ad743897c00AAfaEF7Fb9A7EcE8cf3


