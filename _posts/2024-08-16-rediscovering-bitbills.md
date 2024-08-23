---
title: "Rediscovering Bitbills - The First Physical Bitcoin"
tags: bitcoin collectible.money crypto-forensics crypto-collectibles crypto-numismatics
---

In May 2011, an innocuous post[[[bitcointalk.org/index.php?topic=7724.0](https://bitcointalk.org/index.php?topic=7724.0){:target="_BLANK"}::lsn]] appeared on the bitcointalk forum. Titled "Introducing Bitbills!", it became the first stepping stone in what grew to be a thriving collectibles market in the Bitcoin space.

Described as "the first physical incarnation of bitcoins", the premise was simple - A Bitbill is a small, credit card sized plastic card with a private key embedded in it. The private key is hidden behind a tamper-evident hologram, and can be revealed by peeling off the hologram. The card also has a public key printed on it, which can be used to verify the balance of the card.

The cards were produced in denominations of 1, 5, 10, and 20 BTC. Intended to be a "bearer instrument", the concept was simple - you don't have to teach someone how to set up a bitcoin wallet, nor do you have to wait for multiple confirmations. Just hand over the card, and as long as the tamperevident hologram is intact, the recipient can be sure that the card has not been spent.

Given that at the time 20 BTC was approximtely $75, the mode of attack where someone might reveal the key and manufacture a counterfeit card was not considered to be cost effective for the reward.

The cards had moderate success - they were produced for nearly a year, with confirmation that production had ceased as of 15th May, 2012.

As time passed, more prominent collectibles appeared - Casascius coins, alongside other notable mentions like Satori and BTCC, produced tens of thousands of pieces. However, Bitbills remained unique - they were the first.

There was just one problem - just how rare are they? In a collectibles market, scarcity drives a significant portion of premiums, and Bitbills were so early that they didn't know they were collectibles. In latter years, most creators published address lists, or at least mintages, to let the market make an informed choice. However, Bitbills were more novelty than precious. They were handed out for free at meetups.

To unravel this, I turned to on-chain evidence. All Bitcoin transactions are recorded permanently on the blockchain, and with a little bit of data analysis, we can determine how many Bitbills were produced.

## On-chain forensics - Tracking down Bitbills

My starting point is to look at the funding source for a Bitbill. I picked one that I knew was definitely a real Bitbill, and looked at the transaction history for its address. `1EfjzEoKYp91koXBY8MazraUcapy9mRpLA` is a 1 BTC Bitbill, funded in transaction `e912b3a38015b72a38c43e158f78940818908164f988c81bc98f94d7e106fc76`.

This immediately gives us a wealth of information. The transaction appears to fund 141 potential Bitbills (and 1 change output). The transaction originates from `1FhTe1bMtoKHDbNw13v3BQ9sb2kFTagaRH`. The change is sent to a different address, `19XFbTZ2K3JEW2y9GFbeLyikfaKtEUNTds`.

Looking at the origin address more closely, we see many similar looking transactions - several outputs matching Bitbill denominations, and a singular change output.

Based on this, we can say with some degree of confidence that all Bitbills are funded by transactions originating from `1FhTe1bMtoKHDbNw13v3BQ9sb2kFTagaRH`, or from a change output that can be traced back to it.

Thus, discovering the total mintage of Bitbills becomes an exercise in following the transactions.

I wrote a script that broadly works as follows:[[I'm not the first person to use this approach - Charlie Lee followed a [nearly-identical process](https://bitcointalk.org/index.php?topic=3334918.msg50843287#msg50843287) to track down the very same Bitbills in 2013. I only differ in not excluding outputs under one BTC. However, this does not impact the discovered set of addresses, and both Charlie and I end up with the same Bitbill list.::rmn]]

1. Start with the known Bitbill address.
2. Look at outgoing transactions from that address.
3. For each output, categorize it as a Bitbill based on the denomination. The odd-one-out is the change output.
4. For each change address, repeat the process.

This lets us recursively discover change addresses, and build a list of Bitbill addresses.

Running this script produced the following tree:

```bash
1FhTe1bMtoKHDbNw13v3BQ9sb2kFTagaRH
  └── 367fa61d75e0581d45fd21c77a17d6c84b3cf63cc401a619ee4754dbd9770336
    └── 1CydWn65Hydy3bk2sja4ca7TNhKgo3ZRya: 4.09000000 BTC
    1CydWn65Hydy3bk2sja4ca7TNhKgo3ZRya
      └── c64d75151a170e7e18241395efde31e838529ba2a1064de5d392a3e03333715b
        └── 1NVwABgxcThGR3B7uzSEuLmwSfeKTJzkUL: 3.08000000 BTC
        1NVwABgxcThGR3B7uzSEuLmwSfeKTJzkUL
          └── 460cf68924d01f4ad52f04917d0a62d8f0aedf2b09024c9d8fb95635d6242508
            └── 1oXcPjQXMPRjUAybP4AUaHcYv1QxvGnCG: 1.00000000 BTC
            └── 13qLxBe75y3CoLc1Yj5xHvGG2LtPpoZNgJ: 2.07000000 BTC
            13qLxBe75y3CoLc1Yj5xHvGG2LtPpoZNgJ
              └── 37f63da2acd5a1d7f6d7632661f63fe097d0f74dadbf385460cabc2c13699fef
                └── 1JSmbryKkZPceMhfRH1bg3nW2pVzsARMLG: 1.00000000 BTC
                └── 1PTmNzMWktFF1PbjULEmzpKG29jfE7bYqU: 1.06000000 BTC
                1PTmNzMWktFF1PbjULEmzpKG29jfE7bYqU
                  └── 48ac7c6ce9493e2ba6f86f1d439d5a06cb1089050741bbf78ca254b9a48993da
                    └── 1FdHUdCjaRT9UbsztMDDDJsvRihw2VhkEb: 0.05000000 BTC
                    1FdHUdCjaRT9UbsztMDDDJsvRihw2VhkEb
                      └── 69a39eb32d93639537e6f3321fd60e608225272699a09dad5f511f38139bd6fb
                        └── 1Q3yzUYcrMwugWMmnkehoKhM14eftmcdjA: 20.00000000 BTC
                        └── 13zFbf7T6YH451M4XuHrhYxjvYLV7GVzFe: 20.00000000 BTC
                        └── 1DKVbMk66tfNN13GMekSsnaEsrKPNVAJ9e: 20.00000000 BTC
                        └── 1Kt4RRr7a31PAd1QmnU5H2rFUjBo6NAq8e: 20.00000000 BTC
                        └── 189Eioo3X61C2SGqLRGbN5DJw5kcx7R3aJ: 20.00000000 BTC
                        └── 12svwJEMQQmh7YvmTHS4dzXjG2ft9GoN8B: 20.00000000 BTC
                        └── 14oYkqAwgqE9Sy5BjJF7MoAL8HQB6QW5gq: 20.00000000 BTC
                        └── 16ySADfFpG2UGLm81bWvuHFN6SKuot9WyL: 20.00000000 BTC
                        └── 17vm5mFEieuZKyXueemwnp5XsQZdgVWr29: 20.00000000 BTC
                        └── 1Kz52qVU4h5xqSsPfM929KfJa9Bxr5z141: 20.00000000 BTC
                        └── 1AyTyB34UGfxdSdeUruWp6DmaBGDy6RByb: 20.00000000 BTC
                        └── 1Cf8RTXZz4fHuMT832wmZa7hesF5FHgYXa: 20.00000000 BTC
                        └── 15uSGUQ1Jw6VPobnsdZMqP3U3AdCJBFwZj: 20.00000000 BTC
                        └── 1DqEotXCZJ5TgY3SpLLWwhq4vZkJ1LLZiH: 0.01700000 BTC
                        1DqEotXCZJ5TgY3SpLLWwhq4vZkJ1LLZiH
                          └── 88c5e1c51c66ff0e4048911eab494daa07938cf92fba3e972d79c5f3465aebd8
                            └── 1PVZG6Sa6DQnSWaz2FGk5FeUrfG4xi7m12: 1.00000000 BTC
                            └── 1NjFgqyCPhqMcEZz5y8T276RkY7tEbu4we: 1.00000000 BTC
                            └── 13BWXeFq6H9tTSF71PhKfN6XBjyaTxxYg5: 1.00000000 BTC
                            └── 1LQwUhQTtTFBEFV41ToecwmhSpasXKUzsX: 1.00000000 BTC
                            └── 1LN3WN32M42jrsDmo2EhHSwko5rGqGG332: 1.00000000 BTC
                            └── 1AcRdqWy4CjnsUofgDbskRAJJwFsYALHga: 1.00000000 BTC
                            └── 18ZjoW9KMEVL8LVpRicDXQCriw5JZHqKcQ: 1.00000000 BTC
                            └── 1CukUy9KuRjAZ1HKtCoZQZeTcwmz57RYe2: 1.00000000 BTC
                            └── 1DNTLXsxN35vXaVq6zxtFkDw2vreNeYeyE: 1.00000000 BTC
                            └── 1DinHSQjvogApZnWQze1BnPVZoRUZvgKVR: 1.00000000 BTC
                            └── 1FavxeFH9qeZEj9v2mMP83c1Svq6A5EG4B: 1.00000000 BTC
                            └── 1J9gxeE1nsH2qr9qjaawKnbxzrTKHkM6Fp: 1.00000000 BTC
                            └── 1A3tCw8eAAXqRJgUszmTC863KodkPGDQ2d: 1.00000000 BTC
                            └── 1EBfy7cVDYpKNHUfawqiVzVdVQW9Qv29C4: 1.00000000 BTC
                            └── 1LqZtzAAHRR6L7pouB9R1JyXkDZFymBiiZ: 1.00000000 BTC
                            └── 142ZiGQzpk4Mt2k1TvE1W7cSqqMinKFaCa: 1.00000000 BTC
                            └── 181Af534iERPvtXzo8SypVevMNUqMfG6nu: 1.00000000 BTC
                            └── 1HVNTvcwMHSsBHhf3m7NJnGVqxtPijkWyy: 1.00000000 BTC
                            └── 1LKSB5mDXjb1sJFtrRJ6AmfFVbWSPWwNKV: 1.00000000 BTC
                            └── 1Ne6tk6xS7QGfwM1Z9MaE5JyFhpX2W459h: 1.00000000 BTC
                            └── 12UFKkcdfXSudL8tgPkgMjBQQGPR3gb2MD: 1.00000000 BTC
                            └── 1DtCS3PtHHjpu5sG3iWptK6utzrC76RYxT: 1.00000000 BTC
                            └── 1Pe3xi8mzsfZxjqj5HhqcyFzGb1LJE1Fsj: 1.00000000 BTC
                            └── 18jxSCFW9nyo5qLb7yBB2oJQsEbe5ad3NY: 1.00000000 BTC
                            └── 19cwJ6KuWWsqcEcYXuoxt6xirVkWWLf4pg: 1.00000000 BTC
                            └── 1Fehz9mNyAnrAZZdY7rxsMH5u1v1WXhWZe: 1.00000000 BTC
                            └── 12CT1wHCDLCnwg5ktqbxSVWC5BFM5ESaQX: 1.00000000 BTC
                            └── 1GALfcSrGT2xHEdUvNMYQ4Ejhfejz8oudC: 1.00000000 BTC
                            └── 1Hm5zhGDsfBcpNMwzWi5zuEvyVDY2P5NsE: 1.00000000 BTC
                            └── 14fbKUcjNiZc5p7vsxFbqSRBbu7Mc8LMtJ: 1.00000000 BTC
                            └── 1N2FiuRoEstuW5zt9T9M5rZG1PseYtajWi: 0.01400000 BTC
                            1N2FiuRoEstuW5zt9T9M5rZG1PseYtajWi
                              └── d12b522c33ca1d22d070af08ab12280e793f4134c289b834de91e79b2f0f6247
                                └── 1KxMfCMeyARRCFNgNP6ZXTKCSFEhAZBYqp: 0.01350000 BTC
                                └── 15LAYfbJRHdfYAhmjWToCA8eJQMUeVq6ws: 1.00000000 BTC
                            └── 1WABVYtgbEGk4WdvnjoVKEsx8cHwi9uWd: 1.00000000 BTC
                            └── 13bc26x5hqXZaYyz6YoHCG75rKq3MDcBgS: 1.00000000 BTC
                            └── 14FSdoALMFVzD1jLapd8AWxJUNk7HdVSRb: 1.00000000 BTC
                            └── 18389vPe7S58AHEpQcDzEoTMvWpFRtqpgm: 1.00000000 BTC
                            └── 1EJ1gKWXgWZddxYCgxgnnzsAh73sAGVobK: 1.00000000 BTC
                            └── 1Kpm9wbbqAM2Xw1nnsCNnp94RBdxfztG6M: 1.00000000 BTC
                            └── 1ATChD1Aw4PkJV4i3ZXM126D9HUiK5its5: 1.00000000 BTC
                            └── 1J36o61grzs9ZoE9DEicRj3sKdBtZ8THTa: 1.00000000 BTC
                            └── 1KAEbnRf2NBPm934XJgTddMZs3ZbKoh7Wm: 1.00000000 BTC
                            └── 15GqEnkmZSefuyF3gHMVozDrDR6ckkcNrU: 1.00000000 BTC
                            └── 13pk2b2yDK4HAHRs3GEXczmPYA5Yx7My4q: 1.00000000 BTC
                            └── 19hRG4DL9NrPAJR7QYFfSJxHrSjbM6idPB: 1.00000000 BTC
                            └── 1NWAhJPYB2sxir51ynveMUC7un5aj6PmNL: 1.00000000 BTC
                            └── 1HB15oV7k27iXfyLqtmUpGGDTmtgo1E2Pt: 1.00000000 BTC
                            └── 15gaoDGMZ9Jsq9hSRNHaTDehsd34oKEtua: 1.00000000 BTC
                            └── 17hZac21uJ4FrFGsjex3Na4WQ9Smus2Asq: 1.00000000 BTC
                            └── 18SjBDJCX8wXCmqaiY1fR8bR6TgovVyCt2: 1.00000000 BTC
                            └── 16zWS1tiWNmqRE2c3zTPG1a3V1tDSwwqyu: 1.00000000 BTC
                            └── 1Egx48tYRhTwUL2bCn8V8kAY185h7N8vis: 1.00000000 BTC
                            └── 1QCcvT8XxQQPcPCs9b7GEtdVX8tZBfv4AC: 1.00000000 BTC
                            └── 1Bvj3Y4xdjJ2HfHaoo3aTSVyt7pFkCYLhF: 1.00000000 BTC
                            └── 12KamM2fyweJwPV5khXyzH5mSXfPBTBW1G: 1.00000000 BTC
                            └── 193yPtrTu968FfZU1mzLhDkmqKaT7iY6WZ: 1.00000000 BTC
                            └── 1FCn4W7Vzi16M6X7JHo9D6T1ivtsG5scpg: 1.00000000 BTC
                            └── 1KG4SaRUvCaceL8o3k7uWv3dZZFbSgBtkj: 1.00000000 BTC
                            └── 1B6EbFUiG4dHoNQsJZ5ExXPvartFfb6G4Q: 1.00000000 BTC
                            └── 18CX1UiD9bEtgA4ryMv4t4kqL3ZEhXBubz: 1.00000000 BTC
                            └── 1K6CTnb76856RFpd2Tm841ZBePCW1iXRrc: 1.00000000 BTC
                            └── 1Em6HGdY55PAqNgLNJTBsBHWSJt35QXNWc: 1.00000000 BTC
                            └── 1Q9ChdRvhhgRL1f2u8piXVgCfFN2ejSgiB: 1.00000000 BTC
                            └── 1AN1bsc5zo9Mn7XtdtCdyMZyvs1UMWpTNr: 1.00000000 BTC
                            └── 12GnxoXDrGZRQC2NGGbP7h4cnn9yunTdLn: 1.00000000 BTC
                            └── 1PBGwHD2T95QR4g5z93uz6yqA6n6GjdpdY: 1.00000000 BTC
                            └── 14URKVRJzwmSdZkG5C3MuRsEtmNSLpEGTm: 1.00000000 BTC
                            └── 13sH4nbcc4j5zgHzLPbDoceHkoBjRFYtHz: 1.00000000 BTC
                            └── 151yi9xn8aAzPUGC83f2dzEGefLAqehXb4: 1.00000000 BTC
                            └── 18pU9kLmLzLmMJ6JYa1D6pKb5Apa9UXwuu: 1.00000000 BTC
                            └── 1597YCDF9GYYHnhv6XnuFKdB39iiHndufJ: 1.00000000 BTC
                            └── 1HUcXkydXwDVMVWWYRGqFLK5w53FhXinem: 1.00000000 BTC
                            └── 13nA64juTkvTrZUTxxc1i5cnEVTAZD9MLJ: 1.00000000 BTC
                            └── 1NtPFbDGNrPA1FJyrifjyvymdmFFiwGBAa: 1.00000000 BTC
                            └── 1DFocLf8E1khhsiSYUX6E1VYa1MNYmRkko: 1.00000000 BTC
                            └── 1DdXAeZLLHLiJvMK5BabY3qgtajq5NvtzW: 1.00000000 BTC
                            └── 13XUpUYMYYAnVA6y6vxPedwrqaJ5MwaF5e: 1.00000000 BTC
                            └── 17DNEeaMyZqjw6TnMGuimqeoQd74qrFQ3d: 1.00000000 BTC
                            └── 1Hq9SyZ66sKH8ZibcNTS6ozPsSdY35qEqN: 1.00000000 BTC
                            └── 1PUAbi7AF8E6chD7F2mrn71UTyiesdzUxs: 1.00000000 BTC
                            └── 1AvHKXKTUkujD85zfdxSCSm8UigCCpqTzJ: 1.00000000 BTC
                            └── 1JUnFfYdC1N5KCwg9GjfumwLZ6H7pF2QHF: 1.00000000 BTC
                            └── 1AoTy4kp9a7dP6aUJpXirdSpnRgrE1W1n1: 1.00000000 BTC
                            └── 1GM1VE2E3ezNr1T4xwFeQW99Eue9sL8e1Y: 1.00000000 BTC
                            └── 1CNZZcnjP57W9BxgBDcgxk5TcHXmnmhZSA: 1.00000000 BTC
                            └── 13XWQCV1wXExsqzL176LGZP2eeyzKhGDAs: 1.00000000 BTC
                            └── 112bZCzRbSrzMjtX9x7S6V6H54MPpzZgxH: 1.00000000 BTC
                            └── 1ASKSQ6ocjyFyM3RjqTqG2j9szKcPddG8i: 1.00000000 BTC
                            └── 18DTGBWg2CJPb13FZ16p9K27LHg9hpfgnQ: 1.00000000 BTC
                            └── 1DvicXjKgntsFNEmgS285NHggjGAe8n6K8: 1.00000000 BTC
                            └── 1797wgMXediJVZffxgJEAZwsriZEtKdvLi: 1.00000000 BTC
                            └── 13E1qhfCo7VwPnC68G8GmP4kEZdeXpESsP: 1.00000000 BTC
                            └── 1NjN7UPkSa67yuGJAyS4JoyYnaFvjU2Sr3: 1.00000000 BTC
                            └── 178Gey3ZLmkaSFQQLuwhnp4jg7XBnZF5Xd: 1.00000000 BTC
                            └── 1LdHvaMML4CyTs26eTQ9aAzmRnswQqLkHt: 1.00000000 BTC
                            └── 15WCcXLXVfs5io7TJAeFC3vmE3uaAmaV74: 1.00000000 BTC
                            └── 1PxgHJaRqpqT5TMzSvZViJDZKcHqPHDJeo: 1.00000000 BTC
                            └── 14T9baKzESj6TVP5wWG24PSxqHG5YZa6Pv: 1.00000000 BTC
                            └── 12EiBvbR4RDaBd8vqVhgFeNbPV9C1bm6UW: 1.00000000 BTC
                            └── 1FrrRCoUyxihj8SoQHG8RAfB9jkzrUTnX3: 1.00000000 BTC
                            └── 1CksXQegbUMm8H5YvKxq5mtyhuU5gb17bQ: 1.00000000 BTC
                            └── 127RMjCoWfFqWQ1Wgj5RanB9SCZNunLDRb: 1.00000000 BTC
                            └── 13tag4i5MH6cbpFuq9pZogrnbuArqkn5uD: 1.00000000 BTC
                            └── 1FnnD36mgf5gQP5atLm57TDVXkVuPxdbRP: 1.00000000 BTC
                            └── 1CF75rPnWtm3bAjyncP4EGaLK3SkbQGig3: 1.00000000 BTC
                            └── 1LmkYPUmwX4fruZpXKNeuNfvUotVwwyJ3i: 1.00000000 BTC
                            └── 169ZhbqeHXNnoaHcVoRQdZXZ6zm8XvbVXf: 1.00000000 BTC
                            └── 1Eq3p6hUv3d9z7z1VHvn4m7JLCEM1B53tK: 1.00000000 BTC
                            └── 1LKwppAmbmrCzRoUFURKSEWkmdAFFVLsFr: 1.00000000 BTC
                            └── 1FJLmGd9hk3UnxWRUHXksZCpXDT1E2CTJc: 1.00000000 BTC
                            └── 1HCu9wT3x458Jy7h5RedA8zui2LK8TEhVa: 1.00000000 BTC
                            └── 1GBrgwKX2DxL6wDPiibWkSuhXdULfLcqpd: 1.00000000 BTC
                            └── 1KUoDdJD4ebvaVTrpY7nAKNuVqr5ZxtDt8: 1.00000000 BTC
                            └── 1BBoevCZbPqnRMJSA3Jke5ZYyr8ggVy9oP: 1.00000000 BTC
                            └── 1EPcJnfeijJ7Lwbhjxww3x39zbmz8UJwFb: 1.00000000 BTC
                            └── 1BuQ67Hyd9xkjMpouT2pZXAwGZSTavYxM8: 1.00000000 BTC
                            └── 1CsuZV94Q26sccMD1vrStK6CgSnGXcxwiF: 1.00000000 BTC
                            └── 1PSEkNMyvGDVcWXyQEjouGoHBMg36KzDN7: 1.00000000 BTC
                            └── 16D29XivzNxUXMEPNLCXAMwtAmQz7Y6A4k: 1.00000000 BTC
                            └── 1G5Wj8jZvRfWmxWNAk7DiEfVsMnB76ynho: 1.00000000 BTC
                            └── 1MPaUy21dgNHdM4SE1ontvdkiiCAtfBEPN: 1.00000000 BTC
                            └── 1CP9ayvaNHzGZHKvmrUzicKW1tLYXGusoo: 1.00000000 BTC
                            └── 1LwCEaUQbvnYJLLRBjHwesZQUCSFcn8KZ5: 1.00000000 BTC
                            └── 193SeoU1uXUn2XeV6YbCZgjuquqGQNM1Yt: 1.00000000 BTC
                            └── 1GpcpfUSXF1GiTsprp733Bskp9kZJGBCvE: 1.00000000 BTC
                            └── 1H1XQis9PkwUdyMScppxzJnXwnNdvDWku7: 1.00000000 BTC
                            └── 1B3ryRmoza7mrk94xXHVsZ6Lir29rke5Xa: 1.00000000 BTC
                            └── 1Q14BHBHhT2pp8Cd666ktHC1azYhFsb82G: 1.00000000 BTC
                            └── 1KKpxAF9qySFsocXYFzbGkYqURaq5Pr7eT: 1.00000000 BTC
                            └── 1CkPV4wB5nxb7akLccpqoTxeFSJoE4XjYS: 1.00000000 BTC
                            └── 1GF5mtRzUsypQn8UYvJV8Y9cy5pStzQmkh: 1.00000000 BTC
                            └── 1CpVkoS89dPeCnh4DDxXKFTg532v6wAXpF: 1.00000000 BTC
                            └── 1FGS9UtQVHKpmvPYfp7HvKnmBzdqhM5npr: 1.00000000 BTC
                            └── 1AABpNhw7povpcWSSYjTxzH82BJg6CLyiv: 1.00000000 BTC
                            └── 1F9LSmQDyuFsdDchxjCWBTcthaFVYAVdzi: 1.00000000 BTC
                            └── 1GJFNWknw2TXfkPVYD5zjFBs5Fft3t2wUa: 1.00000000 BTC
                            └── 1GPvyGpKL8ohGbXEGB7n21DzdfpQnG6e6e: 1.00000000 BTC
                            └── 1BkFezai85uzQeaHTcPZYgKWjYmiUMHuj7: 1.00000000 BTC
                            └── 1LCxx6rQoBaXP3iHdh1gMJ8bKsEhX2tDLX: 1.00000000 BTC
                            └── 19SjKkVCqL6cMCjhDZojD8ekNF6QpJfdFw: 1.00000000 BTC
                            └── 14q34CPkftx69b7HwjUAasHjj9GKnffU6B: 1.00000000 BTC
                            └── 16wYQ9Zw1QeVAnsMsjXCQtxXMJ4Fzu8hw8: 1.00000000 BTC
                            └── 1N6oostJjNXUKDNfE7Y6nmAvHX6sEQmrDj: 1.00000000 BTC
                            └── 1BPEDMSjMmFYa1sHPeUTxyfJeHEx3aVSqz: 1.00000000 BTC
                            └── 1BDPuhgSZrh6hFJwmd1GBiWVHDxuHMVw3V: 1.00000000 BTC
                            └── 1EBaHFpnRucqpGc3sfZQXGrNU7Y1eVTuYN: 1.00000000 BTC
                            └── 16QXivCrDANeDPCa1WVXPYLJQeUBazVEeZ: 1.00000000 BTC
                            └── 178QLqfczWzGtGKJCBE8gRFMGUYdHR9s7H: 1.00000000 BTC
                            └── 18CdahbQyySQhW9F3zHFFoXDpaK3vs5tGR: 1.00000000 BTC
                            └── 17WtHjeCfDc3FXY7r8swqHcVNMNtmjo9y6: 1.00000000 BTC
                            └── 1KP16kFQr6daG4poDxQdLbjuFAfmEMcV1J: 1.00000000 BTC
                            └── 1Lbg6NampnTKzbAtpKXfQbmBeKovpc3pWw: 1.00000000 BTC
                            └── 1866Wk1cDj3t15B96oVsB1sEF5mPwfR7Pt: 1.00000000 BTC
                            └── 1HBFkbReXtkJewez3tWJaqDSs1FqLZYMyh: 1.00000000 BTC
                            └── 19Y2CAmGxKRVYfbK7JEFqkd5fQ49L7hmxQ: 1.00000000 BTC
                            └── 13sqd9mtmYvkdK5qYGDx56ESE9KjU97SZn: 1.00000000 BTC
                            └── 1C11k4sTFbM3TjF3u7RxSrVQjP3pabaEjb: 1.00000000 BTC
                            └── 18JtzcpS3w6FujHu46SFpLdzkW143f9w4F: 1.00000000 BTC
                            └── 1Ar1BoAxzRXgxfeoZummJSofZBQWVFhwAw: 1.00000000 BTC
                            └── 1Q1CRWGcMbR5vcr5haJ9td8mMEaFnFSwvq: 1.00000000 BTC
                        └── 126mEdXokYrsUSAhB6jjLr93EJFbnkc8yj: 20.00000000 BTC
                    └── 168jSV5uZp2VtMPyBm1UxdeBuRnBA7cqs7: 1.00000000 BTC
        └── 13C1ucrhTCJGmmAAHAiZSj9ag9ukhaUnmB: 1.00000000 BTC
    └── 17pbedcsbyfa4M26R7WLohK6sxYEovfw4o: 1.00000000 BTC
  └── 4912c531d4b280ef58254cc9788544c9cba5b2eabc377d3feaa66d4cfef0366c
    └── 1BmbdXZBHXttB6asm69rcjpLaUti2onJFo: 20.00000000 BTC
    └── 1Npm2PErEHHcFozWhBd5pa3smU6eyq3Z4q: 20.00000000 BTC
    └── 13vU1X1wnFWAorVKD1MdX9fsorrytrDBAT: 20.00000000 BTC
    └── 1NKg73XiUzAhsDuWTUPLxptY337Sf39yZA: 20.00000000 BTC
    └── 13vxo7WTwWVENDggpsUXYPEkBSQgvppnv5: 20.00000000 BTC
    └── 1CWDNoBFuytK4JAjYRBVtpaMREq8dsXEqF: 20.00000000 BTC
    └── 12ZQ4bnu9KCfWXAQz8YNYKPLtFE4DPwUtZ: 20.00000000 BTC
    └── 1EuttrML1xEvbtcTatxQsEcMKKQS48qsE8: 20.00000000 BTC
    └── 1H33yfS2rLmPVhegnqCxZp3uNaco2wTFqp: 20.00000000 BTC
    └── 151XvxjnBXGpjDWZXST9Kr9gRRC8DXswE8: 20.00000000 BTC
    └── 1BC8QFogYJc2to7xvtPiv8LespAYMQKFQ8: 20.00000000 BTC
    └── 12h3hpoD8ocPDZKd9xkpp3n3WGa5fL2z1e: 1.00000000 BTC
  └── 9f9c271886e860898ca3781b21e20b734fda44c2c5e712bb936da61ca56bfe91
    └── 1HLHk4kMf6YYbykWSVCxjf7z7n6VFjSjoh: 5.00000000 BTC
    └── 1CKDstPd4rTExF9mVH482G5vCm4sMhAKqi: 5.00000000 BTC
    └── 14TNWHAXZM736s1N3HEEd6m5qpK9ZXtxHm: 5.00000000 BTC
    └── 1HTTr6iUrdGCM5eWfX5vSzQwhqDSjeDttG: 5.00000000 BTC
    └── 1D1suRS5hscHpZTxPZYysSBok8stUAKbJ5: 5.00000000 BTC
    └── 18VVd75aUNj4ekXUJKFnhXMWW1qgQvXKoG: 5.00000000 BTC
    └── 1MtefNLWNN6EfAadzSSTQgAVnXbHffGzCA: 5.00000000 BTC
    └── 1Nh48GNL47L4dHiGLK297JiEY3nfymdS1W: 5.00000000 BTC
    └── 1GELDVJJDu4MYqfe9QY1HzdNkTfHkLd1zy: 5.00000000 BTC
    └── 19Abu1ykvFmAGAuRpNoW9EqbigjP7ywJMx: 5.00000000 BTC
    └── 17GoG6opnac8Usw8cVoeti3UcYRcNEtMZS: 5.00000000 BTC
    └── 1BMr1QjZeapsiHUSicY59AGrw29Y5Kw3iV: 5.00000000 BTC
    └── 16xRknaC3iFcRWbBU2GupwNHZELpiExd1E: 5.00000000 BTC
    └── 1GW4xhgKLNbsSb6s6z7xypUY2vSDsFrueR: 5.00000000 BTC
    └── 1M1c5QgfawAeQKFZiMBThDEZtCBAihFmuD: 5.00000000 BTC
    └── 1MYqkSdSwwQPsWNfui24g1d1CxDFvCqswK: 5.00000000 BTC
    └── 1J1F8KinonhhGvMqRV3MGKQcz6tHp39HRo: 5.00000000 BTC
    └── 1HRBu3ePinFFp4bVFMFGQmTsaYao91RoG6: 5.00000000 BTC
    └── 1KpCUtR5gWQ2dXJ8Hr4hjg8DUrLEF66H3J: 5.00000000 BTC
    └── 18LHAhgbtwbjzcpW4vPq42fmPkCkQqJNzD: 5.00000000 BTC
    └── 1MiHec5gxYvrqjPemMhZSWyQvv2nEtJqK7: 5.00000000 BTC
    └── 1NnbkV9yDtgWdhURL3sGNeS1wo44vat1H3: 5.00000000 BTC
    └── 1FdsQoEjt812zfuqf5cc1xU2ocJJqxD7aa: 5.00000000 BTC
    └── 1McUrFKamXvSxPa7wW2jnddyAPs3iEkGrr: 5.00000000 BTC
    └── 14Q9ePcjNsBqemnynvLTwakpEy9inzH2Ck: 5.00000000 BTC
    └── 1MAUwvegfy1jQHZXhGVehCBc2VXbPazhZw: 5.00000000 BTC
    └── 14KVqQ5xgcFmPEKeHsMzYTtecVRhum1eJZ: 5.00000000 BTC
    └── 15MfHXxWM7MKhLrhpCQaDS8KG2SipjuvsK: 5.00000000 BTC
    └── 14qq1ZPU6QWhnLkJibC4VTjCsuH6frTs4Z: 5.00000000 BTC
    └── 16biEJTFy8QtRKTqS4obpZRgvyjTXQsChZ: 5.00000000 BTC
    └── 1KrGmNY2PaWgAx6288RDLkZgst2SvGAN1f: 5.00000000 BTC
    └── 13VzAmJufAEm9beqQwZadtjoDgL5rrTtkX: 5.00000000 BTC
    └── 16FPtWG7v1qnUNHPsUdMLCBRKFnpLTmewY: 5.00000000 BTC
    └── 1LrmHbGt2mQ6NYXLLohqYW3o2ZL6gnhKYk: 5.00000000 BTC
    └── 14bMhmMegveSbbA9oUetX4QDoQrjj6JD5Q: 5.00000000 BTC
    └── 1DWK3AKKmJexn5qzX91cQzAS69Q8LJp5nD: 5.00000000 BTC
    └── 15v54jKVAq2xgwfiz7NGDnu8ku1DLTmCon: 5.00000000 BTC
    └── 195q5ezhRKmxLipwZVQ6LCNivsfetxjvYM: 5.00000000 BTC
    └── 17DmUfdAbrY4NSyhgXgpNQiK34SpNT3Zih: 5.00000000 BTC
    └── 1PHhNNoaXepcXRY6gLsKEV7uYT4JW78rhg: 5.00000000 BTC
    └── 1FcEzRJjSSTRXZAjoZ77eVRupHtVBubMhe: 5.00000000 BTC
    └── 1CrwF3aEiz8wS5QzijP2KG4eDejQUewSd7: 5.00000000 BTC
    └── 17Fggwq8rZrTofHavJJbKvqHWDqH5zBz6X: 5.00000000 BTC
    └── 1MxUAvmQ8h23JyWXeztHmWFhrDJuCDbwNw: 5.00000000 BTC
    └── 166UKfLEVa1rQ4Qs2kKFyN9qdMoqrJe2Sk: 5.00000000 BTC
    └── 1N9aNFukLpicBwXUVyKSTyQ6bzWTK8p1VV: 5.00000000 BTC
    └── 1LxDudGDkiCWc9uwAqfN6ChzYCEwzehbuw: 5.00000000 BTC
    └── 1GPR14YXvz4EmDD7GgozPQ2KvpVGC5hmYd: 5.00000000 BTC
    └── 15BH7XK5Kkbc1Aq1eiXRYAP9Zoa16BJC3M: 5.00000000 BTC
    └── 1QJzhHfPmMMAFSZdPDWU8RosH6iZgtRF8N: 5.00000000 BTC
    └── 1AJAhbR6tpTWPMrhq6jMvwxFCrV8t6qASF: 5.00000000 BTC
    └── 1EmtzwNxWr1o4wkFPirFY6meh9cFufNLo5: 5.00000000 BTC
    └── 1BusxDXMzh9iwPxbgBRVDQwK1Q2X6M6MT3: 5.00000000 BTC
    └── 1DSCyxD8VUVCJYE468UeLZ3DPS5dghgKdN: 5.00000000 BTC
    └── 1335PxG6ywwh9iMWPNZGA39REV5rThWxMi: 5.00000000 BTC
    └── 1J3g31MhuGPR6UVeqRXsdmYhpQqiSj5mn7: 5.00000000 BTC
    └── 1ELd4XLQdLbE8CuTpxM8gkw8Qxiw13TvR2: 5.00000000 BTC
    └── 1EeVJzMoi8VPGbRw6vvjYn1ZFq3PCUP1Aw: 5.00000000 BTC
    └── 14QafEdQYRgcMPiWyFKUJbiAfXWK28G1v3: 5.00000000 BTC
    └── 159f42MrZtDoXZ13S4LzphMAdSGk8CUpRE: 5.00000000 BTC
    └── 1PiWkdL7c2qNGazg42r2maMirQQykGkrKn: 5.00000000 BTC
    └── 1DDnjTq6f1vyc39CcUFUngEGW3SZfpaUXT: 5.00000000 BTC
    └── 1GY3bgZBmobfuoTH6k1xouMveN9iWDVAmR: 5.00000000 BTC
    └── 1PxRZbYriqDpbTNCFHMBQk8FvUPZYuM1Lt: 5.00000000 BTC
    └── 1QKzKDZKgL1VK159AYzLWETRb4k7XSFLsU: 5.00000000 BTC
    └── 1HcKqpjmcZv3zcVS5wocrZhdbk486qiaZo: 5.00000000 BTC
    └── 17huHD7N5bnaXQyrdnG9kp9KhpDv5FUe33: 5.00000000 BTC
    └── 19vjPgQv9bmKZXe7QbibFUzDG9eUjd4xs3: 5.00000000 BTC
    └── 1JRPbUED5uF8iy2qCMeF1tN1yfjHSwig2Y: 5.00000000 BTC
    └── 1L7omrLJEDjRv6W9zX9J4nzKWEva6CUoeZ: 5.00000000 BTC
    └── 1AhJCuPYt5x3rTXtPPo7Rs1Zxtc3w9XwrM: 5.00000000 BTC
    └── 1E84QaKsCU5azaVc5V7XbfAiqefHd4CpR1: 5.00000000 BTC
    └── 1NxrJzL9yS9ZrkdcZidbX3cR7RjpNvCfrN: 5.00000000 BTC
    └── 1NFcnBhUUbSyXMa2gouwTbaFkKMKVBHu7z: 5.00000000 BTC
    └── 1AR4QRUbCGaU5nqgXLiBny6X6soxLpUyB9: 5.00000000 BTC
    └── 1KaSFauEZtcVLUzD2EpfaD9DyCnP9n35bt: 5.00000000 BTC
    └── 1BMPdV8R2d45mTJ771bias1KSoFGwstnsV: 5.00000000 BTC
    └── 1ESK55m8YGjPpDgx8HAqY88HwvvPM26YHJ: 5.00000000 BTC
    └── 1DbEjWit3cDDtViPgFSwqLsbz74HQ1aA3C: 5.00000000 BTC
    └── 1GgroqP7yPEiWP8XVAYztJQokz1izDuaCL: 5.00000000 BTC
    └── 16DUXAPUkUpCcx9Aqbqpi7dK2Tj2XWhPxJ: 5.00000000 BTC
    └── 1ETqQ4KVYHb7LgKkyzzSNYXFSu55h7quJa: 5.00000000 BTC
    └── 18oYCez1RoY6kUpqfgHkcpAnwKm2bC9nmi: 5.00000000 BTC
    └── 161zGiQAaU5pez1wxfEqfSEBo918mQwvp8: 5.00000000 BTC
    └── 1BsRX4BrbdrWowyDgHAe4JBeMBCwC8kxvQ: 5.00000000 BTC
    └── 1E9bRc5SB5Yqpk8gkNzSLWGdjGRU9ZozHQ: 5.00000000 BTC
    └── 14GerdPidfdFvYqXM4ASqYmnx7aAvCpJPi: 5.00000000 BTC
    └── 1NPjMD3ARata6fH379kJN3Ye7bEipC31EB: 5.00000000 BTC
    └── 1EcXRpaM2nsanfDSgcwWxJatFVaD7NgaJY: 5.00000000 BTC
    └── 15zc4Qpu6odRq8UsXZzdUiFDMVBk28mBPX: 5.00000000 BTC
    └── 15qRUEqMukNJTukTyoNjKNKTa8uhqwVLgP: 5.00000000 BTC
    └── 164Tk9pUYAzQ4qe6MJpVs9FFmdtQrYHvKT: 5.00000000 BTC
    └── 12G2ftwiXhMymQTGeZM7AUHQXqpst1qFUU: 5.00000000 BTC
    └── 1Pru8siLqEc5MLWvra1aSHQ96mWuNcCymG: 5.00000000 BTC
    └── 19qK5bhFpD69QxE6Yd388v16ktWWfPK4tx: 5.00000000 BTC
    └── 1MsVRmtrfUsGy2UeuSLn24kuR8X3PD3UH5: 5.00000000 BTC
    └── 14ofEHC9nkZEMqaduq2UBUNTt99vZFCijJ: 5.00000000 BTC
    └── 19apGS9VM5C7QEBcatmLF3vPZTrFXCcGek: 5.00000000 BTC
    └── 1EgqMxDGUJa1zw2fzGV6pTwipD4XbSmKp8: 5.00000000 BTC
    └── 1GHTvv1niRWeuvmuKTPn3w9hD7MH9RRrEu: 5.00000000 BTC
    └── 1AJTxUHN4EP8JubWpR4mRS8obFpF3q6Jjh: 5.00000000 BTC
    └── 1DHbmJrq134NFoRrPtJX227pohzYGQwUzi: 5.00000000 BTC
    └── 1PPuo9o8JrgVdbjyTRnzKwxdiQwe1q1SHK: 5.00000000 BTC
    └── 1APiCU2T5XdzYoKBKut7nQBPxpdJEG1sVz: 5.00000000 BTC
    └── 1LULerHg2AZstsVZ8NnZN2bcdTBgv42nfT: 5.00000000 BTC
    └── 1DWSsvdQABH6jmBhDqb7s6cUcpMQTnqteZ: 5.00000000 BTC
    └── 15YzqERbwS5E7GE3c3QQ9GYRMjGExoWVXM: 5.00000000 BTC
    └── 1MVFuiA8FkWwtpcYAaz4W4cVu5R9rQ6Drj: 5.00000000 BTC
    └── 1Hxcfkfza1snGvJRt9hFf2KymJ6C1XNBYw: 370.00000000 BTC
    1Hxcfkfza1snGvJRt9hFf2KymJ6C1XNBYw
      └── 4f3578642172b157dc254877e958a4b22ab60f079cfa01f5973eea611e33988e
        └── 12bEBR8gBTWv3XdU628ZkRMtJQD9J3Rbbj: 20.00000000 BTC
        └── 1NjP9hx8MemA8AtRXWQxep9X92UvGsXd34: 20.00000000 BTC
        └── 16CL8QXcM5cGukxB8qihnacwbtMrdLvuy8: 20.00000000 BTC
        └── 1PtwJBf7tuZwtyd2vQmMi41SeR8EFwWyEX: 20.00000000 BTC
        └── 1ELpmfYHoDyf3AJwtFhA18DiV9M7hYFpYv: 20.00000000 BTC
        └── 1LKsQSQQsGT4Fhp5PbvmCugWd9heipypKz: 20.00000000 BTC
        └── 13RD6VyyhW5cEmzSBEWVW31ytbki35XiWR: 20.00000000 BTC
        └── 1Mn2JvH77GqM8oSHkwWEJtrgWBnvuFH25g: 20.00000000 BTC
        └── 1HiZAiRELMzsrJsb39LU4WYCSAr1Cnt3VX: 130.00000000 BTC
        1HiZAiRELMzsrJsb39LU4WYCSAr1Cnt3VX
          └── a0fe308fd3c24e098a74fee2cf9d7f6c044e16b2c820e40d913d675c7011f8e6
            └── 15FNFfgkNpesLg4MrTMmAXajgASYSof19j: 1.00000000 BTC
            └── 1K6hw5mgcitK6z32YWkBMZhT2fwcchW2mo: 1.00000000 BTC
            └── 1HJjGhkt8zHv72Vx6dYfHkxXbHEYjpNbss: 1.00000000 BTC
            └── 185PPKrqepcCAkCS4RoHKKu9H75Dxg7yfT: 1.00000000 BTC
            └── 1DcB6ftZv3y9U62wU8nqSRcxjm1My8uWYP: 1.00000000 BTC
            └── 1CwJN4NiZHjH6Yc6GPx7BYAPvSC56BM2SQ: 1.00000000 BTC
            └── 1AA3krdbG7C4bCPvwnQr2gv464GwTfRctE: 1.00000000 BTC
            └── 15qDZ559NZzXVxfWrrpkZG8wJdPvrybdHM: 1.00000000 BTC
            └── 15hoq231zKqZPHwkeJiJZW4AiqdGKMustQ: 1.00000000 BTC
            └── 1B9zMSA5rVqEvdrd2WZsaJcNKuVeQCQ6Ww: 1.00000000 BTC
            └── 1BeuQB4X9FzW3TuVEkRVdhwivYi3Y2QW6K: 1.00000000 BTC
            └── 1DfoA5v1xJRSsRgNMxESjASotFJjJYGX1p: 1.00000000 BTC
            └── 1QBeEYp1NDqx9fxkmMemV488L6LUgWgeGM: 1.00000000 BTC
            └── 13v9mtruBP9UVKff7YLy4fkzccbpVWSBBk: 1.00000000 BTC
            └── 1DBHBdVFMgCiGNkmxSETDNLXA1QrvRi7sF: 1.00000000 BTC
            └── 1AcRvhsyJF9BCxDZWefzMhA4v5Hu2EvT39: 1.00000000 BTC
            └── 1AXux96R5gVKcVsJxQuqSMBwwK7ezam7sj: 1.00000000 BTC
            └── 1GfEoaKGdQs15WSTLdRbt7VZ6rPh1kzfr5: 1.00000000 BTC
            └── 1Hos9Gj7XeCUEA2DmGrN1AiQdkHDa3nUfc: 1.00000000 BTC
            └── 1Er7D9icjbngH2yD5BetJDbUnv9EyasYYo: 1.00000000 BTC
            └── 1158YqLj7MgCA6iiX9x7P242HC8EU3FocZ: 1.00000000 BTC
            └── 15EzqxdK7ZHKen2XHHeHtXtNgexJGh2McC: 1.00000000 BTC
            └── 12pachdY3rBG4C4JZFRF7kUNBfgvKjXZ1h: 1.00000000 BTC
            └── 1CrdVA6KUDLiSUwQC7SFeqBh3mEd4wdp2L: 1.00000000 BTC
            └── 16EEQvBbsTygf1dd98ThKwrQA3w88SPWMf: 1.00000000 BTC
            └── 1Heoiesb13FY6YzeCkr6y9iXvvbsNZrAcK: 1.00000000 BTC
            └── 1C32JqrhwSBP6xH12wjRGcowVq6X452WBt: 1.00000000 BTC
            └── 16Z5J6R83z7GA25oJPvohoXFxaLftC1RKs: 1.00000000 BTC
            └── 1BE5YG8YCzADhVnFMTEFV9YqFbd2BaMVar: 30.00000000 BTC
            1BE5YG8YCzADhVnFMTEFV9YqFbd2BaMVar
              └── 69a39eb32d93639537e6f3321fd60e608225272699a09dad5f511f38139bd6fb*
            └── 1J3PJmK2sCS7p511frpGGcZv1YhGLo9RLF: 1.00000000 BTC
            └── 14JjMSyD77HU5WPYWw9LhpbcuGgrWrextP: 1.00000000 BTC
            └── 19Ngvmmb7Risp13ScXjxAr7QRQdA9KUp2z: 1.00000000 BTC
            └── 1D9zreP3D9yiaMex43GtdwB5EwwcC9yyfA: 1.00000000 BTC
            └── 1P6xoXyhR8rscftJ1PB5S7vuK9QnimMkqM: 1.00000000 BTC
            └── 1GsrBB8xkUPDp5mxvPevacyS85qydhqMNk: 1.00000000 BTC
            └── 1Ho7ngBHHMpBfQJ5NBh6E5NmrYT1BDsQcd: 1.00000000 BTC
            └── 1EHzU83pTbNr39TkTJSRvygRmsD4c8mkKz: 1.00000000 BTC
            └── 1N62yfzFu965DGY14SqwJ2RSJ2sQMXMdQ5: 1.00000000 BTC
            └── 1AjHfYqXL7erox3JDUP6k9qeQzEU4yEwBc: 1.00000000 BTC
            └── 19MLXMqrpTsQddF42nSZx597R37mAS6T6J: 1.00000000 BTC
            └── 1GiYBrg5CcbFtDz9W8yXTdsW8EXD6kDmQa: 1.00000000 BTC
            └── 17rvvk8dZzZuCtLnRLrGm15fD6DZ89sPF4: 1.00000000 BTC
            └── 14wKRJ7bqWixbyeXxDWurhckQyRyfPUGLi: 1.00000000 BTC
            └── 1B1YN8xUnw7mNJcvSAAJTLKbCncNmyxSog: 1.00000000 BTC
            └── 1Kp4Joh9mUNKRubhzN5ASmwkbY13zgPZ3L: 1.00000000 BTC
            └── 1LeNs9d4KAUe1Z4EEiGJKjew2LKXV2sVi5: 1.00000000 BTC
            └── 1JuS7vPMbhuDLGzHs6JnnzctYnJFFUUXiD: 1.00000000 BTC
            └── 17b4DxLvaRjpbqq9u9y2FJ2yms456FgHqp: 1.00000000 BTC
            └── 13njS9poyn1wC6j94PwG3bfA3yXdktbr8F: 1.00000000 BTC
            └── 1FvbvfW6TypdYt8xGiKWdBf88SX6AD6jtz: 1.00000000 BTC
            └── 1MXwftiUBGVTG35H2JKnCmXCq6ukbjaPgt: 1.00000000 BTC
            └── 18wyH3qL3Cdh4wsTLK9frqHdngKP5iNmbV: 1.00000000 BTC
            └── 1NaYur81Mf7Ukse24TYpLR8LWRh6HVd1cy: 1.00000000 BTC
            └── 1L3qiJAux7hFr4fRgH2Shm5konUuhE2UHv: 1.00000000 BTC
            └── 1DWwc9q4CiFXywpMCCGV2sQpXqiZJX28av: 1.00000000 BTC
            └── 1EM1hVHuifpFgXWcTGnYVUEjnwZYBjxif: 1.00000000 BTC
            └── 1KMfzEDPXfYycQqFKMjfRsaxhiUYefLsi5: 1.00000000 BTC
            └── 1LBdpcLwB62Fx6u3Yj2jhY1KE5Ey7GTWSn: 1.00000000 BTC
            └── 1LZbvvivPMrcYh7DbGymnDkm4u7jDJmocW: 1.00000000 BTC
            └── 1BXo5jRKQYNqmoCJigYdUJrJwsNE65LiB9: 1.00000000 BTC
            └── 15SpqQ1qogye8T1VThr7BZZSBVr2m2SKqB: 1.00000000 BTC
            └── 1MGtWvfjnvNZDAwLpqTT7gKNZyFRp76nHK: 1.00000000 BTC
            └── 1413guUYdoFNcw5eWguHsyBXhmzCyhxirQ: 1.00000000 BTC
            └── 13zkoHWXm99YHVnWWGcusJx6XeCMAAJzzJ: 1.00000000 BTC
            └── 1GYzrjGJD5CkL1B5d5fjmg49jrGYiYC9RZ: 1.00000000 BTC
            └── 16wS59GU2wVVceP9iQLLyAoqeRYsGTETFN: 1.00000000 BTC
            └── 17vGGoMQzPgdfEr994tPzKXFP2deedwsks: 1.00000000 BTC
            └── 19J555qVghj3MiHLHWurUz7f7aDnbfE6cB: 1.00000000 BTC
            └── 1EDchzbPuNK66Vx8SpQfXeZJDUw9oC1gy8: 1.00000000 BTC
            └── 16s4HAUaah6nSynu76UwrHDHQv9f1boEWu: 1.00000000 BTC
            └── 17we51GMWBEgRhoXJVydDSHa91SFTgdkuG: 1.00000000 BTC
            └── 1D4CwcfWCeJbNo3icbW2asLs4fy2Dp2QSA: 1.00000000 BTC
            └── 1NcnwLzRXKwYezBqvc1RUYFo7yTxC1tQbH: 1.00000000 BTC
            └── 1HG3n6wg3kKUsfeWpAJgE7LytB4T9kLca4: 1.00000000 BTC
            └── 16RN7RXwjsw5wnLtYfgegWaa9RxNXvcVnr: 1.00000000 BTC
            └── 18oE9RbrBmAQemY5J2fMPM7Xyc5MMMAEMo: 1.00000000 BTC
            └── 1A7urLN9Cch8dzBEa45Q71ZL9LUYmq8sPn: 1.00000000 BTC
            └── 1NXar17mxbgzcEmNNiZkKzdD84QVtQk4mT: 1.00000000 BTC
            └── 19QdT6XYsFwMGvm3cZ4neLF7CRYteVr3UX: 1.00000000 BTC
            └── 1F8WNHomqPJHFprvK4eUq2NV1cn3rLAEi4: 1.00000000 BTC
            └── 1FVWAnugRgGgBLsWFzQN7uutnGST2Wd7sQ: 1.00000000 BTC
            └── 1C9DRfENUTsoxszVXbxbtxgxHLbptwCpMQ: 1.00000000 BTC
            └── 1EL6tVPor2Rp2bAhEPGmwgteLBVvkQSWLh: 1.00000000 BTC
            └── 1JkKDN5z9JTUeXCkjj5qH7pfzvbufWFdek: 1.00000000 BTC
            └── 1FsiuiKXBc7vN5qLDVUS3VS3tzxjLxmzw3: 1.00000000 BTC
            └── 1CK1aRBs6g9gFu8FH21qNUh8omVzCH2ggj: 1.00000000 BTC
            └── 1MidwTnWBy1djGrhpgPYyvoDbowgTRF5m4: 1.00000000 BTC
            └── 1Mc5Vc1GK8D8ZcEYM5zzxxeAGgd2NmM5Ce: 1.00000000 BTC
            └── 12jGnrvKts9Hn568YmCyMZ7XiQoJZ1mYbC: 1.00000000 BTC
            └── 1NeaH21nu4uTECbgMB5jipZBSd3YGiT8i3: 1.00000000 BTC
            └── 1J1gu5aATcmFEYBbSij27UjQfDxwguxR8S: 1.00000000 BTC
            └── 1Jp7ih3D7nFKfGFMbUd7JQuRsr5Zm3yijy: 1.00000000 BTC
            └── 1BdXhbp2txKH7XJKcpaYFbb9Pk5SjVdSoD: 1.00000000 BTC
            └── 15PTEYHfm4ZvcnHXps1PH7iapujv5WjjFZ: 1.00000000 BTC
            └── 1HetKyeThQAVtFfqj2bTs5AgRHt32HYzVN: 1.00000000 BTC
            └── 1L7pUvxVYskFx2Tyuo1smExNVyxGG9csTS: 1.00000000 BTC
            └── 11iaPD87dD2KdJPBm7fL4u9JUx48vR9Mg: 1.00000000 BTC
            └── 16o37mKF3wzDca5rWUZrNJ9BAz4B3RbZZs: 1.00000000 BTC
            └── 1F9tP9cpYxsbJH5B15Emrzzsv1mGRHBDeb: 1.00000000 BTC
            └── 15TQmUBG3PVPtHZPuxCwmaE4bTNfKxfQAv: 1.00000000 BTC
            └── 1JksfLRNS9qxbwjCFWYoQko1q2WL7pi3iL: 1.00000000 BTC
        └── 1AkSGB4EZAmbEyQSXN99enreEseR94km2c: 20.00000000 BTC
        └── 161kUKZdFb3mcrCB3ZX43ZDDACiWvDfwc8: 20.00000000 BTC
        └── 1Ft7Ke7T3nDQp7Uq69oLt8FRaRFnGU7erw: 20.00000000 BTC
        └── 1K5LyFjFfYqCnkF6AoFmSWCNMNLMfywte: 20.00000000 BTC
    └── 132LFefLrecHn6epx5DN6AnLBbvyjnyNJH: 5.00000000 BTC
    └── 1CmTxoFiB7iDR5gQDC7MXkx6Byc6fD2mWd: 5.00000000 BTC
    └── 14gdDrghpN9uPV9vhRs2R2hm555uUtb69P: 5.00000000 BTC
    └── 1LkgUDEvmyAVw2RrXyE5vw6eLyXah5An4Z: 5.00000000 BTC
    └── 1D7dybK4p6gzda479AoTHB8fBr4WUHUHWj: 5.00000000 BTC
    └── 1BGXKZ4aiPXaWLid4XhPVPUt4LJxk1LVNd: 5.00000000 BTC
    └── 1BCmwDD4D881cJzios91RYcAvitFCNa2X8: 5.00000000 BTC
    └── 1BgHGM8ESemiLuE4LgBAborbjCrfH8h6Y8: 5.00000000 BTC
    └── 1FAkm7SArhSmnfwJeaYvsG28RZy6aivzan: 5.00000000 BTC
    └── 1JtxsmnnqsDzgbtWZwiZNPToM99Pjq6qxi: 5.00000000 BTC
    └── 13cdmkTeYqmtwG42Jc19jbxa66eEnJX53T: 5.00000000 BTC
    └── 1Lmm1yiDu2VnNQZmZKSvMXCBXez3ZqKkZ7: 5.00000000 BTC
    └── 176Lebg6YzTVDdCTSPhvonNFygmDDBHVW2: 5.00000000 BTC
    └── 1AbQmyCXbUQYx9SG2zPpq1RGaG1abSLbno: 5.00000000 BTC
    └── 1HkkDxdU7JxqtUhgBxu5sKWq5KkZLpJLbh: 5.00000000 BTC
    └── 1JoPTAhoCaG9ewbW7xs8eiffwQdFVXi7np: 5.00000000 BTC
    └── 1CnDeCLcgXMesjEkmRWcJHnyNrqUXCA55C: 5.00000000 BTC
    └── 1DmtcCvmkVqYj116kXQVd3zWKGsi587znt: 5.00000000 BTC
  └── 661c4f738aab05dc57030814dec5b581793967c5a21b845c4ed9ef3dbf3030a3
    └── 16DK8VwwHdGZMUTnokTMRXrxuj1y2KsVF6: 1.00000000 BTC
    └── 1NpcLuQvux8AaerkHtGWNF9iaKtYcUpqBY: 1.00000000 BTC
    └── 1Y8UyLdFqZW5wtee3t2CjSzcqqGFAneSP: 1.00000000 BTC
    └── 1FChmZZzrzs1LtqaLtt6hEV1EpGAdb2UGq: 1.00000000 BTC
    └── 17jzzKVzEuAR3dWVAhXhwtgd9NPaVzanpt: 1.00000000 BTC
    └── 1JVLbdhPBWSf7oqwqZZzRupBSaTCLjLDGK: 1.00000000 BTC
    └── 1CoUDiVJCwgQ9gtDKCdTNsNnLyT6iS9Ghy: 1.00000000 BTC
    └── 1F9iRC3AcDtefmmUqbXLa8DtXSpkwJVQTy: 1.00000000 BTC
    └── 1PvWGyaaeSwVvDGRY1qK2jcx4d1GwnztPv: 1.00000000 BTC
    └── 1Pj1SVcCr2CMLf7g8fvEVXZRY16i79T8jp: 1.00000000 BTC
    └── 1C18WZGx76zEqVKmehsShX4FfLTnD4Dc1H: 1.00000000 BTC
    └── 1EYrv7mzmo6ujRE2TfGyo6QYNoHrHnVdSy: 1.00000000 BTC
    └── 18mA6ZExhU57JNYi8KxMCuwxGCE2BLHU8X: 1.00000000 BTC
    └── 1D4tKZoTbxXn2Mps2Ei9GEzS8aB2mY4rbJ: 1.00000000 BTC
    └── 1JbbSWppx9JVWrecgZ6oe5eiAN6x3Pafwg: 1.00000000 BTC
    └── 17yPZ6xQP1PZsbwU4iKydnBLXVLtwDcfVK: 1.00000000 BTC
    └── 1J1qgUfkp2YNynq4gEG219wUsqEtrxBw44: 1.00000000 BTC
    └── 1BRskJdCLB2iyTZaZKEUzMm11E4xfdYRVU: 1.00000000 BTC
    └── 19yvQuKYc4tYfa9rjmVrTob3S8tsG2oz3H: 1.00000000 BTC
    └── 1NLyh94bdEahVFXuu4pZP6bU5KiM4U39jX: 1.00000000 BTC
    └── 1DsJwZuTJSCqg5uroYKwQmAccfXWAmJeY9: 1.00000000 BTC
    └── 13YHMksJxQV43YXB98cZ29Q3X1vyeWW5m6: 1.00000000 BTC
    └── 17jenHkzeLuPZJeCHr5Gqh1CX6Euv3DAfj: 1.00000000 BTC
    └── 19FUG7YadWbfBcTu2oqVA9tmcaNZDvsnGd: 1.00000000 BTC
    └── 1BG5Hv7p3a7bJCNzuyp4XYGeQZLm1q938v: 1.00000000 BTC
    └── 1AwzNVir6b5T2EdMhp8FvVMkz8ZR1im6wQ: 1.00000000 BTC
    └── 1DP74UTsr9JcXADbXe9amKYwUa2dRfFJTC: 1.00000000 BTC
    └── 15GpwyKMe9nvRKdjTCYCUUMdKvnRYTJN2N: 1.00000000 BTC
    └── 17Vv6bzjnPsaFfZHc3EggJYG1JwwtZ8gob: 1.00000000 BTC
    └── 1LLaYeXYLZivP33zpQ1V7zVCe1ZRRoL2FD: 1.00000000 BTC
    └── 1FZBb4drE4rZt2Y19h6ae7amhZUtbrFqRH: 1.00000000 BTC
    └── 19qDHTrQcrGK6zoJJmbZcTX2NR5eDsptLN: 1.00000000 BTC
    └── 1HCpkGwJ49sZ61ZgxkPE9X9UwBR7GYPQsg: 1.00000000 BTC
    └── 1P5gyVJ3qLLQrPLfCPDTxrNrVvjY6Gm1wS: 1.00000000 BTC
    └── 1JJQWJar9ZbhgiAoLsJf8Rd9aJeLoqaq7X: 1.00000000 BTC
    └── 1MAuMnSDZRM2tjQcVYNUnX2nz511nNFMB4: 1.00000000 BTC
    └── 1N3zwDKnMFWzcPLSxZBvd4BuBP9uWiFF3H: 1.00000000 BTC
    └── 1CTMQj9J6yUF9VNBs4uUddkfdhsANVAj6D: 1.00000000 BTC
    └── 19rbyy4VTwEu3emYj6Ghmeiisny9SHyvPi: 1.00000000 BTC
    └── 17dhFngG6TyJ2rM8jnRoVXruDbHStThdkY: 1.00000000 BTC
    └── 19udbZ7337QmVoMc2qntbvTB3CcuBQtfaK: 1.00000000 BTC
    └── 16MCFX9VkZwNcJLH5uKztyboeyJbrnj3FJ: 1.00000000 BTC
    └── 1GKAt9NUnPZNpNw8UiYY4bfAdUrH1GYZiJ: 1.00000000 BTC
    └── 1CeEGU9YHrF5xLdoSbWHdNDd5Lvj5g2fy5: 1.00000000 BTC
    └── 1EyAdyv6zxmEhnASMGwwkKxfKmXCZFWR6P: 1.00000000 BTC
    └── 176igCu3zN3LgSMJc3K29hiGNs4QZxt3GK: 1.00000000 BTC
    └── 112jfSyLN5NxkSKrBcVfe3xKXeXGEAEXFF: 1.00000000 BTC
    └── 1PxNAcHf5vWUG4PMMYJu7WBpFDq2bmKeER: 1.00000000 BTC
    └── 1BZRjosYJTJGXkh4pb4dUcf3HUgWiHFpiv: 1.00000000 BTC
    └── 1QEk8SCapWttMxSjes5gWm3MtL3MSZcSAN: 1.00000000 BTC
    └── 1NWLRh3gffC7vJFvg61ivvZnnn1fhofNsX: 1.00000000 BTC
    └── 12pyULyrLXBirrCBLMRuaenTZ4Rz9VrLAZ: 1.00000000 BTC
    └── 1NLrPLdMXV765WYPHBZ6it63MS6SkMWQcM: 1.00000000 BTC
    └── 1ARkb11x5qj32dPnWaQhf9sacyLy6GKgS8: 1.00000000 BTC
    └── 17jsMDRqd6WkyxUvmcFvKihLFT7yC9hbuY: 1.00000000 BTC
    └── 18bVAtF7VsW81HSMJqDGv52A1oouVTrx3H: 1.00000000 BTC
    └── 13afbDdkBpMD5vNwzc6SgLdtDrUiYUCSCM: 1.00000000 BTC
    └── 19exe96qyvGoSYdfr4dFdWGQTTjaTs9N8v: 1.00000000 BTC
    └── 1ELv6ooKax7n68xRKvu4tAo1aAa4YUzC6r: 1.00000000 BTC
    └── 1KrxCsjaJzGT9dZ49JnVbQ6M8vnaRgJnPz: 1.00000000 BTC
    └── 1G3sQjsQJ7P736JYX9iWTM2cqsKTv1DLnG: 1.00000000 BTC
    └── 1BKtPB3fuNs9fDBitruAm1tY5m1mA3U7Vo: 1.00000000 BTC
    └── 19dw5aWMVf2W2WVQ8L1g1r55LATQdTVuax: 1.00000000 BTC
    └── 1NmhZCNsVXeAUxQoLhcMxE1hoqJF4SCsKi: 1.00000000 BTC
    └── 1MEuqpmMQVW7Yxj84zQRN8Ths2KiTH9yQd: 1.00000000 BTC
    └── 187N4s3fbgRGSYjimj4HcDLgpQwhyvLR3Y: 1.00000000 BTC
    └── 1FqsfHyERuJu9Mn7TtYvkKhvkrLurfxYTC: 1.00000000 BTC
    └── 1LvtbthdoQkr3cvuXxthwPMLfT8pKsf7a9: 1.00000000 BTC
    └── 1Aogs4DyjfQVQCNBTGQXD8thbAErinSELr: 1.00000000 BTC
    └── 1Bbpr19LQYtVZ9biPGkA8wTPsgcz9cDiA7: 1.00000000 BTC
    └── 1srLft5Bad6trVTQvKprgYW2oCMygoc1z: 1.00000000 BTC
    └── 19xednLgLR4otQbTexgmpiAf9pPeR3pj68: 1.00000000 BTC
    └── 126y22B1acn2SKQZqDPjTxdZALSyiUQynT: 1.00000000 BTC
    └── 16uQLXnCwmCzQagz2CN8pNp9kzddibiW1X: 1.00000000 BTC
    └── 1GHcrzTtRdxqhA9vsN17GxSoTD6fNez223: 1.00000000 BTC
    └── 1K9jZMBuzM2moHACTnm92SywgYh6Dr6gSv: 1.00000000 BTC
    └── 15XbNHudosM26kUF2Tm8RfgHQaUSzRdDeQ: 1.00000000 BTC
    └── 1BggezMH5NH9nUg72osJCuDcBgT3xGaVgT: 1.00000000 BTC
    └── 1MkN8p21AszJVmWyqF2HhvP5mRMxRJPRxC: 1.00000000 BTC
    └── 17v9h1iYKSxggnnwfVJhMVCeVaecEycrZ3: 1.00000000 BTC
    └── 14afdXd5YfqxiyzQeN4sgMw7dVUw9P3Tpn: 1.00000000 BTC
    └── 1BV8Jri21edjN7UmHiKpqqUwTkH5YzcURK: 1.00000000 BTC
    └── 18c8WMw8nsodSVemLk38HrrzugUNybJCva: 1.00000000 BTC
    └── 1FaTPo93jPArCXmteAt6igykVeXKsmy6Xn: 1.00000000 BTC
    └── 12hbAnastn7CTRR2bKys9mtzp56eFmv6JR: 204.00000000 BTC
    12hbAnastn7CTRR2bKys9mtzp56eFmv6JR
      └── c262797f2d7c790db63a4e5ce4b4bb2a511de2c0e1516dd6d7b6b6f0c749f5ea
        └── 1AJvoQz47csmbUZGepV8g7BzniDKsuW2Xq: 20.00000000 BTC
        └── 1KMoQJkzURCwLr3FGDRNegoBXXnw3s71rT: 20.00000000 BTC
        └── 19jSVkc3CimckabiNtzmS3nmST628UVs1Y: 20.00000000 BTC
        └── 13oAMWVYYiyPV6cAixESxntsSETncM59Ld: 20.00000000 BTC
        └── 17tScceuAxJN7gaRN9LLkrawfQody6sgBX: 20.00000000 BTC
        └── 1PmC2f6sJv3xPbgZym8Tp9y1sTtiPCyGLp: 20.00000000 BTC
        └── 13xbLwBrHHGwaUqtKqVm1b3qR6pqBsSmzX: 20.00000000 BTC
        └── 1Pr18hVk4UZhipzFhoRhbETCAsdggiGgyd: 20.00000000 BTC
        └── 13DQkdcd2pU6MhcbioPYagDLRtzy5rWWhW: 20.00000000 BTC
        └── 14Pb6g4jxmbFKHTF4yb78tTjdjDYck2pAM: 13.96000000 BTC
        14Pb6g4jxmbFKHTF4yb78tTjdjDYck2pAM
          └── 69a39eb32d93639537e6f3321fd60e608225272699a09dad5f511f38139bd6fb*
        └── 12CzE7JZPvS5Vukvo9aabKTBChtggWSVXw: 20.00000000 BTC
        └── 1JCiTRbkgniVbqbeWHtRPg1H3SCCVXrd9X: 20.00000000 BTC
        └── 17SwYcujbxiwmL6GvGD2xamzCppodhuFBS: 20.00000000 BTC
        └── 12QSpHcM8ZWVaKkRvL1Jc1h7zoQoWL2TTp: 20.00000000 BTC
        └── 1J1GvAXaNN5EKatKPx86qq4YngbA9Ya55T: 20.00000000 BTC
        └── 19kCNtz6Nq6rcG19uEpmBRcMwEBTScDicR: 20.00000000 BTC
        └── 1C6Ne9ACY3zFwXBXymYx4mja1oPxKGgbDn: 20.00000000 BTC
        └── 1LX2391AmPrG4tWQTpAZEsRfKu8NLputwZ: 20.00000000 BTC
        └── 1HJ3Px3YFTPZdJge7rRoXdZXRbKPtT8aZC: 20.00000000 BTC
        └── 1242jTyzbB5Z9DnQvttWz9KfJoYyeopaz4: 20.00000000 BTC
        └── 14WQKkQFwzxVMHYx2M2Q3VZRb516SS4dzp: 20.00000000 BTC
        └── 1JBrjjq8dc6b1hveUTLdYWWfiHsfjpsxSE: 20.00000000 BTC
        └── 1CnRCtjajUp6JoB6cYyFjumYv8UgaLiGG2: 20.00000000 BTC
        └── 1MgATq9ZcFM4YtQ7nR1KmPHefoEEkrLdgK: 20.00000000 BTC
        └── 168F8dckWDrBtV6NA3YynMJHz5HLpcFhEt: 20.00000000 BTC
        └── 16hdkmqWPYPgRTFhsaPgRRyiK62jy4H4Uw: 20.00000000 BTC
        └── 1JsgJ5CypEySGRYrkd4oJJCjwmn1faPsyh: 20.00000000 BTC
        └── 14Z5PVYeEEJPfaWNiez47Z9e5reB5xATnS: 20.00000000 BTC
        └── 1A2iVM9HAEEgz1kJ1885AYgBhR27zigcCn: 20.00000000 BTC
        └── 1Hf2xkDqMdghVM8YsfzfHoNnvoHVfqyCnC: 20.00000000 BTC
        └── 18dCiZ44qicPMmi1eoygpHqCPUWQfxQwsG: 20.00000000 BTC
        └── 1EzPchRJwD3dxb2grqufpxMLDSiboLjfCy: 20.00000000 BTC
        └── 13jHwvmnXGPqwP39ue5N7fnDcL8moSBnbA: 20.00000000 BTC
        └── 1Lf4oF4LzrNUsvSoBnnQjLoua5LteK3QiK: 20.00000000 BTC
        └── 1BZHicKhKtM32jJsL6dUi16Jgmhke2z22T: 20.00000000 BTC
        └── 19DtvmRWU7MuwyH4aJwMeNr8bjVmbfcoZR: 20.00000000 BTC
        └── 1FoicoWyw7d9smxWEuVUNXwuUmEENfsXr6: 20.00000000 BTC
        └── 1Daz36oMvzTfaKKHynzs6tFDGbK2vecqp9: 20.00000000 BTC
        └── 1h6tSi7H1wgCP3ogBKPxESwejaxjBG7Tq: 20.00000000 BTC
        └── 1NKAhZhfXqzWkoL83Rfvcci7Nqzwi9GEQc: 20.00000000 BTC
        └── 1K9QXxYkvFiGkh7xmFsCvoTf9FizVzwBwJ: 20.00000000 BTC
        └── 1FpcmNVxnVHdXahWBtndCtH3jBAmVDGoAT: 20.00000000 BTC
        └── 1FsFioeNXT1iTNxnCFyRS598jhbwsUEQou: 20.00000000 BTC
        └── 16C1JmEFf6Bc69B9SujmgLrUaUJXqnRH6v: 20.00000000 BTC
        └── 18DMyfRuhyezH3twKC6BzYUdL248NpFUp7: 20.00000000 BTC
        └── 19fRGerVg965RVxqYD86FcWRVW5xuCnWte: 20.00000000 BTC
        └── 1D5fJbaztBXYMzDfgPfPCKRaZ22Fm5T8oW: 20.00000000 BTC
        └── 167dnNtjeZMKeyJWHZWHUKd5R2Mkfm8hsc: 20.00000000 BTC
        └── 1AHgGcw8LjzhsRKtKxjvyRSUSAnkcAEMjX: 20.00000000 BTC
        └── 1EFMZDCXBMR7qbEh1rWGPBhe9TyPbcEWa6: 20.00000000 BTC
        └── 18vN7swMVLgfcBbexWNiYZCGthFBp4vaqe: 20.00000000 BTC
        └── 15ZFyCSTpfVNNSkuTsPaPkVBteJLovwFQT: 20.00000000 BTC
    └── 1AAqdgbF1dEdA5Lu9gKxjn3zNmn9wNFJzL: 1.00000000 BTC
    └── 1H7iBebgYYczw8h3aiSKx4VpCMviySkdvS: 1.00000000 BTC
    └── 19vAZ8ZkHaSHF2rYtc5aHb52w7CLXpcQoQ: 1.00000000 BTC
    └── 1FMzsCQJSBxJFH6EnbWKhATB1zPE59dpQ6: 1.00000000 BTC
    └── 1LtUFKrqx1JpTKYM1VfzYsjQujxunNtZwV: 1.00000000 BTC
    └── 1EsU1FkWjjAYaGiqz1iRC3KwqmbDXb5mPb: 1.00000000 BTC
    └── 124Ai4MQxY1rg4dLq3zK56gZkaweBCqFHn: 1.00000000 BTC
    └── 1Ga73i2kpWScJgA7gwyx9YZHMwrrPCVKVx: 1.00000000 BTC
    └── 14Ya2HmVEUgwo87c6bxjqwMktiXf7KiLsD: 1.00000000 BTC
    └── 1FgmmkHRVs6BjuPKMZFDccHNN4LsoYMTsG: 1.00000000 BTC
    └── 1CgKDtMjqov8UWL772ePsrhMxQKynomS3p: 1.00000000 BTC
    └── 1PjdhGjwMrdBG66qoWP5FArCt5om6BaLsB: 1.00000000 BTC
  └── e912b3a38015b72a38c43e158f78940818908164f988c81bc98f94d7e106fc76
    └── 17Dc5TeRe2hSTxgFgWX12NScoNg8rAG7dU: 1.00000000 BTC
    └── 1FWmKyRmBqruNgibDujZufkspeHAN2wQZg: 1.00000000 BTC
    └── 16AzRekFURWciL82Fkn5jYYsLMhxykD5NN: 1.00000000 BTC
    └── 1oUvsfzGBQtxCsDstDzqxdR8Wos8QBoM4: 1.00000000 BTC
    └── 1J8CMme5cmKxuhyeUtJ99wB77AUKRXQsDo: 1.00000000 BTC
    └── 17GgKqfUp84127RPNpYg77KoEgUkzdoBtr: 1.00000000 BTC
    └── 1LehBy5PUWWMurx2nALzb4R1VZqvh2m3XX: 1.00000000 BTC
    └── 197G8pnCTGrJkd5dZdMmkVicspd8z2W14y: 1.00000000 BTC
    └── 1CYG2bs6xVXwRasqwEjL8yTB5Bcw1BzpDb: 1.00000000 BTC
    └── 1Cd9i8yboDhPC6Kv83S5ULcYLYmqU7csed: 1.00000000 BTC
    └── 15hAG5TGS4XQiPsLfsnfnJ76xwpyeKuCk8: 1.00000000 BTC
    └── 15zSucU9CU95FK9rjfqNsgSDg5JMWp4xrU: 1.00000000 BTC
    └── 1EegdruSUYz7k61iR4EGWnRXMYNaYVFmBY: 1.00000000 BTC
    └── 1A3WqrfV9Zp2YdyBMPufrtiaybbG9mg7ye: 1.00000000 BTC
    └── 13HEpi6G2te1jx3y3kSRyEzHNhRDAoPR2x: 1.00000000 BTC
    └── 1ARTz8V6naLH7GatrU2Ac6VgZo9Qf5bwDL: 1.00000000 BTC
    └── 1K8ajkCqUZVGNeoynQYB3jDGCvEA2joRBp: 1.00000000 BTC
    └── 1DYZ4LurQ4vrsUedqQ7Ea9As8scc9TA5PA: 1.00000000 BTC
    └── 19LoS4xSAHkViS1B6eRgAKv1g7RW8TbWTK: 1.00000000 BTC
    └── 1JspJikNPyUgiBemKs7xdVAKPcWQA8LNEn: 1.00000000 BTC
    └── 1KgLQsoLwYHZTLhWpQFzDdFY2yJ657KyGY: 1.00000000 BTC
    └── 1LZbdubnNRswUCTg7HXzXB6yj8xa7dg2md: 1.00000000 BTC
    └── 16JGAY49MAf6QYkHAxhZK3v3joe7XpTTQU: 1.00000000 BTC
    └── 16nLqbMTstts8gupT9PfQkza3qRKRGgaPV: 1.00000000 BTC
    └── 19Mrf83EWGetcEmnqmrpNaS63tDfWrbRkT: 1.00000000 BTC
    └── 1BAsPCPXpbo7VMMxrbRGt9KP4qTgL7sQqo: 1.00000000 BTC
    └── 1Du4p2qX9Qjh7LyjKAE5dmBFs5STY7wofb: 1.00000000 BTC
    └── 1N34Ab4nViAuLrZutkX1agJmPmMrDpRC8u: 1.00000000 BTC
    └── 1GiyuBa2AdeRqfJ8fZyN59J98F3wr6sAp3: 1.00000000 BTC
    └── 1Lgm87NDqUh3di4etWCMtKBxXvDB1amBBG: 1.00000000 BTC
    └── 19ZsXQANqS3pTmiRkV8mcG527C5XPmviB5: 1.00000000 BTC
    └── 1EfnZnmfPLMwF8z7RJ1LK7ts7MakiC4b92: 1.00000000 BTC
    └── 12idtys9GpQetfkT49XSdwav3vdoLCDTLr: 1.00000000 BTC
    └── 1CQgyK7YCa5V9TBiy8tRwjfxsdaCkxmA5U: 1.00000000 BTC
    └── 1FgNoCz2Qry7i7WY181b3AUxjset9ZzxH7: 1.00000000 BTC
    └── 1HYAGxHrceXy9TeY8meLdzBpcLgnxDTG7w: 1.00000000 BTC
    └── 1oYfis9tJMUaujfQLoGASmHCpdcexX8xo: 1.00000000 BTC
    └── 1EBBwkXikPbh7uXwJiXEd5AfK5Y4SYd5Np: 1.00000000 BTC
    └── 1Pgbwh8Nvncoqpkr794j6vHsG3wbFePRPR: 1.00000000 BTC
    └── 1BoHTGE5krZuiRMoEwVPt6V4e7CrcYEy6q: 1.00000000 BTC
    └── 1EX3Nyzsa6rs5yxv7UKbVTCqDqp766NHVd: 1.00000000 BTC
    └── 1Q6D2wAfi5yaqxEhr1QYPbkkwrrozj582t: 1.00000000 BTC
    └── 1MVddnLoqc8K7MdnpDjHPCyVUkeencJEUy: 1.00000000 BTC
    └── 1BWeSBVChv5ZtMLvLEfwHiAdu1KZXcX5dV: 1.00000000 BTC
    └── 16bWeyhKQg4SdY8XpfWtCdMRLjp4E9XqYJ: 1.00000000 BTC
    └── 1AquZ68eFQVVfPjFMcALXjpqSGPZbidXXB: 1.00000000 BTC
    └── 1Q49e1Aw4K7AdCr6vzewsPcWbbMC9CTPgt: 1.00000000 BTC
    └── 1vJmruesm6WyVEeheCEX1dTNFjiFo14Ww: 1.00000000 BTC
    └── 14nwhmt3dtUUYbfMxuWbuvUe5wz1xZJeRW: 1.00000000 BTC
    └── 16BYXSPvmV1rAZniiAuagAw9Pbqi4VtTbQ: 1.00000000 BTC
    └── 13uout8HEnCxoFPXiUKqexXNZvxNZVmAH6: 1.00000000 BTC
    └── 1AsYSxYP6PqpL9qhp8F7Tj35MKNNsjwBZA: 1.00000000 BTC
    └── 1MSBoFQPq1qARQCxbLXmWVho9Yxe5hz65m: 1.00000000 BTC
    └── 17VUn7QwqdcxA37QSk6MLrZfdMAS5Y2H4S: 1.00000000 BTC
    └── 1DTMuPhm74D9o4ERSjWcbCcnYpn1EFNuyN: 1.00000000 BTC
    └── 1M6gr69GUm1paDHwe1rW9EstKvhUrsC49N: 1.00000000 BTC
    └── 1Du9RTMtaWDrWWdndzFdwgWkUFEDSY1gzd: 1.00000000 BTC
    └── 1JZf3NsC4tfsi2UQvTzW2rDxjbSt7Ro4BA: 1.00000000 BTC
    └── 1K9zX4K6vANJe6uBBUNMLHELDiNpkXfYLd: 1.00000000 BTC
    └── 14x3z2UjJ6nkDY7BtstyQqWzYsibJkQkX3: 1.00000000 BTC
    └── 1GEn7Mha7mG7z9GHhJw5ipeUQCGFQJKKKq: 1.00000000 BTC
    └── 1Cip9DFjwarbJQGJEREhh9rk3pQhGBv4L9: 1.00000000 BTC
    └── 1Hbr46BDoqznevaPsWAtUpEkK6jS4CxoSR: 1.00000000 BTC
    └── 16gUffoRLAmFQRMHfRFQ9qkcgBUkyGANEV: 1.00000000 BTC
    └── 121kyuMYPKRCmPBbssiP74vmNNKUKcVhxw: 1.00000000 BTC
    └── 13PCUAxTwGmpB7vdiHAdtEDJ6o5VKCnqcP: 1.00000000 BTC
    └── 1E3pNqjFcXRFQ4XD2g3G7GWXXb9BVL9rSE: 1.00000000 BTC
    └── 1Ept7MknRxC2LaT6F3QcvqJ6kGgQTxhGRS: 1.00000000 BTC
    └── 1ADoTNYq7BF5ePMZJw5Jnw2H2qf6er8TAd: 1.00000000 BTC
    └── 1Ccsr6hiWWh8HtfqafnQkRqmHcDNnMgU2K: 1.00000000 BTC
    └── 1Knu7CELYTk6Rt9Powm39YNnrL34PhfLRz: 1.00000000 BTC
    └── 1664Mg1oFShN5ksEYWFjHMRa9nHdCWyesY: 1.00000000 BTC
    └── 1MBK4Hyoxgk1pQJz4XJp8s5JMR5A7x7zpg: 1.00000000 BTC
    └── 17rRUzB27fEhQQLdBjvLSQZRJSATa41UbC: 1.00000000 BTC
    └── 13QvHzFizRgoGP8jQXNMb1FS3XQu4vcYQt: 1.00000000 BTC
    └── 1GZYT3whPiJohehg2Q6CEWeLFyM4Gmoh9n: 1.00000000 BTC
    └── 19XFbTZ2K3JEW2y9GFbeLyikfaKtEUNTds: 233.99700000 BTC
    19XFbTZ2K3JEW2y9GFbeLyikfaKtEUNTds
      └── 69a39eb32d93639537e6f3321fd60e608225272699a09dad5f511f38139bd6fb*
    └── 13PXaaW9yZqE7wesZYYMqnx4tHbB2KxjGi: 1.00000000 BTC
    └── 126jRsxFerQiWA5GhmATZjqhojwkRcRc6S: 1.00000000 BTC
    └── 1JVy4DoYQW9JNjNDMLWvrCGH5ZaZn6XjXC: 1.00000000 BTC
    └── 1NA1HY3qMvhaPcZ9cLUCqtcBPsSdkfHfGC: 1.00000000 BTC
    └── 18UMqMaZz3YHXUzuYWvaNKv29g6GFLb4V4: 1.00000000 BTC
    └── 1BbMCKydGe1kbaFJpKzZWG54Zvnv5rr7ad: 1.00000000 BTC
    └── 1D9AsegCs68DmVxFx3tzfARDzQhEgVyFcV: 1.00000000 BTC
    └── 1M8uGvfbirE4YGctTAQsuh51yV6LH9dyom: 1.00000000 BTC
    └── 19wMmy8NkVYyUmVL1hGSQnJhu7fa39NG7W: 1.00000000 BTC
    └── 1PvHf31kY9EPhDxi9gGGGG1vyo2NttB3hC: 1.00000000 BTC
    └── 18k74WJGV4MpihLmR7EN7JL44F2n4vYwUv: 1.00000000 BTC
    └── 1AH9imHhiiyahNyLdhnDk3rr9dhDenaTT3: 1.00000000 BTC
    └── 1PzbHnWgibFyQs4M5HaUMsWkL24HsrUnFM: 1.00000000 BTC
    └── 1F6QfciSkNntX9ajqpEAwXqP66YUa84Gzw: 1.00000000 BTC
    └── 18zSs2Z1kksBzvKJ4CD31RxrWBq5dEbYHM: 1.00000000 BTC
    └── 146J1XZ2ijPT7MxiYiLr8cnUT3PYuC8zTa: 1.00000000 BTC
    └── 1JShCtN5KdvRCWjg5Yt1w13gmXb1ap2P2S: 1.00000000 BTC
    └── 1LKisiMowZN7nvGaFsjwKjJH3KcNwPEwpS: 1.00000000 BTC
    └── 1JDAR5KZJezd39knvYAbdAeg6aAQgjdPzr: 1.00000000 BTC
    └── 14bEcpGrqhr8XbJ1LDaFFT8KADPUrqNb8z: 1.00000000 BTC
    └── 18sRJBMSmQ5umnufGgjavFUYAcqZTC4KQb: 1.00000000 BTC
    └── 1D7SQ9RfJhhkbanJBpMdTfzNVEWrNyH1zp: 1.00000000 BTC
    └── 1iqKgxHunryvcTq8yp2CBJc4CnV91LHHj: 1.00000000 BTC
    └── 1gQGbFjupurdhycTDen4AJwen4mWjKj8R: 1.00000000 BTC
    └── 1EX5nYBCh5LLDszkciPVTf9gKp71tvE4T8: 1.00000000 BTC
    └── 1FQYKDpYoAdyYMSajGWn9qdDseFqx69RTc: 1.00000000 BTC
    └── 1MWpCn1QYBvezTqLBH3FSoF3HNhuRQWG5J: 1.00000000 BTC
    └── 1GxmjhEsgL76xT5JPWRVEcK2Ba9texGuq1: 1.00000000 BTC
    └── 13W8WrzGtyKHkvMjBREMfunH9sb7v3J9Tv: 1.00000000 BTC
    └── 1Mq9LbuVEsCyPaiH6ETyEv57F7V9S1SYbV: 1.00000000 BTC
    └── 1KHLKWfuAmb2xFUKi2QQq5h4cMVRRYnuP6: 1.00000000 BTC
    └── 1JP9hRN8eJ5VCGp4S48qEpEZC2Fie1PfXt: 1.00000000 BTC
    └── 15S4uYpV6kwAHwiF5QC99x2kpSQZK8G1r6: 1.00000000 BTC
    └── 1Lb8X1PrDio99ZuNEvNAeQVsYr9Z1f5DTz: 1.00000000 BTC
    └── 1DFuM5M5DsENJwFdn7t4QEVuTHKLhvh5Cx: 1.00000000 BTC
    └── 1CVytN3qrQJ4uKaZcA793AWt5Yg3tafaEm: 1.00000000 BTC
    └── 1CkLtm2si3m9CWLax8KqvBHMkk3tMX49SP: 1.00000000 BTC
    └── 1DK4ErGF8Quf9dviW8FiFByCYbV4QUnbfB: 1.00000000 BTC
    └── 15T6hsNavNXLuZfNhFymsAvBh7PpeoDk7q: 1.00000000 BTC
    └── 123HbKmuHLqvA6U8xJxvyueuZS9LbqBcQ3: 1.00000000 BTC
    └── 1MdLfjZNX9HTJH9bfvexxWYBapqmC2FwUW: 1.00000000 BTC
    └── 1LP29fDMK8mjvbL1zriubmAicg3TaLEaZn: 1.00000000 BTC
    └── 14MunRHFZMwaLJf2zfUtcz8jFHPY7MJY8z: 1.00000000 BTC
    └── 1EfjzEoKYp91koXBY8MazraUcapy9mRpLA: 1.00000000 BTC
    └── 1ETR1dHtPzBYMncpZqEa3k91c6DFk8bjjB: 1.00000000 BTC
    └── 1Pzhdc1NmR1GCmDT24DsePVjWbqBp7oxuw: 1.00000000 BTC
    └── 1RKcn4VBhrMTWYRJ7X75u9SwMqVNDEUaK: 1.00000000 BTC
    └── 1NSyVFa713xLd7copn4yxw8stSvHYoTruS: 1.00000000 BTC
    └── 19mSUtxqprdRFD2CpdrMURVWQDHVZcsCqY: 1.00000000 BTC
    └── 1PWKJ1MUFSuVL2WY9D3b1JS8ttB93SUNef: 1.00000000 BTC
    └── 1MTUJ2yU14io5PHpKVgKAfVvAStFxJcCVP: 1.00000000 BTC
    └── 1Jj4YWvwHetLs3jURYmQVpbqszcbZe1Amw: 1.00000000 BTC
    └── 1JBNJkLSJu7baWEwUEz6xWchr3AnqxhxBq: 1.00000000 BTC
    └── 1BZgPBo462xYUJUgfTubFBcxiZuJfAnHwn: 1.00000000 BTC
    └── 1DDBZT1Lm3q8APzNoamh2ybjTYbBT31qUp: 1.00000000 BTC
    └── 12LZqg93FBEjJuXLo4N8MouLXkSdYf4svS: 1.00000000 BTC
    └── 1M5CAcjeQV3bNKRncytbQFhEDGsNji5cFw: 1.00000000 BTC
    └── 1HEPNnqzaRSqWjqzFgFmLrVsxC4oNfVczy: 1.00000000 BTC
    └── 1D5MTWMPUNpxutNkqJTVamTmNqg5ueRP5m: 1.00000000 BTC
    └── 1PaQPZRU7jdpAzGzNGEgE6aZwWVfT79yjx: 1.00000000 BTC
    └── 1Q1rJCQ4ABTArewGhLE9TYqxuTAWoUssuZ: 1.00000000 BTC
    └── 1DrRg5Ck5ruVmNn8DXjj7SM4DstrKcKfgs: 1.00000000 BTC
    └── 15qMB9Rsqq2mdvELT6tBk8PV3LBjueYtWG: 1.00000000 BTC
    └── 1PiEHXEyB6nxxe6iXcjWiYMPw9yPr6iEZo: 1.00000000 BTC
    └── 1CoZnmsDzPvW9L6RByhV6fWrFSGFxNczcR: 1.00000000 BTC
  └── 88c5e1c51c66ff0e4048911eab494daa07938cf92fba3e972d79c5f3465aebd8*
  └── 1ebd60c8ee1263501dde868e3856a484a2cae165412622e9882227d64411026d
    └── 1HFyaH3ZayeZrBX6yyU3xeQ9KA1JSTkHX9: 87.00000000 BTC
    1HFyaH3ZayeZrBX6yyU3xeQ9KA1JSTkHX9
      └── 3f6093819980e48688b6e375f646d3956a28749d8d4bcc41a597c6c8ec40778e
        └── 175X7AV6g9UhS8mTPnFTfjLmRAsyJXCVpp: 1.00000000 BTC
        └── 165DRG2beKZsnMdUXYFe1nU2Roi4T5nLu3: 1.00000000 BTC
        └── 1Nu2VpkU1TjzBXVbfvkL7mxDn6itUn4bY2: 1.00000000 BTC
        └── 15AcSLuvNq4vob3hZLyiCTiNX2gmNcgdM5: 1.00000000 BTC
        └── 19VGmDkmHWurCkNJ5Ref5XNF5NBTLnxGwQ: 1.00000000 BTC
        └── 1GqJNmnYrWXftryXVQbRN52Ar31BwHye6H: 1.00000000 BTC
        └── 16azYWxJyoQkjwDrGyMMnD1fo7L8QqjtHS: 1.00000000 BTC
        └── 1BXgaXqV1TKa9nyrHzCWUFarL4R2whvAKn: 1.00000000 BTC
        └── 1A8yLazWAbLTFVFZC8GbAkdvfMX7Qvqfe3: 1.00000000 BTC
        └── 1PYjQowtTYT5aFpnsV7RMaM6k2KxRCMFvV: 57.98000000 BTC
        1PYjQowtTYT5aFpnsV7RMaM6k2KxRCMFvV
          └── 5b7a35a7d2be184ad9932eeda5a7a4c9306c4e43dd70b813b3210e525ae18898
            └── 17U5dJA3Gz16rTtNjkusGeryEwgr7vR5y6: 1.00000000 BTC
            └── 1PDKMaeD9Rk2fW3GZiinkC6aEkoFrcDxB9: 1.00000000 BTC
            └── 13NPpoy24UaNBKFjqCRAv2KqkBCz1zDVVD: 1.00000000 BTC
            └── 12eYqyefcdBZqaYQn8wrjVB9r83KcXfDpr: 1.00000000 BTC
            └── 1GpHRwCfyFNSU2MDHcNNNERMfkU5juUAzs: 1.00000000 BTC
            └── 1KU6z6sz7mG1zDqVeDUrgMM8g8sMA3YKJr: 1.00000000 BTC
            └── 14Ajr3iUzYnTdUeoPpCcj8fCGShnv129KN: 1.00000000 BTC
            └── 18XLgmGUooA5BU4SSd19UetZxAgcQEhaVd: 1.00000000 BTC
            └── 15mKdbgxXfYKj1VUbWjBWqHycBBLEURmL3: 1.00000000 BTC
            └── 1MoLf13DeF1wLtFxJyvyRbTjqMSA9bTYBJ: 1.00000000 BTC
            └── 18odvGqtpsRh27xUxDND3qT7du7hMcS5it: 1.00000000 BTC
            └── 1ArmSvXJddmWW5RRNEqr8xSADFabzp2dZX: 1.00000000 BTC
            └── 13WEYe1sDpvngo47xTgdC2JZFDqJUSkzw7: 1.00000000 BTC
            └── 1Cz82b4whxQX5WqMY5UH6HWghwDisyVGVr: 1.00000000 BTC
            └── 1AydWTkfDjcd5MbJk5x5yHmuCRxwRqyCxu: 1.00000000 BTC
            └── 16yWjx2Uyuo8CYrHKiMb5iitn7akhPALha: 1.00000000 BTC
            └── 1NY7sZg2WwaXAoytJPgx4LjfiuuM3EZKz5: 1.00000000 BTC
            └── 1D6YbPQap17vmvxghWPNfBcuKfLjoz8Z94: 19.96000000 BTC
            1D6YbPQap17vmvxghWPNfBcuKfLjoz8Z94
              └── 4157853bf55daf4c57d6b5ee2ed27859afaf36ef38127cb92a43e4000797f4ff
                └── 16DgzFZiRQKyF6t5JxCDJP8yNmJfv9uZvu: 10.00000000 BTC
                └── 1PaqLwnrPEiuqS2YW5BuEWDFVYfgidfF2r: 10.00000000 BTC
                └── 1CiyaoDihaprxUBGuoxHm563qye1cDghfr: 10.00000000 BTC
                └── 1KFjntF7DEAT5CQiMqrQBFPbHv9seu6TjK: 10.00000000 BTC
                └── 1Fbq1HmzbweYrstLaFbM3dPsUKk1f5P1RE: 10.00000000 BTC
                └── 16NWDVLUyyv8X9zfp8KQVqnh16FAp3TRr4: 10.00000000 BTC
                └── 14WVyyU65THNHkkd9cARbwz78wnS4H99Wr: 10.00000000 BTC
                └── 1BNs3ghuT6JyCWfDQFAQ89Lebf8sxtMBU9: 0.91000000 BTC
                └── 1JUjbTHp4ki6LFPFHYtC8isdteku5N7ZSG: 10.00000000 BTC
                └── 14yaepF9jXpBvh6MkhpULjMLgUeLtTbBBZ: 10.00000000 BTC
                └── 1CtDMjnf9bdWgd3VNAMFLrArCnGNRMNr7d: 10.00000000 BTC
                └── 13rqdpRT8ELktdnRLVyVgmTAW8xZ8A79yV: 10.00000000 BTC
                └── 1PjKgQ5KydwzCnFVtjcr4GGuSC9KNJpucU: 10.00000000 BTC
                └── 163xmvguurRjyy3ZQrcgDsBN82vztxar8y: 10.00000000 BTC
                └── 14MBw3NH1M4Uw3dU2xJRT3BULSsvvSbXQH: 10.00000000 BTC
                └── 1PW2kYYULFFmEieu7RMbxb8Ee8ATSYQkGM: 10.00000000 BTC
                └── 1GrGyxLBzmRgv1pcfYAeTVMDyeWGKUNoJE: 10.00000000 BTC
                └── 1ACq3LQpEp6TmTRwDT3JgbocEWhoTZa2xK: 10.00000000 BTC
                └── 1EHSryS8EhFZizAWWst3SsKjsJUJVGpX3b: 10.00000000 BTC
            └── 1FRqpbosfGijiRtCVz7eVjQpJ31Bsdmeox: 1.00000000 BTC
            └── 18kjQxzYYkUHQ2UCU5uGZca3vXo4bczdT2: 1.00000000 BTC
            └── 1958wxWv5ZQsyzbLCLQWGdDv7YwjiFRyzJ: 1.00000000 BTC
            └── 1GDnSAhbaPPzB9mgid56ADaFr23B3Caeap: 1.00000000 BTC
            └── 1G51yix3MmSFwEzD77cJ6sLgVCoZQgS2nU: 1.00000000 BTC
            └── 1FygYZX2TXqmopRY2WETV1CBZPposoisTM: 1.00000000 BTC
            └── 1DkmN1TK5PgEzALXdBeB8kuWh5XuMHWj7j: 1.00000000 BTC
            └── 1HMDAJJZVYhHnjaXU7qgBcBMv2PfyW1qg2: 1.00000000 BTC
            └── 14PKy4pmxkmjwh2eFGXoWKrUJt5wnor2SP: 1.00000000 BTC
            └── 1AL7o5p9YzGULJHyVC1TA3BzZbGN7bVz8v: 1.00000000 BTC
            └── 1BLriV1zBsnpdiwhtPsaNcy48dw1e6vtNL: 1.00000000 BTC
            └── 1FQyyRYLpSECr365Y4KjTr7zaJQvVGNMTM: 1.00000000 BTC
            └── 1JKYAuZR1NxciUfugiB245ogUqdzjgxGND: 1.00000000 BTC
            └── 1G7wed81PN1wFeSvv4RTfFcb7LP9HruiGY: 1.00000000 BTC
            └── 1Pus867YQv7nwAoqewRtGXqhL9kBsv4RoR: 1.00000000 BTC
            └── 18y1ViTzLzRKopme4pcaCMR4MuG5bBwEtK: 1.00000000 BTC
            └── 1CiKeXdginbbDUmQPV5NrK6QZAoCAtVjvZ: 1.00000000 BTC
            └── 14VKEdUCBsce6x4UucJfGDq5r3Miqs16vb: 1.00000000 BTC
            └── 17R9oKvqR4NJhy5uYEppLnhrPUjpsg5mQ1: 1.00000000 BTC
            └── 1EXoxfEPbH2UrdFVpSihCd2DLe5FEddzgj: 1.00000000 BTC
            └── 17CoutvJ2fxbNbor7KTMaryNgKKXY2jrDt: 1.00000000 BTC
        └── 1ByfjmkjGxGGSKoNKga5yD8ZrocvrNRJZ8: 1.00000000 BTC
        └── 14XkaBu9hbKGgmeiigEgRRDmLmVF7du2Ev: 1.00000000 BTC
        └── 1PvXGhStoTYybGWahaCXXod4xANWeUduda: 1.00000000 BTC
        └── 1DTih2SjeSXi5gdWGjz6EjwxfuokD1XaCj: 1.00000000 BTC
        └── 1JgTdf4iDRer23ZY6rB5epKaxbMLepUHcY: 1.00000000 BTC
        └── 1MbS1N8C7tqtt36F6p3Ud5V9XVE9H8JBgV: 1.00000000 BTC
        └── 1GAQGtYgKG9ztJgfgqcxXPV7ZaU1bVuFfk: 1.00000000 BTC
        └── 1HCbdB7aju63nwN7JSoeZWXwWCgfF4kwRS: 1.00000000 BTC
        └── 1NdnHyv4zqTi8z5XM2W26zf9yTYRmJmxVN: 1.00000000 BTC
        └── 1Lht9pxa9BtfBDSyChzXCc66HTvxwv4NT4: 1.00000000 BTC
        └── 12g1egrBd1hGeD6L5gzZGLMMvsLtKV1xLN: 1.00000000 BTC
        └── 1ArBriNcKD3UTgwLtJ5g899GMB8qVoAtNz: 1.00000000 BTC
        └── 1FjHANXWHrg2tmKxHAKrPQVePNkxN8nw9p: 1.00000000 BTC
        └── 14RXFYoaLsh2SXDKLEMbf2FbpwbA2B38p7: 1.00000000 BTC
        └── 1GM37PjoLLPsXUDgeLkYzzbGYXGkhiNEm7: 1.00000000 BTC
        └── 1Q9MMYcCAbqE3pn6BHNKHbdQWXdy2zBnJ6: 1.00000000 BTC
        └── 1G1KfBiAKPbUzFBPwRqJcyMV23yzc5gSaQ: 1.00000000 BTC
        └── 1L5YaoA5EmMFfAmTF24n4esgijZqLmwR5n: 1.00000000 BTC
        └── 1Aq1wxzZdeRkP7uaffTEYe6tdNd2JGePeS: 1.00000000 BTC
        └── 1PMeL8dAUf7QheALs1KX3qLUMCpALHRJQt: 1.00000000 BTC
    └── 1HsWjgX9N7Vn8boSh12ggTpBjY9vPMDmaQ: 1.00000000 BTC
  └── 5deec1f3d9165a2fcd50e29e2a717bb55fad43147abf9719bed85762b96a8e9d
    └── 1JTZcVy44WmmycHYxkNnvupRpW3VknHfiS: 1.00000000 BTC
    └── 1JVqesXMAGNzxDNQ2TPRSnakmMfXJoNn1B: 1.00000000 BTC
    └── 1L7p6s4Z62XS3PKhXksLjB9MdKbSteJGrd: 1.00000000 BTC
    └── 1A7FjAFzk6UdRG9UMzm73DcpPYU8dTY8gE: 69.99000000 BTC
    1A7FjAFzk6UdRG9UMzm73DcpPYU8dTY8gE
      └── 4157853bf55daf4c57d6b5ee2ed27859afaf36ef38127cb92a43e4000797f4ff*
    └── 1Jo2DWeEqMx9DEVXfu8Gr2Jy4YzV3ofqUW: 1.00000000 BTC
    └── 1GNWSizCVP7K8kFkTii4D6cWXtZrwMmor5: 1.00000000 BTC
    └── 1D5wUqrcJjvaaPRNjuAVxbnvc3xLyP39ye: 1.00000000 BTC
    └── 1DsJRebJyQHpJ8rLdFEgywyJowUAxVHt8Z: 1.00000000 BTC
    └── 1B7Xe3MKvW2M3Qdt4LS8cr2519g2kxNkKv: 1.00000000 BTC
    └── 17R6pGLf8GbXgnD1vnC1wa7FQ3QwjBLupt: 1.00000000 BTC
    └── 1PQ2VFWW1wz1zDh8oG8EmqD9GuvRCa2J1L: 1.00000000 BTC
    └── 1PNoDnQbtTv7njs5MiHnGcBByAF5338ZKz: 1.00000000 BTC
    └── 13S8vf1UNRbMZteUG2jDs7MnEDQtpVD9eH: 1.00000000 BTC
    └── 18kYwZxAZ87ShL2qjxoch3SFoThrzx9p3u: 1.00000000 BTC
    └── 15bag4QWwQmv53ZSEnQxAPuvbZe6gWuQ8v: 1.00000000 BTC
    └── 1NcsxLQKoFVqUX2qJXiE669qBCWZcEdjpm: 1.00000000 BTC
    └── 15CeXrYxtzRxtLFWvxasydJxor2WcVuKYe: 1.00000000 BTC
    └── 1DG7LxLBu1eZB9jJ5aoC4GBe3t6GPRR8Mu: 1.00000000 BTC
    └── 1B4KEuwPnWYzeAbKWdUX7zasRWkLNnBVDs: 1.00000000 BTC
    └── 1BPmAjYsXvm39cwBcsJqbZ9bRqnCYu8tVX: 1.00000000 BTC
    └── 1FknTPNmVSqj57z84qzHijAnMBWj3Hu79r: 1.00000000 BTC
  └── c0ecaa1869188c0544b33d07b983434f545cf7bfaf669a48ced3951316439616
    └── 1EKYaN9XeW2TLygYE66AcYfGjLT9UGt2dS: 5.00000000 BTC
    └── 126yaPje7RkLYTMLhDdiAwRTLM5qmewJhp: 5.00000000 BTC
    └── 1LpJa4yRw99EVp1tXXB33Y8okqkawENhUr: 5.00000000 BTC
    └── 16tVDDFb13RzeW9qpL4ZDDFcvSa1DqMZyk: 5.00000000 BTC
    └── 1DxtR7FXZnM2xyb8Mae77XNBBW1gXskwGe: 5.00000000 BTC
    └── 1JtAFhqwgAkVJLwkAm8jUXDKfJvCVYvYeM: 5.00000000 BTC
    └── 1EsqUUJXbNfdZPETpqpK9q1XtC8t1juzmy: 5.00000000 BTC
    └── 1NUwqZsbNhMwEA289FEsUrGQLTnLZe7DXT: 5.00000000 BTC
    └── 1CXTquwt54M2nVJkhkXZfVfARxkBo9yAGb: 5.00000000 BTC
    └── 1HJLaR3NRjg8iof8UmdEugiE8y3FUUngAt: 5.00000000 BTC
    └── 14MF8zVKTmx8DZkYApawtNWQL6WTqs8KGZ: 5.00000000 BTC
    └── 1eMfnArLKC2S6VMgsYQB12d1Hn5dd5sYo: 5.00000000 BTC
    └── 1CsNyy7cVRBh9uy2yvgcJvwJGLAFRF7wSL: 90.98000000 BTC
    1CsNyy7cVRBh9uy2yvgcJvwJGLAFRF7wSL
      └── 4157853bf55daf4c57d6b5ee2ed27859afaf36ef38127cb92a43e4000797f4ff*
    └── 1JzgsHEQfU5W7W4P91cAA86cxHjHbSbc43: 5.00000000 BTC
    └── 1241BdUkDUUBBJmeHSVjQjeHbjiLDZMZdY: 5.00000000 BTC
    └── 1Hqv22oCVNs2JP5tYpAAKTGGvoE8Pjyhkm: 5.00000000 BTC
    └── 1EBm4UvwqssQC67sqHT4fdMe3fu6q5y9GB: 5.00000000 BTC
    └── 195Ps5rxnpvsXjGGFN5xRaCpJndqE1HPdC: 5.00000000 BTC
    └── 1D1zb8ptPVGC6ypn7ZRT4WouMzvX3BPHCu: 5.00000000 BTC
  └── e440fb286972e803bb3ea4c007ef355854174f9bc2025b35cb9ef00a98b6c005
    └── 199nyG8Z5CBVuHZuCoSqYUdcAS4q4BUEHt: 10.00000000 BTC
    └── 1MKP7wFGg6qf5rck8A8mD85TMfzBNnM9YG: 10.00000000 BTC
    └── 1Bwx4Ehu9ywoxQKiYCbtG6mbAkgisW9H6m: 10.00000000 BTC
    └── 1MvX8zhePgq3fkWXtHY4HQUoBdLs4nijzk: 10.00000000 BTC
    └── 1Dk9fosZvvJnee2LCMJLngbtRsGror1p6R: 10.00000000 BTC
    └── 17Zg6YqFP9ftTGyqGAaFk6ZQLDbC4oNen9: 10.00000000 BTC
    └── 1XxUHZrDfbqzYkkRs4dxTvwz9xsaaEgw8: 10.00000000 BTC
    └── 1P6ZWkVZQGcRUR6jSKV165rDQj7mMdnmZn: 10.00000000 BTC
    └── 1EyB63e7YuNqWcjktgbjDth7ncKTBBC3y9: 10.00000000 BTC
    └── 1SzhR4v5hYf3fKu2US5FbuAFhcnY5xUrg: 10.00000000 BTC
    └── 13vmyWMa9kJ7UtsSXSZ61mxgvXQQGX4zwv: 10.00000000 BTC
    └── 173zTWASJjaAFssUSoWjtVA7bTNparfmbg: 10.00000000 BTC
    └── 1GG14EscCgjdRZrk9JrZJRhH5Tb7FFsJBT: 10.00000000 BTC
    └── 1NsQmU61A22xiuSvaDZ6YvZiyFvrSoz2JE: 10.00000000 BTC
    └── 1LJzK4SoL7h3SfmPRnxM71JU81H2kGe6rh: 10.00000000 BTC
    └── 1JyxTFSEaYXXSxWBjjF5XyFPut44VbM2Nn: 829.96000000 BTC
    1JyxTFSEaYXXSxWBjjF5XyFPut44VbM2Nn
      └── c262797f2d7c790db63a4e5ce4b4bb2a511de2c0e1516dd6d7b6b6f0c749f5ea*
    └── 1NNmdNFEm56tD8ujodrvabfUqoXyY5Q5RP: 10.00000000 BTC
    └── 1CfWyA4bfSwzEhrVnrVNkkxdybxhakN5vZ: 10.00000000 BTC
    └── 1AzDAssrzKrsd2HoJ3mDPQjzqsDSvF8L6M: 10.00000000 BTC
    └── 1FSAkq6EaiFz1mvJAWkCfpC7a9FSr8Ci92: 10.00000000 BTC
    └── 18iNoijWa5cgWbnH1oCkgHrhCMPpJzSEmx: 10.00000000 BTC
    └── 159VjbaYgh9FNTuEHQ7R8igyETqULWSNF4: 10.00000000 BTC
    └── 13BiLzSJFJUp2XjBtGr9bLsFU2pPMsGs5s: 10.00000000 BTC
    └── 1PaxoVGfvgmWAuten4CwE8sKhrwxWX8fu4: 10.00000000 BTC
    └── 1LBbkY51Zywv2m58mXZisVeNkeDQWSh9bN: 10.00000000 BTC
    └── 12NBYNJDjcrZCwFSKGGE4zLk22Xd8a2fz7: 10.00000000 BTC
    └── 1LnFmYZJbZPttqx9gGYFsGnxD68LMyBRm6: 10.00000000 BTC
    └── 1JY8ywzuSH8pPVe3dK39VAFiVPRpgSsVSm: 10.00000000 BTC
    └── 1HDdG7db9epmTTSJ3P9fRGE1JGRGTDRhYG: 10.00000000 BTC
    └── 1KQKxiGioUREcUzhy4US1CRQ69uYjZX42Y: 10.00000000 BTC
    └── 1MQjzqi4NXnXZZRzADNX7cijKqKekaVSff: 10.00000000 BTC
    └── 1N9bQA8gEb2o6uXhaRjaayNPLoKbT41LN3: 10.00000000 BTC
    └── 17zMKhURErHpLBrrdJjvXwV2i631BLoRNQ: 10.00000000 BTC
    └── 19bWHN9vExHt6fo6DGjHf6SQnaSGYTAWiV: 10.00000000 BTC
    └── 1412SvGyy4NqfhggW1GCgdt4m5pv3guEYt: 10.00000000 BTC
    └── 1LTbopgFutYL2ikxpVctmfNb1uEStWDtDF: 10.00000000 BTC
    └── 19TyUYKLxfLaTvywiTQ4x9ihGPi4aB2CrG: 10.00000000 BTC
    └── 1D6MWiZgyNwh4Mp4yAXEcKTZwURzYb1QPJ: 10.00000000 BTC
    └── 1FSr3ELWDdu5PZMoXqaeReqMiV5tZi9Uhu: 10.00000000 BTC
    └── 1DDT4BCsmf2P1RqtAWftT6ksEJsWcBtyPG: 10.00000000 BTC
    └── 1295X25FTRuCg3Pb43QyjLsgRurqHg4J2X: 10.00000000 BTC
    └── 16zBuN3RmJ2FtV2yD75VitCvLgyLU1YQwr: 10.00000000 BTC
    └── 1FcRA7QqknjT1hsNibWtgYenfGrVLkEfx5: 10.00000000 BTC
    └── 1BCJgTM7ot9LQyYvgPrnb7p3CMcWsUh9ps: 10.00000000 BTC
    └── 1GxWo7q1UQoonZfrUmcNqGrsmqtQSPVMcP: 10.00000000 BTC
    └── 13jKUbwHaWQyPF7fY4kEkMUjy1ewKH5Pgv: 10.00000000 BTC
    └── 1PiNWdQARxBpt6g9GdAEYHkPiiGa8EvuYb: 10.00000000 BTC
    └── 19qy4JY44Vb5nbAbEuCd8maJ2mVy5N9ekc: 10.00000000 BTC
    └── 1LFGCWWBqj4MiumH84oxa3HjBUgZmngkrD: 10.00000000 BTC
    └── 18viBXsPt7WFUku7TNgsT9Uz7UQjc31r8g: 10.00000000 BTC
    └── 151HCiHhaTqQAdD93j3c7bdV2r6nJEcC5f: 10.00000000 BTC
    └── 1AhL2PqbSWX9Yy1NRM87zw2Y3HpUDVtCvC: 10.00000000 BTC
    └── 1DhHYLRS5q7yBZyihysrAJnvqN5DtpHX5Y: 10.00000000 BTC
    └── 1i3UQCBscLv3q7xQkk8yka42uXBDTKFwE: 10.00000000 BTC
    └── 19JSinMfcArqu6CkLauocKDVTCdfpAtweT: 10.00000000 BTC
    └── 1QCE3zKKZ1KCuzduYVGDQ55WfEVtnY9Wnh: 10.00000000 BTC
    └── 17QeD5mpxDRJHpsCzagoey9y52WtpcCgoP: 10.00000000 BTC
    └── 1Gg5V8ZjU17aDbr6edCVQFf5ef3Hw3fUSR: 10.00000000 BTC
    └── 13gfPHStgixjzu1vMfhwuitp3Kk65RVGbP: 10.00000000 BTC
    └── 18EypXUsMFGkCGuCJBdwnUXtBFcmmytcQK: 10.00000000 BTC
    └── 1652TSE6k9YU1eZJy77K45bEnhqV7Ms4iX: 10.00000000 BTC
    └── 18BkdfUiuyeQWPofL9ueHeY7UuyZ8hzcVh: 10.00000000 BTC
    └── 135u35xwxjVRjgi7tgdQrwwgGFmeLb8Yb1: 10.00000000 BTC
    └── 17H5HLe2WcjX6S71DA4EmA82q4BjeoBMok: 10.00000000 BTC
    └── 1HDfDJN9KxH9HXqUa9E1xX3Fi4MFrdVgR8: 10.00000000 BTC
    └── 1GCA9ik2mbPKgaUU6pBbXcBFstaHiKSX65: 10.00000000 BTC
    └── 14a9EUNjjeNh9StRbeH16PgvJaGa8dvHkL: 10.00000000 BTC
    └── 149xjEZYKBjsQXfzxWRt3L7bREdEtHMv3b: 10.00000000 BTC
    └── 1MVoXVLvcPq98HKQH1Emd5BMZvVKdGhoXN: 10.00000000 BTC
    └── 16KJzXD4cRqPMt7xzYNu6rZCxikC5HW2CY: 10.00000000 BTC
    └── 1FHkWgKPY3a6eSJjNYQAqgZDRmz55Fh79h: 10.00000000 BTC
    └── 1MWHGSNmKRDMeHCKQ3V1CH6TYvn9hCLBzF: 10.00000000 BTC
    └── 15vb9aeM4Tk3yt9VtwzMxU1jMf1XsPJCTy: 10.00000000 BTC
    └── 14njCxtgWgtU4tM524brM3fJvp6s1yjqS5: 10.00000000 BTC
    └── 17w8MenSniZcDtEaHobKGFB7sVzMDpM67K: 10.00000000 BTC
    └── 15KGEFFWbwafoVRg21CKZnXkn1DtWiwHez: 10.00000000 BTC
    └── 19HW5w72JyGi3cbnRWpTRvghA9Fwor8aj5: 10.00000000 BTC
    └── 1D71eCxQHi5hcVvcmSUb5GbMPdhzQKYpEo: 10.00000000 BTC
    └── 19cgkx6vmcJego1BMuBE8aT3uQoGmcmeKH: 10.00000000 BTC
    └── 1N8A71cPb2FAmpSyfiKKGKQXs8ALPizyVi: 10.00000000 BTC
    └── 1LyJte9bkzCVn3YmEngztV1Sxe8LLHeWwo: 10.00000000 BTC
    └── 1Lg3Lsxs4D8q2pz5wYYU6mmRfAPZkJz6mc: 10.00000000 BTC
    └── 12AHp2aF47izsx66F9n9SxVCSCDJfuychg: 10.00000000 BTC
    └── 1N6wTiPFXWXdWkbj9J6HNm5yHVETmojAkP: 10.00000000 BTC
    └── 14qJZRApNuAEd7tmXK8MWSoR99hLMX4iQg: 10.00000000 BTC
    └── 138prbgoUdzmgxskWSTRzTXUqjFBjXoMcP: 10.00000000 BTC
    └── 1JS1WWcUx65YSDnRx7sjaoHuo6gN4r2jGE: 10.00000000 BTC
    └── 1MQuyT1y7CkRHgjoj9PAYmN4KEvanwjEku: 10.00000000 BTC
    └── 1MyWJyxUcbdCFEVLuJ6UXaF26vNN49Jb2E: 10.00000000 BTC
    └── 1HLNzKeKQtzhwukp9CorbpKua8W3vJY6u9: 10.00000000 BTC
  └── 69a39eb32d93639537e6f3321fd60e608225272699a09dad5f511f38139bd6fb*
  └── d12b522c33ca1d22d070af08ab12280e793f4134c289b834de91e79b2f0f6247*
  ```

Transaction IDs marked with a `*` are detectable through multiple paths in the tree - I only process them the first time they are encountered.

## Bitbills Today

With this set of addresses now available, I was able to load the entire Bitbills collection onto [collectible.money](https://collectible.money){:target="_BLANK"} under [Bitbills](https://collectible.money/creator/bitbills){:target="_BLANK"}.

As of this post's publication, a total of 928 Bitbills were produced. Today, just 395 remain intact, with only 26 20 BTC Bitbills and 27 10 BTC Bitbills intact respectively.

1 BTC Bitbills were produced in the largest quantity, and also have the highest peel rate - nevertheless, I believe a significant portion of the 293 remaining 1 BTC Bitbills are lost to time, given the lower redemption rate than other denominations.

This also means that today, only 26 full sets of 1, 5, 10, and 20 BTC Bitbills can be assembled.
