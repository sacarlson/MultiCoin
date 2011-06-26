Bitcoin Escrow

Bitcoin "escrow" are coins that are under the control of multiple
parties.  The initial implementation allows n parties to vote, with
k good signatures needed (k <= n).

This eliminates single points of failure and reduces the trust required
in many transaction use cases.

Use Cases
---------

Escrow: send money to an escrow coin with three parties - sender,
receiver and escrow observer.  Require 2 signatures.  If sender and
receiver agree, they can send the coin back to sender, or to receiver.
If they disagree, the escrow observer can break the tie by signing with
the sender or with the receiver.

Immediate payment: send money from sender to a coin with two parties -
sender and payment observer.  The payment observer will only agree to a
single spend of the money, which prevents double spending.  Of course,
the receiver has to trust the observer.  For protection against observer
failure, additional observers can be added.  TBD: requires escrow change
for leftover funds

Secured funds: to increase security of funds, require 2 out of 3
parties to sign for disbursement.  This reduces the chance that funds
will be stolen because more than one party must be compromised for a
successful theft.  This is also useful for storage at third parties
such as exchanges, to reduce the chance of mass theft.  Instead of the
exchange having full control of the funds, funds can be stored requiring
signatures from 2 out of: account holder, exchange, third party observer.
TBD: requires escrow change for leftover funds

API Calls
---------

sendescrow <escrowaddrs> <amount> [comment] [comment-to]
    <escrowaddrs> is of the form <n>,<addr>,<addr...>
    where <n> of the addresses must sign to redeem the escrow
    <amount> is a real and is rounded to the nearest 0.00000001

redeemescrow <inputtx> <addr> [<txhex>]
    where <inputtx> is the escrow transaction ID
    <addr> is the destination bitcoin address
    <txhex> is a partially signed transaction
    the output is either ['partial', <txhex>] if more signatures are needed
    or ['complete', <txid>] if the transaction was broadcast

example 1:
bitcoind -rpcport=18332 sendescrow '2,mm4YFeg8SGug132beNpSapBguYBu8h95Rh,mqD3wx6FoSX3Xjr4U2EsrAPNwWmCcvxoTs,mn7wuh14df8pN2pJSf6qYEJJ2fhyLTTTUK' 1.23
e232e0055dbdca88bbaa79458683195a0b7c17c5b6c524a8d146721d4d4d652f
secound line here was returned and is the transaction number

seen with listtransactions
  {
        "account" : "",
        "address" : " unknown ",
        "category" : "send",
        "amount" : -1.23000000,
        "fee" : -0.00050000,
        "confirmations" : 4,
        "txid" : "e232e0055dbdca88bbaa79458683195a0b7c17c5b6c524a8d146721d4d4d652f",
        "time" : 1308988438
    },

transaction must be confirmed before you can redeemescrow

bitcoind -rpcport=18332 redeemescrow e232e0055dbdca88bbaa79458683195a0b7c17c5b6c524a8d146721d4d4d652f mqD3wx6FoSX3Xjr4U2EsrAPNwWmCcvxoTs
[
    "complete",
    "a1c8a7c558835f45d9f584934f2602f444efb0467940cf83eadc97199326c909"
]

seen with listtransactions
 {
        "account" : "",
        "address" : "mqD3wx6FoSX3Xjr4U2EsrAPNwWmCcvxoTs",
        "category" : "receive",
        "amount" : 1.22990000,
        "confirmations" : 1,
        "txid" : "a1c8a7c558835f45d9f584934f2602f444efb0467940cf83eadc97199326c909",
        "time" : 1308988950
 }

in the above example all three of the coin addresses were from the same user by getnewaddress 3 times
so it only took one redeemescrow to redeem it to "complete".

example2:
bitcoind -rpcport=18332 sendescrow '3,mm4YFeg8SGug132beNpSapBguYBu8h95Rh,mqD3wx6FoSX3Xjr4U2EsrAPNwWmCcvxoTs,mkNoLd6NWRjSi2aniEbWiytYpBKmCSrFeU' 1.25

mkNoLd6NWRjSi2aniEbWiytYpBKmCSrFeU = address from graybox all others are from bigboy

   {
        "account" : "",
        "address" : " unknown ",
        "category" : "send",
        "amount" : -1.25000000,
        "fee" : -0.00010000,
        "confirmations" : 1,
        "txid" : "00c6e4477f4c522b8caa4b3da665d3eb4072381f38bc00772f22f965e60dfa26",
        "time" : 1308998810
    }


bitcoind -rpcport=18332 redeemescrow 00c6e4477f4c522b8caa4b3da665d3eb4072381f38bc00772f22f965e60dfa26 mwVR3MVvM8uWetb55ekXSPXVPbyzQzrnxq
[
    "partial",
    "010000000126fa0de665f9222f7700bc381f387240ebd365a63d4baa8c2b524c7f47e4c60001000000fd18010000493046022100ed6a544e1c7e6deb2311f6ea039c77e5256be2b736a03f69df504d92e395321c022100f7d87310bfd79fc83d341d37b041e5793e17c9065efc5e0eb9297f8e8600f039014104ce6242d72ee67e867e6f8ec434b95fcb1889c5b485ec3414df407e11194a7ce012eda021b68f1dd124598a9b677d6e7d7c95b1b7347f5c5a08efa628ef0204e147304402203f251646144e7e23cdfd70365a25a8017776206261f07573c5516fe8ca44e158022039442133b2c279915ee67ec3602fb7550a178f9d1bae6c3dda50ee4a430a672a014104ce66c9f5068b715b62cc1622572cd98a08812d8ca01563045263c3e7af6b997e603e8e62041c4eb82dfd386a3412c34c334c34eb3c76fb0e37483fc72323f807ffffffff0130327307000000001976a914af3781eca2ec6fa25b0fbadece761dfd7685306488ac00000000000000000000000000000000000000000000000000000000000000000000000000ffffffff00020b66726f6d6163636f756e7400057370656e74000000000000000000000000"
]

bitcoind -rpcport=18332 redeemescrow 00c6e4477f4c522b8caa4b3da665d3eb4072381f38bc00772f22f965e60dfa26 mwVR3MVvM8uWetb55ekXSPXVPbyzQzrnxq 010000000126fa0de665f9222f7700bc381f387240ebd365a63d4baa8c2b524c7f47e4c60001000000fd1a010000493046022100c3fcfe4ddf94a148638f09fb2d8cc4c4e20e033abc344c23dd2fc2d9415ecb96022100da6cba72e5e6f454294d502eb06fee5668016efe2b91c5560b21cdb50c18eeee014104ce6242d72ee67e867e6f8ec434b95fcb1889c5b485ec3414df407e11194a7ce012eda021b68f1dd124598a9b677d6e7d7c95b1b7347f5c5a08efa628ef0204e149304602210097b8a0386e7da86f85fc9dac08b7db8eea75af899a8933623cd0f001ce68ff46022100e0835cac9f21c519cfe0761058359f20c7366019d503de0685d0eeb5ada7a8c3014104ce66c9f5068b715b62cc1622572cd98a08812d8ca01563045263c3e7af6b997e603e8e62041c4eb82dfd386a3412c34c334c34eb3c76fb0e37483fc72323f807ffffffff0130327307000000001976a914af3781eca2ec6fa25b0fbadece761dfd7685306488ac00000000000000000000000000000000000000000000000000000000000000000000000000ffffffff00020b66726f6d6163636f756e7400057370656e74000000000000000000000000
[
    "partial",
    "010000000126fa0de665f9222f7700bc381f387240ebd365a63d4baa8c2b524c7f47e4c60001000000fd190100004830450220665d3a6810e56a5735f8a30f59b4df9717dec6fe68913f2f8bfe197610e31211022100844fefa9de8b24c4578761ad3cb5da85570d69d3933295fbe738f46117332616014104ce6242d72ee67e867e6f8ec434b95fcb1889c5b485ec3414df407e11194a7ce012eda021b68f1dd124598a9b677d6e7d7c95b1b7347f5c5a08efa628ef0204e1493046022100ba52eb61e4b508b4ff76827b8a7e6980e9335d64620fef0ecef7dba5865a1c94022100ea1c8e170fcf37cafa8a261d74808c21d969b5c746e12790bab9412f2160bff6014104ce66c9f5068b715b62cc1622572cd98a08812d8ca01563045263c3e7af6b997e603e8e62041c4eb82dfd386a3412c34c334c34eb3c76fb0e37483fc72323f807ffffffff0130327307000000001976a914af3781eca2ec6fa25b0fbadece761dfd7685306488ac00000000000000000000000000000000000000000000000000000000000000000000000000ffffffff00020b66726f6d6163636f756e7400057370656e74000000000000000000000000"
]

in the case above again all three were of the same user but I'm not sure how to redeem this one.

I was also able to have a single address to another wallet with signers required 1 and was able to reedeem it 
on the other sytem with the transaction number and address but never saw any transaction on the other system walet 
recieved until after I enter the reedeemescrow.  I also found from the sender side if he tried to redeemescrow it 
only gave your a "partial" state and hex return.


example 3:
bitcoind -rpcport=18332 sendescrow '2,n4VnTQGeVYgvFvDFBP8cVat6Fvi9uCfvzM' .12
error: {"code":-4,"message":"Invalid escrow address format"}

since there is only one address you can't have 2 signers and you get this "Invalid escrow address format" error


example 4:
bitcoind -rpcport=18332 sendescrow '2,n4VnTQGeVYgvFvDFBP8cVat6Fvi9uCfvzM,mkZhsm3KpeFUeTqFjh5LtecHvZSWCTSyza' .12
2df3050037b3289ca97f690867674bfafbe664c6ceee88fae72b910a6508961c

sender side has the address mkZh... and reciever has the address n4Vn...
on sender side listtransactions (nothing is seen on listtransactions on reciever side for n4Vn... address):
    {
        "account" : "",
        "address" : " unknown ",
        "category" : "send",
        "amount" : -0.12000000,
        "fee" : -0.00050000,
        "confirmations" : 0,
        "txid" : "2df3050037b3289ca97f690867674bfafbe664c6ceee88fae72b910a6508961c",
        "time" : 1309051227
    }

