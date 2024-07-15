# crypto

Collection of explanations to better understand deeper crypto topics for applied research and development of distributed systems.

## Math Fundamentals

### Elliptic Curve Cryptography

Based on algebraic structure of ellipctic curves over finite fields. Points on the field are denoted by `(x,y)` coordinates.

A group `G` is a set with single operations with the properties identity, associativity, invertibility, and closure.

A field `F` is a set with the two operations addition and multiplication, satisfying the field axioms. An example is `F_p = {0, 1, .., p-1}` which contains multiple groups and is a finite field of integegers modulo `p` (prime number).

Curve bilinear pairings "combine" (through various operations) two cryptographic groups to obtain a third group. A map `t =  G_1 x G_2 -> G_T` has linear properties in first and second argument. Linearity is preserved in both arguments.

Curve pairings help solve more complex problems on the curve. If we have `P = p * G` and `Q = q * G` (where `p` and `q` are scalars, and `G` is a generator point on the curve), we can compute `R = p * q * G`, a point in `G_T`. If `R` and `G` is known for another party, it is computationally very unlikely to be able to retrieve `k = p * q`. See ECDLP.


### Commitment Scheme

A cryptographic protocol that allows one party to commit to a value without revealing it and the ability to do so later. Two fundamental properties: hiding and binding. Hiding ensures that the committed value remains confidential and cannot be inferred from the commitment itself. Binding ensures that once a value is committed, it is immutable (integrity).

A Pedersen commitment consists of two steps:
1. Commit Phase: Alice chooses the desired value `v` and random value `r`. She computes the commitment `C = (g^v)*(h^r)` (where `g` and `h` are generators of a group `G`) and sends it to Bob.
2. Reveal Phase: Alice sends `v` and `r` to Bob and he computes the commitment `C'` to verify that it matches `C`.

From [1]:

A commitment scheme is a pair of algorithms `C(v,r) -> G` and  `Open(v,r,C) -> {true,false}` s.t. `Open(v,r,C(v,r)) accepts for all `(v,r)` in the domain of `G`. Satisfies the properties binding and hiding. A homomorphic commitment scheme allows commitments to `v` and `v'` to be added to obtain a commitment `v + v'` with the same security guarantees.

A Pedersen commitment maps `(v,r)` in the first algorithm above to `vH + rG` and the `Open` algorithms verifies the commitment as previously. This is a computationally binding and perfectly hiding homomorphic commitment scheme.

A range proof on a homomorphic encryption `C` is a proof that the committed value of `C` is within a given range `[a,b]`. Default range is `[0,2^n]` and cannot produce an overflow. It must be a zero-knowledge proof of knowledge of the opening information of the commitments.

From [2]:

In Monero (XMR), the input value spent and the value of the outputs sent are encrypted and opaque to everyone but the recipients of each output. Pedersen commitments allow you to send XMR without revealing the value of the transactions as well as verifying that they are valid and not creating new XMR.

Pedersen commitments mean that the sums can be verified as being equal but the XMR value of each of the sums and each individual XMR input & output value are undeterminable. In addition, the ratios of one input to another is undeterminable.

It is unclear which inputs are being spent as the ring signature lists both the real inputs being spent and decoy (dummy) inputs so you don't know which commitments need to be summed.

From [3]:

In Mimblewimble, and how it differs from Bitcoin. In the latter, you have a key associated with every input and a signature of the whole transaction with that key. Mimblewimble has a multi-signature across all inputs and all outputs. The sum of all the keys on the outputs minus the sum of all the keys on the inputs results in an elliptic curve point which can be treated as a public key. It represents a multi-signature public key for all the transacting parties (all the owners of every input and all the owners of every output).

## Consensus and Mining

### Merged Mining

Every hash calculated contributes to the total hash rate of the other merged mined chain(s).

Mining two or more chains at the same time, also known as Auxiliary PoW (AuxPoW), such that one chain can trust the other's work as their own and accept AuxPoW blocks [4]. Work done on the main chain (the parent) can also be used to check for validity (a solution) on the auxiliary chain (the child). Merged mining specifically targets Bitcoin like PoW systems (for example: Bitcoin-Namecoin, Monero-Tari).

A miner that is merge mining must continuously assemble blocks for both chains (including transaction sets). If the miner finds a solution for the parent chain (with higher difficulty than the child chain) it is valid for both and the block is broadcast on the parent chain. If the solution only satisfies the child chain, it is broadcast there. 

In the example of merge mining Bitcoin with Namecoin, if solving a BTC block, only an extra hash is required in the coinbase transaction's `ScriptSig` field. For the Namecoin block, it would need a reference to the Bitcoin block through its `AuxPow` data structure which contains comparatively more overhead compared to what is store in the Bitcoin ledger.

Example of an AuxPoW block on the Namecoin network where the shaded fields are there by default (when not merge mining):

[!namecoin_block_with_aux_pow](./assets/btc_nmc_merge_mining.png)


## References

[1] A. Poelstra, _Mimblewimble_, 2016

[2] Moneropedia, https://www.getmonero.org/resources/moneropedia/pedersen-commitment.html

[3] MimbleWimble with Andrew Poelstra, https://www.youtube.com/watch?v=aHTRlbCaUyM

[4] Merged mining specification, https://en.bitcoin.it/wiki/Merged_mining_specification
