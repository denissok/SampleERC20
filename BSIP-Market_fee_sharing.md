```
BSIP:
Title: Market fee sharing
Authors: OpenLedger
Status:
Type:
Created: 2018-08-13
Discussion: https://github.com/bitshares/bitshares-core/issues/yyyyy
Worker:
```
 

# Abstract
 

When creating a new asset, the asset issuer is the only beneficiary of the market fees in the current implementation. And the only way to affect the asset community growth is the market fee percentage. For example, one can decrease the market fee and it will result in somewhat larger number of trades with this asset. In this way the asset issuer might get a bigger profit during to increasing the trade volume with this asset.
 

However, there might be another opportunity to promote the asset and stimulate the trading - use native Bitshares referral program. At this time unfortunately an assets owner is not able to share market fees with registrars and referrers to stimulate them to market the asset trading, so we suggest to add this possibility. Furthermore, enabling this feature for MPAs (e.g. bitCNY or bitUSD) can provide additional bounty for Bitshares registrars and referrals which can lead to more traders joining to the ecosystem.
The workflow may be as following:

An asset issuer defines the market_fee_reward_percent - what percentage of the market fee he wants to share with the registrar in asset options. Registrar defines reward_percent for the referrer for each user(using the already existing BitShares mechanism).
Market fee reward is accumulated on the user account. The user decides when they want to claim the market fee reward and move it to their active balance. There is another operation created asset_claim_reward_operation. Each user pays network fee to call this operation.
 

# Motivation
 

To make promoting BitShares and bringing new users much more attractive to registrars and referrers by sharing UIAs and MPAs market fees with them.
 

# Rationale
When 'fill_order_operation' executing at the moment of market fee calculation there will be calculate the reward for the registrar used the parameter market_fee_reward_percent. Then this market_fee_reward_percent of the market fee is split between the registrar and referrer according to the referrer_rewards_percentage, which is set up during the new account registration by registrar (please see the parameters for register_account).
 

# Specifications
 

## market_fee_reward_percent asset option
Percent of market fee that will be paid to buyer's registrar. Set by UIA issuer.
 

## market_sharing_whitelist asset option
 An optional list of accounts (configurable by the UIA Issuer) could provide increased control over who is eligible to get market rewards.
 

## New database and wallet api methods
**get_account_reward(account, asset_symbol)** - Returns information about the reward amount for specific asset.
**list_account_rewards(account)** - Returns information about the reward amount in various assets
**claim_reward(account, asset_symbol, amount)** - Claim account's reward for the given asset.
 

## account_reward_object
A new BitShares object type in implementation space impl_account_reward_object_type = 2.18.X that tracks the rewards of a single account/asset pair
```
account_reward_object {
    owner_account,
    asset_type,
    reward_amount
}
```
## account_reward_index
A new index that stores objects of account_reward_object-type in graphene::database and allow random fast access to objects by given criteria
```
account_reward_index multi-index of account_reward_objects
indexed_by
    [id]
    [owner_account, asset_type]
    [asset_type, reward_amount desc, owner_account]
```
## asset_claim_reward_operation
A new operation used to transfer reward to the account's balance.
```
asset_claim_reward_operation {
    fee
    claiming_account
    amount_to_claim
}
```


## graphene::chain::database new methods
**get_reward(owner_account, asset_id)** - Retrieve a particular account's reward in a given asset
**adjust_reward(account, delta)** - Adjust a particular account's reward in a given asset by a delta
 

## graphene::chain::database modifications
**pay_market_fees(seller, recv_asset_object, receives_amount)** - Split pay to asset issuer fee, registrar fee and referrer fee. If the registrar is not in the market_sharing_whitelist, split_pay will not happen and the entire fee goes to Asset issuer.
**calculate_market_fee(trade_asset, trade_amount)** - Calculate value for previous function.
**fill_limit_order(order, pays, receives, cull_if_small, fill_price, is_maker)** - Append hardfork (HARDFORK_REWARD_TIME) check. Use old or new version of pay_market_fees() function.
 

## asset_create_evaluator, asset_update_evaluator modifications
Append hardfork (HARDFORK_REWARD_TIME) check. Validate additional asset options. Apply additional asset options (market_fee_reward_percent, market_sharing_whitelist)
 

## Unit tests
 

Add: reward_tests.cpp (asset_rewards_test, asset_claim_reward_test)
 

Modified: fee_tests.cpp, uia_tests.cpp



# Summary for Shareholders
The new modification - market fee sharing - will allow to bring in new clients for BitShares by making it financially lucrative for registrars and referrers. This modification is interesting mostly so asset issuers and registrars/referrers.
 

# Copyright
This document is placed in the public domain.
 

# Sponsoring
This BSIP is presented by OpenLedger
