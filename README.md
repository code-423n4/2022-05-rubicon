# ✨ So you want to sponsor a contest

This `README.md` contains a set of checklists for our contest collaboration.

Your contest will use two repos: 
- **a _contest_ repo** (this one), which is used for scoping your contest and for providing information to contestants (wardens)
- **a _findings_ repo**, where issues are submitted. 

Ultimately, when we launch the contest, this contest repo will be made public and will contain the smart contracts to be reviewed and all the information needed for contest participants. The findings repo will be made public after the contest is over and your team has mitigated the identified issues.

Some of the checklists in this doc are for **C4 (🐺)** and some of them are for **you as the contest sponsor (⭐️)**.

---

# Contest setup

## ⭐️ Sponsor: Provide contest details

Under "SPONSORS ADD INFO HERE" heading below, include the following:

- [X] Name of each contract and:
  - [ ] source lines of code (excluding blank lines and comments) in each
  - [ ] external contracts called in each
  - [ ] libraries used in each
- [X] Describe any novel or unique curve logic or mathematical models implemented in the contracts
- [X] Does the token conform to the ERC-20 standard? In what specific ways does it differ?
- [X] Describe anything else that adds any special logic that makes your approach unique
- [X] Identify any areas of specific concern in reviewing the code
- [X] Add all of the code to this repo that you want reviewed
- [X] Create a PR to this repo with the above changes.

---

# Contest prep

## ⭐️ Sponsor: Contest prep
- [X] Make sure your code is thoroughly commented using the [NatSpec format](https://docs.soliditylang.org/en/v0.5.10/natspec-format.html#natspec-format).
- [X] Modify the bottom of this `README.md` file to describe how your code is supposed to work with links to any relevent documentation and any other criteria/details that the C4 Wardens should keep in mind when reviewing. ([Here's a well-constructed example.](https://github.com/code-423n4/2021-06-gro/blob/main/README.md))
- [ ] Please have final versions of contracts and documentation added/updated in this repo **no less than 8 hours prior to contest start time.**
- [ ] Ensure that you have access to the _findings_ repo where issues will be submitted.
- [ ] Promote the contest on Twitter (optional: tag in relevant protocols, etc.)
- [ ] Share it with your own communities (blog, Discord, Telegram, email newsletters, etc.)
- [ ] Optional: pre-record a high-level overview of your protocol (not just specific smart contract functions). This saves wardens a lot of time wading through documentation.
- [ ] Delete this checklist and all text above the line below when you're ready.

---

# Rubicon contest details
- $47,500 USDC main award pot
- $2,500 USDC gas optimization award pot
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2022-05-rubicon-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts May 23, 2022 20:00 UTC
- Ends May 28, 2022 19:59 UTC

This repo will be made public before the start of the contest. (C4 delete this line when made public)

## Contest Scope

The scope of this contest is the core of the Rubicon Protocol v1, an open order book and democratized liquidity system for Ethereum. *Rubicon Pools is a novel order book liquidity system and is the main focus of the contest*. This system makes it easy for LPs to enter single-asset liquidity positions, strategists to market-make & trade with that liquidity, and both LPs and strategists to profit. The core contract of the protocol is *RubiconMarket.sol*; this contract contains the order book that handles all ERC-20/ERC-20 trading on Rubicon.

Please see the 4 core contracts that are the main focus of this audit and more details below (those under “Core”); effectively this is `rubiconPools/` and `RubiconMarket`. Note, there are more minor contracts noted under “Periphery” - these are also included in the scope of the contest. 

### Resources

Here are a few resources to help you learn more about [Rubicon](https://rubicon.finance):

- [Live App on Optimism!](https://app.rubicon.finance/)
- [Whitepaper](https://github.com/RubiconDeFi/rubicon-protocol-v1/blob/master/Rubicon%20v1%20Whitepaper.pdf) - *Highly recommend you read this to learn all you need to know about Rubicon!*
- [Documentation](https://docs.rubicon.finance/)
- *[Live Deployments](https://docs.rubicon.finance/rubicon-docs/contracts/deployments)*

## Smart Contracts

### Core

### BathHouse.sol

This contract acts as the administrator of the Rubicon Pools system. It permissions strategists (via `approveStrategist`) that can access user funds (via `placeMarketMakingTrades` on `BathPair.sol`), manages key state variables across the protocol, and allows for the creation of Bath Tokens (via `createBathToken`). Importantly, BathHouse has an `admin` that is the EOA administrator of the entire protocol in v1.

### BathPair.sol

This contract is the strategist’s sole entry-point (via `placeMarketMakingTrades`) into Rubicon Pools in order to market-make (on `RubiconMarket.sol`) for the purposes of creating yield for themselves and LPs. The strategist cancels their outstanding orders (in part to maintain correct `outstandingAmount` accounting on `BathToken.sol`) while maintaining an active book of bids & asks. The strategist also uses Bath Pair to claim their rewards (via `strategistBootyClaim`), rebalance filled liquidity between pools (via `rebalancePair`), or rebalance filled liquidity with an external venue (via `tailOff`, today exchanging with UNI v3). Note the rebalancing process returns funds to their respective Bath Token and to the Bath Pair to await claim by strategists.

### BathToken.sol

This contract represents a single-asset (`underlyingToken`) liquidity pool that users `deposit` into, in order to receive Bath Tokens (`ERC-20 Token`). This token follows the `EIP-4626` share model. Users deposit funds to receive shares and redeem those shares when withdrawing to receive their funds back in addition to any yield strategists successfully generated for them. 

Please note, that the Bath Token tracks assets that it has a claim on, but have been placed by a strategist into the order via `outstandingAmount`. This is a crucial accounting measure so `underlyingBalance` tracks the best-guess ERC-20 value the pool has a claim on. This effectively mark-to-market’s the `outstandingAmount` to its amount when it left the Bath Token for Rubicon Market. 

The goal of the Bath Token is for liquidity providers to pay a service fee to strategists for generating yield for the Bath Token vault. Note that in v1, the strategist can return funds at a loss to LPs and their reward is realized whenever fill is handled by the strategists. In future versions, the strategist role will be decentralized, bond assets when market-making with LP funds, and pay rewards relative to returned profits.

Note that the Bath Token looks to both the Bath House (via `onlyBathHouse`) and the Bath Pair/strategists (via `onlyPair`) as permissioned actors. The Bath House admins key storage variables while strategists access funds and manage liquidity positions. Special care should be taken to make sure no malicious actors can access ERC-20 funds, namely the `underlyingToken`,  of Bath Token.

### RubiconMarket.sol

This is the core order book contract of the protocol. Rubicon Market allows for easy order book trading of any two ERC-20 assets. Here is [live documentation](https://docs.rubicon.finance/rubicon-docs/contracts/rubicon-market) as well as documentation made by [OasisDex](https://oasisdex.com/docs/guides/introduction).

### Peripheral

### RubiconRouter.sol

This contract acts as a convenient router for calls that access the core protocol. Features of the Router include the ability to `swap` between any two ERC-20s by simply trading through `RubiconMarket`. Moreover, the Router provides helpful queries like `getBookFromPair` and has wrappers to support native asset (like ETH or MATIC) interactions with the core protocol.

### BathBuddy.sol

This contract makes it easy for Bath Tokens to payout their withdrawers any `bonusTokens` they may have accrued while staking in the Bath Token (e.g. network incentives/governance tokens). This contract is a minor modification of the [Open Zeppelin Vesting Wallet contract](https://docs.openzeppelin.com/contracts/4.x/api/finance#VestingWallet). BathBuddy `release`s a user their relative share of the pool’s total vested bonus token during the `withdraw` call on `BathToken.sol`. This vesting occurs linearly over Unix time and is unchanged from the OZ contract; modifications were only made to the custom `release()` implementation.

### Notes

### Transparent Upgradeable Proxy Usage

Please note, in practice, Rubicon uses Transparent Upgradeable Proxies to make sure all contracts can be iterated and improved over time. Please see this helpful resource for learning more about the Proxy Upgrade Pattern. A Rubicon admin (different from the contract admin that acts as BathHouse admin) acts as the proxy admin for all smart contracts in v1. 

Please note that there are some deprecated storage variables that are *intentionally left in core contracts to allow for consistent contract abis across networks and respect the proxy upgrade pattern on Optimism.* These storage variables will not be paid any bounty and are included for consistent contracts cross-network despite additional deployment costs.

### Areas of Focus for Wardens

- Any *loss* of or *malicious*, non-strategist use of *LP funds* in `BathToken`. Any non-approved strategist (chosen by admin EOA) moving LP funds (excluding the rightful owners) is an error in v1.
- Bath Tokens should account for their underlying assets and payout depositors correctly when withdrawing.
- Gas optimization
- Re-entrancy or potentially malicious external calls
- No outside, non-approved actors should modify any state variables within the system. Note, that this should only be possible in `openBathTokenSpawnAndSignal` in which case the user-created Bath Token is added to the system, but still only manageable from a strategist perspective from system-approved strategists
