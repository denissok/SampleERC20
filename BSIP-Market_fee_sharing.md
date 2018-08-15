BSIP:
    Title: Market fee sharing
    Authors: OpenLedger
    Status:
    Type:
    Created: 2018-08-13
    Discussion: https://github.com/bitshares/bitshares-core/issues/yyyyy
    Worker:

# Abstract

When creating a new asset, the asset issuer is the only beneficiary of the market fees in the current implementation. And the only way to affect the community growth is the market fee percentage. For example, one can decrease the market fee and it will result in somewhat larger number of trades with this asset. In this way the asset issuer might get a bigger profit during to increasing the trade volume with this asset. 

However, if we attract more people to BitShares we can bring additional money to BitShares, increase multiple currencies trade volumes and, hence, everyone will benefit much more (including asset issuer). 
So we suggest to add the possibility of sharing the market fee between the parties to stimulate attracting new members. Here's what we suggest. 

Asset issuer defines the reward - what percentage of the market fee he wants to share with the registrant. Registrant who attracted the user defines reward_percent for the referrer (using the already existing BitShares mechanism).
Market fee reward is accumulated on the user account. The user decides when they want to claim the market fee reward and move it to their active balance.  There is another operation created asset_claim_reward_operation. Each user pays network fee to call this operation.

# Motivation

Увеличить привлекательность выпущенной валюты, а вместе с ней увеличить количество сделок в данной валюте. Автоматически возрастает привлекательность биржи.

# Rational
При обработке 'fill_order_operation' в момент начисления market fee выполняется начисление reward аккаунту, являющемуся регистратором покупателя в размере reward_percent. Далее этот процент делится между registrar и referrer в соответсвии с настройкой аккаунта 'referrer_rewards_percentage', которая назначается при регистрации аккаунта, см  параметры register_account.

# Specifications

## 'reward_percent' asset's option
Percent of market fee that will be paid to buyer's referrer. Set by UIA issuer.

## 'market_sharing_whitelist' asset's option
TODO An optional whitelist (configurable by the UIA Issuer) could provide increased control over who is eligible to get market rewards.

## New database and wallet api methods

get_account_reward(account, asset_symbol) - Returns information about the reward amount for specific asset.
list_account_rewards(account) - Returns information about the reward amount in various assets
claim_reward(account, asset_symbol, amount) - Claim account's reward for the given asset.

## `account_reward_object`
```
account_reward_object {
    owner_account
    asset_type
    reward_amount
}
```

A new BitShares object type in implementation space impl_account_reward_object_type = 2.18.X that tracks the rewards of a single account/asset pair

## `account_reward_index`
```
account_reward_index multi-index of account_reward_objects
indexed_by
    [id]
    [owner_account, asset_type]
    [asset_type, reward_amount desc, owner_account]
```
 
A new index that stores objects of account_reward_object-type in graphene::database and allow random fast access to objects by given criteria

## `asset_claim_reward_operation`
```
asset_claim_reward_operation {
    fee
    claiming_account
    amount_to_claim
}

```

A new operation used to transfer reward to the account's balance.

## 'graphene::chain::database' new methods
    get_reward(owner_account, asset_id) - Retrieve a particular account's reward in a given asset
    adjust_reward(account, delta) - Adjust a particular account's reward in a given asset by a delta

## 'graphene::chain::database' modifications
    pay_market_fees(seller, recv_asset_object, receives_amount) - Split pay to asset issuer fee and referrer fee (market sharing fee) if referrer is whitelisted.
    calculate_market_fee(trade_asset, trade_amount)  - Calculate value for previous function.
    fill_limit_order(order, pays, receives, cull_if_small, fill_price, is_maker) - Append hardfork (HARDFORK_REWARD_TIME) check. Use old or new version of pay_market_fees() function.

## 'asset_create_evaluator' modifications
    Append hardfork (HARDFORK_REWARD_TIME) check. Validate additional asset options. Apply additional asset options (reward_percent, TODO: market_sharing_whitelist)

## Unit tests

Add: reward_tests.cpp (asset_rewards_test, asset_claim_reward_test)

Modified:  fee_tests.cpp, uia_tests.cpp


# Summary for Shareholders
Данная модификация будет интересна в первую очередь владельцам активов. Новый инструмент - market fee sharing - позволит привлечь новых клиентов за счет делегирования это задачи рефералам и увеличить объемы торгов с минимальными техническими затратами. 

# Copyright

This document is placed in the public domain.

# Sponsoring

This worker proposal is presented by OpenLedger