# Carry Over Mechanism

The so called Carry-Over mechanism in zenyt serves these main purposes:

* onboarding of users who already are participants in the crypto world
* a deterministic and fair "airdrop"

The blockchains of several other cryptocurrencies are scanned and the
states of account:balance are extracted for a certain point in
time. These balances are mapped to Zenyt balances based on MarketCap
of the source cryptocurrency.

The amount of Zenyt reserved for this initial carry-over is 1bn. The
complete list of source cryptocurrencies is TBA, but the number of
source addresses is estimated to be around 50 million.

As the private keys for these source addresses are not known and
cannot be known, the mechanism is as follows: Zenyt supports the
signing process for all source cryptocurrencies supported in the carry
over. The public (known) address containing a balance is mapped -
deterministically - to a Zenyt address.

A user of a - say - LTC address with some balance on it, will be able
to look up the Zenyt balance at a RS Address derived from his LTC
address. And with the LTC private key the user will be able to sign an
outgoing transaction, therefore spend the Zenyt!

For addresses unclaimed, the demurrage mechanism will - in time -
ensure their funds flow back into the system.
