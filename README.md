# Prodefi: prodefi-ring-contract

Smart contract to perrofm linkable ring signature using Ethereum for transaction privacy.

## Introduction

Prodefi is a Smart Contract that runs on Ethereum that offers trustless autonomous tumbling using linkable ring signatures.

**This proof of concept is still evolving and comes with the caveat that it should not be used for anything other than a technology demonstration.**


## White Paper

// TO-DO [Prodefi Ring Contract, Dhanraj Dadhich][1]


## Using Prodefi

:point_right: **See the full [Tutorial](./prodefiTutorial.md)**

-----------------------------------------------------

Prodefi supports ether and ERC20 compatible token transactions.
In order to use the Mixer with ERC20 compatible tokens, the `DepositERC20Compatible` and `WithdrawERC20Compatible` functions must be used.
However, in order to do an ether transaction, one has to use the `DepositEther` and `WithdrawEther` functions.

-----------------------------------------------------

To tumble a token it is deposited into the [Mixer](contracts/Mixer.sol) smart contract by sending the token and the stealth public key of the receiver to the `Deposit` method.

The Mixer contract places the token into an unfilled [Ring](contracts/LinkableRing.sol) specific to that token and denomination and provides the GUID of the Ring. The current ring size is 4, so when 3 other people deposit the same denomination of token into the Mixer the Ring will have filled. Tokens can only be withdrawn when the Ring is full.

The receiver then generates a linkable ring signature using their stealth private key, this signature and the Ring GUID is provided to the `Withdraw` method in exchange for the token.

The lifecycle and state of the Mixer and Rings is monitored using the following Events:

 * `LogMixerDeposit` - Tokens have been deposited into a Ring, includes: Ring GUID, Receiver Public Key, Token, Value
 * `LogMixerReady` - Withdrawals can be now me made, includes: Ring GUID, Signing Message
 * `LogMixerWithdraw` - Tokens have been withdrawn from a Ring, includes: Ring GUID, Tag, Token, Value
 * `LogMixerDead` - All tokens have been withdrawn from a Ring, includes: Ring GUID

The [Aploune](https://github.com/Prodefi/aploune) tool can be used to create the necessary keys and to create and verify compatible ring signatures, for details see the [Aploune Integration Tests](test/aploune.js).


## Caveats

 * #34 - Gas payer exposes sender/receiver
 * #22 - Only Ether is presently supported
 * #32 - Tokens are locked into the Ring until it's filled
 * #12 - Withdraw messages can be replayed


## Gas Usage

Despite being an improvement over the previous iteration which used a Solidity P256k1 implementation, the new alt_bn128 opcodes are still expensive and there are many improvements which can be made to reduce these costs further. If you have any interesting optimisations or solutions to remove storage and memory operations please open an issue.

Currently the Gas usage is:

| Function | Avg     |
| -------- | ------- |
| Deposit  | 150k    |
| Withdraw | 725k    |

## Developing

[Truffle][2] is used to develop and test the Prodefi Smart Contract. This has a dependency on [Node.js][3]. [solidity-coverage][7] provides code coverage metrics.

Prerequisites:

[yarn][4] needs to be installed (but [npm][5] should work just as well).

    yarn install

This will install all the required packages.

Start `testrpc` in a separate terminal tab or window.

    yarn testrpc
    
    # in separate window or tab
    yarn test

This will compile the contract, deploy to the Ganache instance and run the tests. 


#### Testing with Aploune

The [Aploune][6] tool is needed to generate the signatures and random keys for some of the tests. If `aploune` is in `$PATH` the `yarn test` command will run additional tests which verify the functionality of the Mixer contract using randomly generated keys instead of the fixed test cases.

[1]: https://prodefi.io/whitepaper/prodefi-ring-contract.pdf
[2]: http://truffleframework.com/
[3]: https://nodejs.org/
[4]: https://yarnpkg.com/en/docs/install
[5]: https://docs.npmjs.com/getting-started/installing-node
[6]: https://github.com/Prodefi/aploune
[7]: https://www.npmjs.com/package/solidity-coverage
