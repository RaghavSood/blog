---
title: "BTC Archeology: Where was 1.2 BTC sent in 2010?"
tags: bitcoin burned.money crypto-forensics
---

As part of building [burned.money](https://burned.money){:target="_BLANK"}, I've been trying to track down old cases of BTC that is likely lost.

On July 14th 2010 (UTC), bitcointalk user `ksd5` started a [thread](https://bitcointalk.org/index.php?topic=359.0){:target="_BLANK"} talking about how they could not locate 1.2 BTC after playing around with the wallet to test double spend protection.[[Bitcoin protects against double spending using the Unspent Transaction Output model - Each transaction output is uniquely identified by it's transaction ID and index in the output list (vout). Each `txid:vout` pair can only be consumed once in the entire blockchain. If a subsequent transaction claims to spend from an already consumed pair, it is invalid and any blocks containing it are invalid.::rmn]]

In summary, `ksd5` says that they:

1. Started with a wallet containing 1.21 BTC
2. Made a backup of that wallet by copying `wallet.dat` to a new location
3. With the original copy of the file loaded into the Bitcoin Client, they made a 0.1 BTC donation
4. They then closed the client and replaced `wallet.dat` with the backup copy
5. Upon starting the client, they briefly saw a 1.21 BTC balance, before it became 0 and showed a 1.21 BTC outgoing transaction

Notably, at step 4, they did not make a new backup of the in-use `wallet.dat`.

Early Bitcoin wallet managed addresses using a keypool, which pre-generated 100 addresses so that a backup would only have to be performed every 100 transactions or so. However, back in July 2010, this hadn't been implemented yet.[[It was only added in [October 2010](https://bitcointalk.org/index.php?topic=1414.0){:target="_BLANK"} by Satoshi Nakamoto.::lsn]]

As such, the Bitcoin Core client used by user `ksd5` would generate a new, random private key for each new address, which would then be stored in the `wallet.dat`.

When the first backup was made in Step 2, the wallet would have only contained the private key for the address holding the 1.21 BTC (and previously used addresses, if any).

Then, when the transaction was made in Step 3, a new private key was generated for the change address and stored in the `wallet.dat`.[[A change address is an address in a Bitcoin transaction that sends any unused funds back to the sender's wallet. For improved privacy, most wallets will use a new, unused address for this.::lsn]]

However, when the user copied the original backup back and overwrote the now-updated `wallet.dat`, this new key was lost. As far as the original backup was concerned, it had no knowledge of this change address, or the 1.2 BTC sent to it in the previous transaction.

Upon learning that the keys were lost, `ksd5` declined to attempt recovery using any disk recovery tools as to deal with such "small stuff".[[I'm not gonna bother trying to recover the file. It was only 1.20 BTC and I'm not gonna sweat the small stuff.<br/><cite>---[ksd5, July 14 2010](https://bitcointalk.org/index.php?topic=359.msg3006#msg3006){:target="_BLANK"}</cite>::rmn]]

The "small stuff", valued at ~0.012 USD in 2010, is worth ~73,000 USD today.

# So, where did it go?

With the history lesson accounted for, let's try and track down the BTC in question.

As a rule, [burned.money](https://burned.money){:target="_BLANK"} only tracks BTC where the public address is known. If I want to include this loss, I need to track down the address associated with this transaction.

The following searches were done on my local index of Bitcoin transactions, but for the purpose of this post I'll recreate them using [Blockchair](https://blockchair.com){:target="_BLANK"}.

## Narrowing down the candidates

We know that the user started with 1.21 BTC. While it is technically possible that this was held in multiple addresses, given the low value of BTC at the time, we can assume that it is a single output.

We're also going to assume that it was somewhat recently created - `ksd5` only signed up for bitcointalk two days prior, on July 12th 2010. Their other posts are indicative of it being early in their Bitcoin journey.

Lastly, we also know that the coins were spent - the user made an outgoing transaction for 0.1 BTC, which would result in the full 1.21 BTC being consumed (with 1.20 BTC sent to a change address, thus beginning this saga).

With these parameters, I decided to search for outputs that:

1. Are spent
2. Are exactly 1.21 BTC
3. Were created between 1 July 2010 and 15 July 2010

Blockchair's handy output listing and its filters allow us to [narrow it down to 3 outputs](https://blockchair.com/bitcoin/outputs?s=time(desc)&q=value(121000000),time(2010-07-01..2010-07-15),is_spent(true)){:target="_BLANK"}.

![Blockchair output search]({{ site.baseurl }}/img/120-btc-lost-blockchair-1.png){: width="100%" }

<table style="width: 100%; font-family: monospace;">
  <thead style="text-align: left; background-color: #cccccc;">
    <tr>
      <th>Block</th>
      <th>Block Time</th>
      <th>Address</th>
      <th>Value</th>
    </tr>
  </thead>
  <tbody style="text-align: left; background-color: #f0f0f0;">
    <tr>
      <td>67758</td>
      <td>2010-07-15 10:34:11</td>
      <td>1Ndgetj6fyvwf7ia5XpvrUgoy6gz5U4Wf5</td>
      <td>1.21 BTC</td>
    </tr>
    <tr>
      <td>67434</td>
      <td>2010-07-14 22:29:00</td>
      <td>1HRCRTBjdGPmbUVqQhhthBVyeAsGGFqXVJ</td>
      <td>1.21 BTC</td>
    </tr>
    <tr>
      <td>67246</td>
      <td>2010-07-14 14:21:18</td>
      <td>1PVGD99UorsotcKyZYKHSyMxTHpnth1vny</td>
      <td>1.21 BTC</td>
    </tr>
  </tbody>
</table>

## Picking the winner

We can disqualify the first output, as it took place after July 14th - by then, the bitcointalk post had already been made.

Between the remaining two candidates, we need to look at the actual transaction details.

`1HRCRTBjdGPmbUVqQhhthBVyeAsGGFqXVJ` sent 1.2 BTC to `16DsRFSCbv8aPMmDcbP4wYsS4FV9hf6rt2` and 0.01 BTC to `15VjRaDX9zpbA8LVnbrCAFzrVzN7ixHNsC`.

`1PVGD99UorsotcKyZYKHSyMxTHpnth1vny` send 1.2 BTC to `1BDjntPz95d9Ppv7qmWqqY8EUfjF94oQvR` and 0.1 BTC to `15jJwkaxssfrq6jo82tRbHBorvfQ46GgpC`.

At this stage, both look viable - the outgoing transactions match the bitcointalk post, and took place prior to the post being made.

However, we do have more information to filter with:

1. `ksd5` mentioned that they made a donation - while it is not specified to whom this donation was, we are looking at the very early days of Bitcoin. Donations for useful services and sites was commonplace, and donation addresses tend to see a lot of activity
2. `ksd5` declined to recover the coins - this means that at least the 1.2 BTC output must be unspent

Armed with the above facts, we can look into the outputs of the two candidates and further narrow it down.

## Where did all the coins go?

First, let's look at the second transaction.

![1PVGD99UorsotcKyZYKHSyMxTHpnth1vny outgoing transactions]({{ site.baseurl }}/img/120-btc-lost-trezor-1.png){: width="100%" }
*Image from [Trezor's Blockbook](https://btc2.trezor.io/address/1PVGD99UorsotcKyZYKHSyMxTHpnth1vny){:target="_BLANK"}*

The 1.2 BTC output sent to `1BDjntPz95d9Ppv7qmWqqY8EUfjF94oQvR` was subsequently [spent](https://btc2.trezor.io/tx/f9b53a53820d659220b7493a083669c08025423968f5e524a4967900635b8062){:target="_BLANK"} on 14th July 2010. While it is possible that `ksd5` managed to recover their keys after making the post[[Bitcoin Block Timestamps aren't 100% in line with real-world time, which can lead to discrepencies when comparing bitcointalk post timings against the blockchain.::lsn]] and spend the coins, it does seem unlikely for 0.012 USD.

The address involved in the 0.01 BTC output, `15jJwkaxssfrq6jo82tRbHBorvfQ46GgpC`, is a little more interesting. It also [received](https://btc2.trezor.io/tx/77f6d7385e9f7c7dbafbcf9fdf10d6c98f8591cc13e2edd153bb03131f5feb2a){:target="_BLANK"} 0.01 BTC in the same transaction that funded 1.21 BTC to `1PVGD99UorsotcKyZYKHSyMxTHpnth1vny`, and has seen 312 transactions. This is a good candidate for a donation address.

However, all of the transactions involving that address are for 0.01 BTC, and we see intermediate consolidation transactions combining all the BTC sent to it and [another address](https://btc2.trezor.io/address/1BiRpMGDef8rEDZU6RArzYmBGATQSG1zQp){:target="_BLANK"} (also receiving multiple 0.01 BTC transactions). The address was also active for only three days between 13th and 15th July 2010.

This behaviour doesn't match that of a donation address, as they tend to be long-lived and see more disparate amounts.

Based on this, I would disqualify `1PVGD99UorsotcKyZYKHSyMxTHpnth1vny` as the address used by `ksd5` to send 1.21 BTC.

Turning our attention to the second candidate, we see an outgoing transaction that meets our amount criteria.

![1HRCRTBjdGPmbUVqQhhthBVyeAsGGFqXVJ outgoing transactions]({{ site.baseurl }}/img/120-btc-lost-trezor-2.png){: width="100%" }
*Image from [Trezor's Blockbook](https://btc2.trezor.io/address/1HRCRTBjdGPmbUVqQhhthBVyeAsGGFqXVJ){:target="_BLANK"}*

Right away, this seems more promising - the 1.2 BTC output sent to `16DsRFSCbv8aPMmDcbP4wYsS4FV9hf6rt2` remains unspent as of writing this post, which aligns with `ksd5`'s post and the keys being lost.

The second address, receiving the purpoted donation, is `15VjRaDX9zpbA8LVnbrCAFzrVzN7ixHNsC`. This address has soon over 9000 transactions till date, with many varying amounts. It was first used on 11th June 2010, with the last outgoing transaction on 3rd July 2014.

With a little research on this address, we can find out that it was used by the very first Bitcoin Faucet, created by Gavin Andresen. The faucet would give out 5 BTC to anyone who asked for it, and was instrumental in the early days of Bitcoin.

A 3rd July 2010 [snapshot from archive.org](https://web.archive.org/web/20100703032414/https://freebitcoins.appspot.com/){:target="_BLANK"} shows `15VjRaDX9zpbA8LVnbrCAFzrVzN7ixHNsC` as the donation address for the faucet.

This lines up perfectly with the donation mentioned by `ksd5`. When combined with the 1.2 BTC output being unspent, we can say with a high degree of confidence that the address with the lost key is `16DsRFSCbv8aPMmDcbP4wYsS4FV9hf6rt2`.

# Burned BTC - where does it go?

As part of the [burned.money](https://burned.money){:target="_BLANK"} project, I've been tracking down lost BTC and categorising them based on their fate. The 1.2 BTC lost by `ksd5` is a prime candidate for the "Lost Keys" category.

"Burning Money" is a term that predates cryptocurrencies. Within the context of this space, it's taken on a new meaning - the act of rendering coins irrevocably lost, preferably in a provable manner. Bitcoin does support a provable burn in the form of an `OP_RETURN` output. However, many burn addresses have been created over the years, either for aesthetic purposes such as <span style="font-family: monospace;">[1111111111111111111114oLvT2](https://burned.money/script/76a914000000000000000000000000000000000000000088ac){:target="_BLANK"}</span>, <span style="font-family: monospace;">[1CounterpartyXXXXXXXXXXXXXXXUWLpVr](https://burned.money/script/76a914818895f3dc2c178629d3d2d8fa3ec4a3f817982188ac){:target="_BLANK"}</span>, or <span style="font-family: monospace;">[1BitcoinEaterAddressDontSendf59kuE](https://burned.money/script/76a914759d6677091e973b9e9d99f19c68fbf43e3f05f988ac){:target="_BLANK"}</span>, or by accident, such as by losing the keys to an address.

These, combined with several other ways to destroy BTC, contribute to a permanent reduction in the circulating supply of Bitcoin.

Having concluded this archaeological dig, I'll be adding the 1.2 BTC lost by `ksd5` to the [burned.money](https://burned.money){:target="_BLANK"} database under [16DsRFSCbv8aPMmDcbP4wYsS4FV9hf6rt2](https://burned.money/script/76a9143947b5155f8641756a90c935fbf61814a65ec4a288ac){:target="_BLANK"}.
