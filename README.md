# <pre>              RAI_UMA_Challenge by Reflexer Labs</pre>
<p align="center">
  <img src="Reflexer.jpg" width="350" title="hover text">
</p>

## <pre>                                   **Challenge**</pre>
https://gitcoin.co/issue/reflexer-labs/geb/97/100024834

To build a synthetic asset tracking the
[Kovan RAI](https://github.com/reflexer-labs/geb-changelog/tree/master/releases/kovan/1.4.0/median/fixed-discount)
redemption rate movements using [UMA](https://docs.umaproject.org/build-walkthrough/build-process).

## <pre>                                   **Overview**</pre>
<p align="center">
  <img src="UMA_square_red_logo.png" width="350" title="hover text">
</p>

UMA is a fast, flexible and secure way to create synthetic assets on Ethereum. UMA has defined a novel architecture
that enables anyone to create a synth asset that can track virtually anything from stock markets in Iran to gas 
fees on Ethereum safely and securely.

In this challenge I created a synth asset `RR-RAI-APR21` tracking RAI redemption rate.

Also a DApp(fork of UMAProtocol/emp-tools) to interact with my EMP and manage/create their positions.
https://emp-tools-2391flb9z-ashutoshvarma.vercel.app/


## <pre>                                   **Solution**</pre>

### Pricing Model
`Redemption_Rate` is the rate at which RAI is being devalued or revalued, it can therefore be negative as well. It is stored as `redemptionRate` in `OracleRelayer` relayer contract. Mathematically,
```
redemptionRate = Redemption_Rate + 1
```
Therefore `redemptionRate` is bound in (0,2) which makes it a sensible candiadate to track our synthetic asset. But `redemptionRate` changes very slowly and is almost constant upto few decimals. For example, these are some consecutive value for `redemptionRate` :-
```
0.99999999984948877039244582
0.999999999848547411861463422,     // Starting 10 decimals are almost consatnt
0.999999999848547690004497233.
```

To mitigate this for somme extent we can use `annualizedRate` which is scaled version of redemptionRate.
```
annualizedRate = (redemptionRate) ^ (365 * 12 * 30 * 24 * 3600)  
```

Lastly, to prevent market manuplation due to flash loans and other factors which can make `Redemption_Rate` volatile for very short period (which can casue sudden liquidations) we will take Time Weighted Average Price (TWAP) of `annualizedRate`.

#### PriceFeed Implementation
Source of data :- 
- https://subgraph-kovan.reflexer.finance/subgraphs/name/reflexer-labs/rai/
- `UpdateRedemptionRate` Event from [`RateSetter`](https://kovan.etherscan.io/address/0x0641C280B21A31daf1518a91A68Ad396EcC6f2f0#events) contract



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

EMP Contract Deployed  to Kovan using launch-emp scipts provided by UMA at
https://kovan.etherscan.io/address/0x08eA186755Ad743897c00AAfaEF7Fb9A7EcE8cf3


## <pre>                                  **LIVE Dapp**</pre>
Deployed - https://emp-tools-2391flb9z-ashutoshvarma.vercel.app/

Source - https://github.com/ashutoshvarma/emp-tools

![image](https://user-images.githubusercontent.com/17181457/111078364-06d27880-851b-11eb-869d-c1a416fc26d6.png)

A Simple DApp to interact with EMP, manage position, deposit collateral, redeem synth and view positions or liquidations.
(Fork from [emp-tools](https://github.com/UMAprotocol/emp-tools/))

#### Changes made to make it compatible with RAI EMP contract 
1. Disbale DevMining interfaces
2. Replace Old EMP ABI methods with updated one.
3. Add dummy price config for `RR-RAI-APR21` synth.

### Deployment Testing


## Refrence
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


