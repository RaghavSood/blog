---
layout: post
title: Bitcoin's Signature Types - SIGHASH
categories:
- blog
---

Using Bitcoin day to day, one takes signatures for granted. You sign a transaction when you want to spend BTC, and that's about it. It simply helps ensure that no one is spending your BTC without your permission (with said permission in the form of a digital signature). 

This is all true, but somewhat of an oversimplification. Bitcoin transactions actually have a number of different ways that they can be signed, known as the `SIGHASH`. This is encoded into the signature portion of the transaction. Having these multiple signature types allow us to create some innovative transactions that do more than just spend Bitcoin, facilitating trustless sales, and multi-party contributions. This post aims to shed some light on these options, and how they can be used.

Let's take a look at a simple, one input and one output Pay-to-pubkey-hash (P2PKH) transaction from a recent block at the time of writing: [e251decc598148363669ae64cf0ec56f6049a0adc1eb42ac850f949aff0e6e55](https://blockchain.info/tx/e251decc598148363669ae64cf0ec56f6049a0adc1eb42ac850f949aff0e6e55){:target="_blank"}

The signature data in this transaction is a total of 106 bytes (including the push opcodes). You can easily pull this information out using the `decoderawtransaction` method in bitcoind, under the `vin>scriptSig` key.

{% highlight shell %}
PUSH(0x47) # 71 bytes
    304402203ff7162d6635246dbf59b7fa9e72e3023e959a73b1fbc51edbaaa5a8dbc6d2f
    70220776e2fa5740df01cc0ac47bda713e87fc59044960122ba45abb11c949655c58401
PUSH(0x21) # 33 bytes
    03bc3c9134f5a5e3f08287d175d7e43368f72cb93a2e6cbb801b5e90d1ed628e60
{% endhighlight %}

The first push here is the actual signature, while the second push is the public key for address `18w59Xd5ZXPsnx9A7buNgqmAvUNqn3cmUJ`, the sender in this transaction. 

Bitcoin uses [DER encoded](https://en.wikipedia.org/wiki/X.690#DER_encoding){:target="_blank"} signatures, so let's break it up into it's components as well:

{% highlight shell %}
30 # DER Sequence tag
  44 # Sequence length 0x44 (68) bytes
    02 # Integer element
      20 # Element length 0x20 (32) bytes
        3ff7162d6635246dbf59b7fa9e72e3023e959a73b1fbc51edbaaa5a8dbc6d2f7 # ECDSA r value
    02 # Integer element
      20 # Element length 0x20 (32) bytes
        776e2fa5740df01cc0ac47bda713e87fc59044960122ba45abb11c949655c584 # ECDSA s value
# DER encoding completed
01
{% endhighlight %}

That one byte left over after the DER encoding (but still part of the actual signature push) is the `SIGHASH` flag.

The values for this flag are found in [interpreter.h](https://github.com/bitcoin/bitcoin/blob/56f69360dc98bd68704f19646a84d045788d199e/src/script/interpreter.h#L21){:target="_blank"}:

{% highlight cpp %}
/** Signature hash types/flags */
enum
{
    SIGHASH_ALL = 1,
    SIGHASH_NONE = 2,
    SIGHASH_SINGLE = 3,
    SIGHASH_ANYONECANPAY = 0x80,
};
{% endhighlight %}

Although only four values are defined, `SIGHASH_ANYONECANPAY` is combined with the previous three via a bitwise `&` (effectively addition in this case), for a total of six possible values. 

 - `SIGHASH_ALL` (`0x01`) - This is the default in every consumer-facing wallet that I am aware of. It signs every input and output, and any change to the transaction will render the signature invalid. This essentially says "I only agree to move my BTC with this exact combination of inputs and outputs".
 - `SIGHASH_NONE` (`0x02`) - This signs all the inputs to the transaction, but none of the outputs. Effectively, it creates an authorizing saying "Hey, I'm okay with participating in this transaction, but I don't particularly care where it goes". This might seem insecure, and should never be used in single-input transactions. We'll cover why it exists soon, though.
 - `SIGHASH_SINGLE` (`0x03`) - This type of signature signs all inputs, and exactly one corresponding output. The corresponding output is the one with the same index as your signature (i.e., if your input is at vin 0, then the output you want to sign must be vout 0). This essentially says "I agree to participate in this tx with all these inputs, as long as this much goes to this one address".
 - `SIGHASH_ALL | SIGHASH_ANYONECANPAY` (`0x81`) - This is similar to `SIGHASH_ALL`, and signs all outputs. However, it only signs the one input it is part of. It's essentially saying "I agree to participate in this tx, as long as the following recipients receive these amounts. I don't care about any additional inputs to this transaction."
 - `SIGHASH_NONE | SIGHASH_ANYONECANPAY` (`0x82`) - This is similar to `SIGHASH_NONE`, but only signs the one input it is in. This is essentially like saying "Hey, I'm okay with sending this BTC. In fact, I don't even care if it is sent in this transaction. Here's a signed note saying that any transaction including this can spend this BTC".
 - `SIGHASH_SINGLE | SIGHASH_ANYONECANPAY` (`0x83`) - This is similar to `SIGHASH_SINGLE`, except it only signs the input that contains it, and the corresponding output. This says "I definitely want to move this much BTC to this output, but I don't care about any other inputs and outputs in this transaction".

Now, one might wonder why multiple signature types are needed at all, so let's look at some quick example of how they might be used:

- `SIGHASH_ALL` - This one is pretty straightforward, it's what we use every day. Spending specific inputs to create specific outputs.
- `SIGHASH_NONE` - This one is a bit more confusing. On the face of it, it seems like you're burning money by not signing any outputs. Indeed, if you create a tx with just a single input and sign it with `SIGHASH_NONE`, the miner would be able to simply change the output to one that they control. This is mostly designed to be used in scenarios where more than one party is contributing inputs. At that point, such a signature essentially means "I agree to spend my money, provided all these other people spend their's too". It is expected that one of the other signers will then use `SIGHASH_ALL` to secure all the outputs of the transaction, and send the money to a mutually agreed output set. 
- `SIGHASH_SINGLE` - This can be used to send BTC in scenarios where you only want to make the transfer provided some other parties are making a transfer too. Essentially, you agree to move a certain amount of BTC to a certain output, but only if the other input parties to the transaction also move their BTC. This can be used to create a transaction where you and a friend need to pay someone 1.5 BTC, and you are contributing 1 BTC. However, your friend only has a 1 BTC output. Thus, you can use `SIGHASH_SINGLE` to create a transaction with both the 1 BTC inputs, and a 1.5 BTC output to whoever needs to be paid. Your friend can then add an output for their change address, and use `SIGHASH_ALL` or `SIGHASH_SINGLE` (if their change address is at the same index as their input) and complete the transaction.
- `SIGHASH_ALL | SIGHASH_ANYONECANPAY` - This is again somewhat straightforward. You essentially agree to contribute to a transaction, but only if a list of recipients receives a certain amount of BTC. This is a situation like our `SIGHASH_SINGLE` example, but the rest of the inputs are arbitrary, and the ouputs are fixed.
- `SIGHASH_NONE | SIGHASH_ANYONECANPAY` - This one essentially boils down to "Here's some BTC". It offers no guarantees at all, and can be used as a proof of burn. Such a transaction allows anyone to define an input, and include the input in any other transaction, with arbitrary outputs. By making such a signature, you are fully committing to spending the BTC without any control whatsoever. `SIGHASH_NONE` at least enforces the condition is that everyone else with inputs on the transaction must also move their BTC.
- `SIGHASH_SINGLE | SIGHASH_ANYONECANPAY` - This is a rather interesting combination, and allows for on-chain sales of assets (or coloured coins, in Bitcoin's case). I don't think I could come up with a better example than [this one by Alex Mzrahi](https://groups.google.com/forum/#!msg/bitcoinx/pON4XCIBeV4/IvzwkU8Vch0J){:target="_blank"}. Essentially, this allows Alice, who holds 100 COIN to sign an input authorizing the movement of 100 COIN, and an output to an address she controls with a value of 1 BTC. She then publishes this incomplete and invalid transaction (since there are no inputs for 1 BTC). Anyone who wishes to purchase 100 COIN for 1 BTC can then add an input >= 1 BTC, an output claiming the 100 COIN (remember, Alice's output only claims the BTC), and any change outputs if required. This then makes a complete tx, which can be broadcast to execute the sale trustlessly.

This completes a brief look at the various signature types allowed on Bitcoin. Just one last note. Interestingly, there is a bug in the implementation for `SIGHASH_SINGLE`, first [reported in 2012](https://www.mail-archive.com/bitcoin-development@lists.sourceforge.net/msg01408.html){:target="_blank"}. If the input has a vin that exceeds the number of outputs, it produces a signature that is valid, but not tied to any inputs. Essentially, you end up with a signature that can be used to spend not only the input you were signing, but also any past or future outputs to that address! A pull request ([bitcoin#13360](https://github.com/bitcoin/bitcoin/pull/13360){:target="_blank"}) was opened to make such transactions non-standard was opened just a few days ago, and is what prompted me to look into how all this actually works.