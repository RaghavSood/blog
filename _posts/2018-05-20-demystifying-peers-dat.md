---
layout: post
title: Demystifying Bitcoin's peers.dat
categories:
- blog
---

Bitcoin uses a custom format to store peer information. Although the inbuilt JSON-RPC provides a helpful `getpeerinfo` method to list your active connections, it offers no method to query, dump, or otherwise access the information in peers.dat, which contains far more than just your active connections. Having access to the information in this file can be helpful for a number of reasons, such as finding out information about the network and finding more nodes than just your connections to broadcast transactions to.

My interest in this was piqued by [this](https://bitcoin.stackexchange.com/q/75324/7272){:target="_blank"} post on the [Bitcoin StackExchange](https://bitcoin.stackexchange.com/){:target="_blank"}. This blog post is an attempt to answer that question (and cover some gaps in my personal crypto tools) by building a utility that can read and query peers.dat. The post will go step by step, as I am writing this while building the utility.

----

## Research

To work with a minimal example, I deleted my existing peers.dat and ran `bitcoind` again. This gives me a much lighter file (11 KB), instead of a 4MB+ one that contained peer info over many months. I then open up the peers.dat file in sublime. This gives us a bunch of hex, no surprises there. When dealing with hex, `hexdump` is step 1, so let's see what we get:

{% highlight shell %}
$ hexdump -C peers.dat | head -n 17
00000000  f9 be b4 d9 01 20 91 41  99 be 39 46 d6 2c 9f b3  |..... .A..9F.,..|
00000010  e6 80 ef db 3c 1d 64 52  7c 18 c4 3e 0b eb 23 9b  |....<.dR|..>..#.|
00000020  59 46 e3 79 40 f8 6b 00  00 00 01 00 00 00 00 04  |YF.y@.k.........|
00000030  00 40 34 fc 01 00 85 55  f7 5a 09 00 00 00 00 00  |.@4....U.Z......|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 ff ff 05 09  |................|
00000050  8b 05 20 8d 00 00 00 00  00 00 00 00 00 00 ff ff  |.. .............|
00000060  2d 20 82 13 00 00 00 00  00 00 00 00 00 00 00 00  |- ..............|
00000070  34 fc 01 00 08 c2 f9 5a  09 00 00 00 00 00 00 00  |4......Z........|
00000080  00 00 00 00 00 00 00 00  00 00 ff ff 31 49 ae 91  |............1I..|
00000090  20 8d 00 00 00 00 00 00  00 00 00 00 ff ff 2d 20  | .............- |
000000a0  82 13 00 00 00 00 00 00  00 00 00 00 00 00 34 fc  |..............4.|
000000b0  01 00 24 34 f8 5a 09 00  00 00 00 00 00 00 00 00  |..$4.Z..........|
000000c0  00 00 00 00 00 00 00 00  ff ff 25 23 b7 0a 20 8d  |..........%#.. .|
000000d0  00 00 00 00 00 00 00 00  00 00 ff ff 2d 20 82 13  |............- ..|
000000e0  00 00 00 00 00 00 00 00  00 00 00 00 34 fc 01 00  |............4...|
000000f0  b7 84 fa 5a 09 00 00 00  00 00 00 00 00 00 00 00  |...Z............|
00000100  00 00 00 00 00 00 ff ff  b4 6b 56 4d 20 8d 00 00  |.........kVM ...|
{% endhighlight %}

This is a truncated output, as the file continues much the same way.

The hex immediately gives us some clues. The first four bytes (`f9 be b4 d9`) stand out as the [message start string](https://github.com/bitcoin/bitcoin/blob/21f56805531c4b2954d0aaffa34c83f37ed8f9c0/src/chainparams.cpp#L115){:target="_blank"}, which is something you see very often when working with bitcoin's network level implementations. Based on some past projects with Bitcoin, these bytes are ingrained into my head and stood out immediately.

The next set of bytes after the message start doesn't offer any immediate insight, so let's skip ahead for the moment. The next thing that does stand out is a repeated pattern of `ff ff`,  followed by four bytes, followed by `20 8d`. The first two don't seem to give out any information (`ff ff` would be an unlikely candidate for magic bytes, so is unlikely to be a marker in the file). However, the next two (`20 8d`) do give us some useful information. When [converted to decimal](http://www.hexadecimaldictionary.com/hexadecimal/0x208D/){:target="_blank"}, we get `8333`, which is the default port number for bitcoin.

This new piece of information updates our previous pattern to `ff ff` + four bytes + port number (`8333`). IPv4 addresses use a 32-bit address space, which is four bytes. It stands to reason that the four bytes before the port number are an IP address, considering this file is meant to hold peer info.

This is easily verified by taking the first instance of this pattern's middle four bytes (`05 09 8b 05`) and converting them to an IP address. I wrote a quick [Go script](https://play.golang.org/p/MMOucH-oP6P){:target="_blank"} for this, as it is an operation I will likely be doing in the utility as well. The script gives us `5.9.139.5`, which seems like a plausible IP address. Running the script with a few other bytes from other patterns also produces valid IPs, so I'm pretty sure at this point this is the correct interpretation.

This seems like all the information I'm going to get just by looking at the hex above, so the next step is to look at the hexdump for the bottom of the file:

{% highlight shell %}
$ hexdump -C -v peers.dat | tail -n 17
00002b30  00 00 4f 00 00 00 00 00  00 00 00 00 00 00 00 00  |..O.............|
00002b40  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00002b50  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00002b60  00 00 00 00 00 00 02 00  00 00 53 00 00 00 61 00  |..........S...a.|
00002b70  00 00 00 00 00 00 00 00  00 00 01 00 00 00 21 00  |..............!.|
00002b80  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00002b90  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00002ba0  00 00 00 00 00 00 01 00  00 00 69 00 00 00 00 00  |..........i.....|
00002bb0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00002bc0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00002bd0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00002be0  00 00 00 00 00 00 01 00  00 00 2c 00 00 00 00 00  |..........,.....|
00002bf0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00002c00  00 00 00 00 00 00 86 be  3d 3b b8 dd 01 93 f1 79  |........=;.....y|
00002c10  f9 c9 d1 ff f5 d4 cb 38  4d ab 56 25 62 d0 c8 d7  |.......8M.V%b...|
00002c20  d8 82 c1 4c c1 2c                                 |...L.,|
{% endhighlight %}

Nothing really sticks out except the last few bytes. Counting these shows that there is 32 bytes of continous non-zero data. This is very likely a hash, and being at the very end of the file, it's almost certainly a checksum

With the initial hex analysis not providing enough information to completely decode the file, it's time to head to the Bitcoin source code. Some quick searches on GitHub reveal that peers.dat is managed by [addrdb.cpp](https://github.com/bitcoin/bitcoin/blob/8b4081a889b35cac3ccafa4f7109c72ccb087518/src/addrdb.cpp){:target="_blank"}

Browsing through addrdb.cpp shows that the initial assumptions about the message start and checksum hash were correct:

{% highlight cpp %}
bool SerializeDB(Stream& stream, const Data& data)
{
    ...
        CHashWriter hasher(SER_DISK, CLIENT_VERSION);
        stream << Params().MessageStart() << data;
        hasher << Params().MessageStart() << data;
        stream << hasher.GetHash();
    ...
}

{% endhighlight %}

This snippet shows that the checksum algorithm is the same as the regular Bitcoin hashing algorithm (since it relies on [hash.h](https://github.com/bitcoin/bitcoin/blob/172f5fa738d419efda99542e2ad2a0f4db5be580/src/hash.h){:target="_blank"}), which is a double-sha256. We can verify this quickly with bash and openssl:

{% highlight shell %}
$ head -c -32 peers.dat | openssl dgst -sha256 -binary | openssl dgst -sha256
(stdin)= 86be3d3bb8dd0193f179f9c9d1fff5d4cb384dab562562d0c8d7d882c14cc12c
{% endhighlight %}

We strip the last 32 bytes from the file (which contain the checksum), and pass the rest to openssl for hashing twice. Do note that this requires a GNU version of `head`, as OS X will complain about a negative byte number. We can see that the result matches what we see in the hexdump, so that's definitely the checksum.

Now all that's needed is to determine the data format, and the rest of the header following the message start bytes. Switching from addrdb.cpp to [addrdb.h](https://github.com/bitcoin/bitcoin/blob/e1d6e2af6d89935f6edf027e5d4ea1d2ec6c7f41/src/addrman.h){:target="_blank"} leads us to a [helpful comment](https://github.com/bitcoin/bitcoin/blob/e1d6e2af6d89935f6edf027e5d4ea1d2ec6c7f41/src/addrman.h#L287){:target="_blank"} that lays out the serialized header format for us.

Lastly, some more GitHub search magic leads to [chainparamsseeds.h](https://github.com/bitcoin/bitcoin/blob/108af52ef75a466be71d04bb973b794eca17e212/src/chainparamsseeds.h){:target="_blank"}. This states that:

 > Each line contains a 16-byte IPv6 address and a port.  
 > IPv4 as well as onion addresses are wrapped inside an IPv6 address accordingly.

This explains the whole lot of `00` in the hexdump preceding the IP, as well as the `ff ff`. Since the majority of connections still default to IPv4, what we are seeing is IPv4 addresses encoded as IPv6 addresses, which [follows the format](https://en.wikipedia.org/wiki/IPv6#IPv4-mapped_IPv6_addresses){:target="_blank"} `::ffff:IPv4-address`, which is where the `ff ff` comes from.


## File Structure

Based on the above research, a flow of how peers.dat is generated and structured can be put together.

The snippet from `addrdb.cpp` shared above is the starting point for the creation of `peers.dat`. This snippet gave us the base structure as 

{% highlight shell %}
peers.dat
├── header
├── data
└── checksum
{% endhighlight %}

The file header consists of:

 - A 4-byte magic value which is the message start defined in chainparams.cpp.
 - 1 version byte, always `0x01`
 - 1-byte defining the key length for nkey, always `0x20` (32 in decimal)
 - Key length number of bytes that specify the nkey, `0x20` from above
 - 4 bytes specifying how many new addresses there are
 - 4 bytes specifying how many tried addresses there are
 - 4 bytes specifying how many buckets there are (XOR'd against `2**30`)

 In all, this gives us a 50-byte header for peers.dat, which can be encapsulated in the following Go struct:

{% highlight go %}
type PeersDB struct {
	Path         string
	MessageBytes []byte // 0  : 4
	Version      uint8  // 4  : 4
	KeySize      uint8  // 5  : 5
	NKey         []byte // 37 : 32
	NNew         uint32 // 41 : 4
	NTried       uint32 // 45 : 4
	NewBuckets   uint32 // 49 : 4
} 
{% endhighlight %}

This expands our structure to:

{% highlight shell %}
peers.dat
└── header
    ├── MessageBytes
    ├── Version
    ├── KeySize
    ├── NKey
    ├── NNew
    ├── NTried
    └── NewBuckets
├── data
├── checksum
{% endhighlight %}

The data part of the file is simply the peer info structure repeated `NNew + NTried` times (i.e. once per peer). The structure of this can be pieced together from three files. `CAddrInfo` from [addrman.h](https://github.com/bitcoin/bitcoin/blob/0a8054e7cd5c76d01e4ac7234e3883d05f6f5fdd/src/addrman.h#L61){:target="_blank"} encapsulates the information regarding a single peer as follows:

{% highlight cpp %}
inline void SerializationOp(Stream& s, Operation ser_action) {
    READWRITEAS(CAddress, *this);
    READWRITE(source);
    READWRITE(nLastSuccess);
    READWRITE(nAttempts);
}
{% endhighlight %}

We will try to keep our Go code in sync with Bitcoin Core's code to allow for easier updates in the event of changes.

The `PeersDB` struct is updated to include `CAddrInfo` slices for New and Tried peers, becoming:

{% highlight go %}
type PeersDB struct {
	Path          string
	MessageBytes  []byte // 0  : 4
	Version       uint8  // 4  : 4
	KeySize       uint8  // 5  : 5
	NKey          []byte // 37 : 32
	NNew          uint32 // 41 : 4
	NTried        uint32 // 45 : 4
	NewBuckets    uint32 // 49 : 4
	NewAddrInfo   []CAddrInfo
	TriedAddrInfo []CAddrInfo
}
{% endhighlight %}

The `CAddrInfo` struct simply mimics the data serlialized in it:

{% highlight go %}
type CAddrInfo struct {
	Address     CAddress
	Source      []byte
	LastSuccess uint64
	Attempts    uint32
}
{% endhighlight %}

Looking into the code for `CAddress` in [protocol.h](https://github.com/bitcoin/bitcoin/blob/5c2aff8d95a932f82a9472975b1d183da6c99e5f/src/protocol.h#L339){:target="_blank"}, we can assemble a `CAddress` struct as follows:

{% highlight go %}
type CAddress struct {
	SerializationVersion []byte
	Time                 uint32
	ServiceFlags         []byte
	PeerAddress          CService
}
{% endhighlight %}

Finally, `CService` is found in [netaddress.h](https://github.com/bitcoin/bitcoin/blob/c19986940869034cb15e684f014f48fe00e75778/src/netaddress.h#L169){:target="_blank"}, and converts to a very simple struct:

{% highlight go %}
type CService struct {
	IPAddress []byte
	Port      uint16 // This is serialized as BigEndian
}
{% endhighlight %}

These simple structs, chained together, give us an overall structure for `peers.dat`. In a comprehensive tree, it looks something like:

{% highlight shell %}
peers.dat
└── header
    ├── MessageBytes
    ├── Version
    ├── KeySize
    ├── NKey
    ├── NNew
    ├── NTried
    └── NewBuckets
└── data
    └── repeated
        └── CAddrInfo
            ├── CAddress
                ├── SerializationVersion
                ├── Time
                ├── ServiceFlags
                ├── CService
                    ├── IPAddress
                    └── Port
            ├── Source
            ├── LastSuccess
            └── Attempts
└── checksum
{% endhighlight %}

With the structure of the file all figured out, we can move on to writing the utility to parse it. At this point, the utility itself is quite simple - simply start from the beginning, read the header, loop until `NNew + NTried` entries of `CAddrInfo` have been read, and then dump the output in the format requested. Instead of putting the boring code in this post, I invite you to visit the [bitpeers](https://github.com/RaghavSood/bitpeers){:target="_blank"} repo.

That said, there is still a chunk of the file after the last tried address before the checksum that is not decoded. As far as I can tell, it does not contain any peer information. Instead, I suspect it is some kind of per-peer/per-bucket integrity check, preventing an attacker from changing all the peers to a set they control. I may work on decoding that in the future, and will update this post if I do.

## Using bitpeers

Installing `bitpeers` is quite simple, provided you have go installed, and your `GOPATH` set up. If not, go has a [handy getting started guide](https://golang.org/doc/install){:target="_blank"} which you can use to fix that.

Once go is set up, simply run

{% highlight shell %}
go get -u github.com/RaghavSood/bitpeers/cmd/bitpeers
{% endhighlight %}

If your go environment is set up properly, you should now have a `bitpeers` command available. If not, try finding your `GOBIN` (`GOPATH/bin`) and adding it to your `PATH`.

`bitpeers` to easily dump `peers.dat` addresses as either human-readable plaintext or JSON. It accepts three flags:

{% highlight shell %}
Usage of bitpeers:
      --addressonly       outputs only addresses if specified
      --filepath string   the path to peers.dat
      --format string     the output format {json|text} (default "json")
{% endhighlight %}

Running `bitpeers --filepath /mnt/doge/.dogecoin/peers.dat --addressonly` will produce a JSON array of all the IPs and ports in `peers.dat`. You can also pass the `--format text` option to produce a list of all IPs and ports, one IP:port per line.

Running without the `--addressonly` option will produce the full JSON/text output, which contains the following:

{% highlight shell %}
$ bitpeers --filepath ./peers.dat --format text
SerializationVersion: 34fc0100
Time: 1526192792
ServiceFlags: 0x000000000000000d
IP: 42.5.143.180:8333
Source: 8.8.8.8
LastSuccess: 1526746622
Attempts: 0
{% endhighlight %}

If you happen to find any inconsistencies or issues with the parser or output, please open an issue on the [GitHub repo](https://github.com/RaghavSood/bitpeers){:target="_blank"}.