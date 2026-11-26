# DXD specification

**Status:** Launch draft. This text matches the dogex indexer. A second implementation that disagrees with dogex is not DXD.

The key words MUST, MUST NOT, SHOULD, and MAY are RFC 2119.

## 1. What a pool is

A pool is two reserves and a share count. One reserve is DOGE, in koinu. The other is one fungible: a Ðune, a ÐogeTreat, or a DRC-20. The product of the two reserves is the invariant.

The reserves are not inside a contract. They are outputs at a **vault address** that the pool names. The inscription is the order. The indexer is the book. Delivery is a later transaction signed by the vault key.

The indexer MUST NOT hold that key. dogex records what the vault owes. command.dog, or any other isolated signer, pays it. A broadcast of the user's inscription is not the payout.

## 2. Trust

Dogecoin has no covenant that can stop the vault key from spending the pool. An indexer MUST ignore a swap, an add, or a removal that does not match this spec. The key holder can still move the coins. Operators publish the vault address. Users can watch it.

This is the same class of trust as a launch treasury. It is not a two-party PSBT. DRC-20 and Doginal listings that use a PSDT are a different trade: both sides sign one transaction, and no vault is in the middle. DXD is the pool.

## 3. Inscription

The body is JSON. Parsers MUST start at the first `{`. Bytes before that, including a leftover metaprotocol prefix, are ignored.

`p` MUST be `dxd`, compared case-insensitively. `v` MUST be the string `1`.

A legacy body with `p` = `pair-v1` is accepted. Its ops `create`, `add`, `remove`, and `swap` map to `create-pool`, `add-liquidity`, `remove-liquidity`, and `swap`, with `fee_tier` 30.

`addr` MUST be 26 to 35 characters. Amounts MUST be base-10 integers, as strings or numbers. Ticks MUST be 1 to 64 characters. On `create-pool`, ticks are lowercased.

## 4. Assets and pool id

Each leg is an object:

| Field | Meaning |
| --- | --- |
| `protocol` | `doge`, `dt`, `dune`, or `drc20` |
| `genesis` | The lane's anchor. Display ticker is not the identity. |
| `tick` | Display only. Optional. |

`doge` genesis MUST be `native`. A Ðune genesis is `block:tx`. A ÐogeTreats genesis is a ÐA (`block:tx`), a deploy txid, or an inscription id. A DRC-20 genesis is the deploy inscription id.

The canonical id is `{protocol}:{genesis}`, genesis lowercased.

The pool id is the two canonical ids sorted lexicographically and joined with `~`. Example: `doge:native~dune:6324238:6`.

Ðalkanes are not a DXD asset. A pool does not custody a `0xD1` parcel.

## 5. Operations

### 5.1 `create-pool`

Required: `tick0`, `tick1`, `asset0`, `asset1`, `fee_tier`, `addr`.

`fee_tier` MUST be `30` or `100` (basis points taken on the way into the curve). Optional `creator_fee_bps` is capped at 100. Optional `nft_tick` marks an NFT pool.

Creating a pool does not move reserves. Liquidity is a later `add-liquidity`.

### 5.2 `add-liquidity`

Required: `pair_id`, `tick0`, `tick1`, `amt0`, `amt1`, `addr`. Optional `amt0_min` and `amt1_min` default to 0.

The user pays `amt0` koinu to the vault on the reveal and edicts `amt1` of the token to the vault. The indexer credits liquidity shares only when both are present. Amounts MAY be pulled to the pool ratio. Unused payment is refunded to `addr`.

The first liquidity locks 20 shares in the pool. Those shares MUST NOT be withdrawn.

### 5.3 `swap`

Required: `pair_id`, `tick0`, `tick1`, `amt0`, `addr`. Optional `amt1_min` defaults to 0.

`tick0` is the side the user pays. `amt0` is that amount. `amt1_min` is the least they will accept of the other side.

DOGE in: the koinu is on the reveal, paid to the vault. Token in: the token is edicted to the vault first. The other side is delivered afterward by the vault signer, to `addr`.

If the quote is below `amt1_min`, the swap does not run. A paying op that names `dxd` and `addr` but does not parse still refunds the payment it carried.

### 5.4 `remove-liquidity`

Required: `pair_id`, `tick0`, `tick1`, `liquidity`, `addr`. Optional `amt0_min` and `amt1_min` default to 0.

Burns `liquidity` shares and owes both sides to `addr`, subject to the minimums. A pool marked locked at graduation MUST reject removal. ÐLaunch graduation LP is locked.

## 6. Curve

Let `F = 10000`.

The amount that enters the product uses the pool fee tier `f` (`30` or `100`):

```text
amount_in_for_k = amount_in * (F - f) / F
amount_out = amount_in_for_k * reserve_out / (reserve_in + amount_in_for_k)
```

`amount_out` MUST be strictly less than `reserve_out`.

What is added to the input reserve is smaller. The protocol fee is 5 bps. A creator fee, if the pool has one, is at most 100 bps. Together they are skimmed from the input. The protocol fee is taken first. The rest of the skim is the creator fee. The input reserve increases by `amount_in` minus that skim. The output reserve decreases by `amount_out`.

## 7. Delivery

Indexing a swap records a debt. It does not broadcast the payout.

The signer builds the payout from the indexer's unsigned preparation, signs with the vault key, and broadcasts through Dogecoin Core. A relay that returns a txid the indexer cannot see is not delivery.

One transaction carries one `OP_RETURN`. A Ðune payout is a dunestone. A DOGE payout is a normal output. They are separate transactions when both would need their own message.

## 8. What this is not

DXD does not custody Ðalkanes. A Howl or Bork sale that should be atomic between two people is a PSDT: the seller signs their output and the DOGE they want, and the buyer adds a version `0x02` parcel. That trade does not pass through this vault.

The older ÐSwap vision (order books, emissions, ve-governance) is not this protocol.
