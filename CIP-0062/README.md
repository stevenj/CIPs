---
CIP: 62
Title: Cardano dApp-Connector Governance extension
Authors: Bruno Martins <bruno.martins@iohk.io>, Steven Johnson <steven.johnson@iohk.io>
Status: Draft
Type: Standards
Created: 2021-06-11
License: CC-BY-4.0
---

# Abstract

This document describes an interface between webpage/web-based stacks and Cardano wallets. This specification defines the API of the javascript object that needs to be injected into web applications to support governance features.

These definitions extend [CIP-30 (Cardano dApp-Wallet Web Bridge)](https://cips.cardano.org/cips/cip30/) to provide specific support for Catalyst vote delegation and vote signing.

# Motivation

Cardano uses a sidechain ("Jormungandr") for its treasury system ("Catalyst") and for other voting purposes. To be able to participate on the sidechain users associate their mainnet staked ADA with a sidechain "voting key". This association happens on Cardano mainnet via metadata attached to a "registration" transaction. [CIP-15 (Catalyst Registration Transaction Metadata Format)](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0015) defined the initial metadata standard for registration transactions, being the only standard up to Catalyst's Fund 9.

[CIP-36 (Catalyst/Voltaire Registration Transaction Metadata Format - Updated)](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0036) introduced a renewed metadata standard, accompanied by a defined voting key derivation path. This standard brings a suite of benefits, namely; vote delegation to either private or public representatives (Catalyst dReps), splitting or combining of private votes, the use of different voting keys or delegations for different purposes.

The goal for this CIP is to extend the dApp-Wallet web bridge to enable the construction of transactions containing metadata that conforms to
[CIP-36](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0036) specification. This allows for the creation of governance centric dApps, offering greater functionality to users of Cardano governance/Catalyst.

## Use Cases

### Catalyst

Current iterations of Cardano's Project Catalyst offers users a cumbersome experience, requiring wallets to generate new mnemoincs from which a Ed25519 key pair is derived to be used for governance. Private voting keys are encoded into a QR codes and passed to a mobile application, where voting is performed. This process required that voting keys could not be deterministically derived from a wallet's base mnemonic.

This specification supports the creation of Catalyst voting dApps, where wallets can connect, share voting key information, register and or delegate. This offers a vastly improved user experience, greatly reducing the number of steps required by the user to engage with Catalyst.

Beyond improved user experience, this CIP enables the functionality benefits offered by the [CIP-36](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0036) metadata standard. Primarily, the ability to delegate one's voting rights to one or many other voting keys in varying proportions. This allows for the creation of Catalyst dReps, members of the community who advertise their voting keys publicly to attract other's voting power via delegation.

### Beyond Catalyst

This CIP not only benefits Cardano's Project Catalyst, but it allows other projects to follow Catalyst's design to implement governance procedures on top of Cardano. Such projects are able to utilize this API by stating their claim to a unique [voting purpose](#votingpurpose). The notion of voting purpose from [CIP-36](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0036) allows differentiation from Catalyst for other project's governance needs.

# Specification

## Version

The API extension specified in this document will count as version `0.2.0` for version-checking purposes below.

## Data Types

### PublicKey

TODO: this

### GovernanceKey

```ts
type GovernanceKey = {
  votingKey: string,
  weight: number
}
```

* votingKey - A hex-encoded string representing a 32 byte Ed25519 public key as described in [CIP-36](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0036#delegation-format).
* weight - Used to calculate the proportion of voting power, using the rules described
  in [CIP-36](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0036#delegation-format).

### VotingPurpose

```ts
type enum VotingPurpose = {
  CATALYST = 0
}
```

* VotingPurpose - Defines the voting purpose of the governance functions. This is used
to limit the scope of the governance functions. For example, a voting purpose
might denote a council election, or some private purpose (agreed by convention). Currently, only voting purpose 0 (zero) is defined, and it is for Catalyst.

**IMPORTANT**: The list of purposes in this specification should be considered the
authoritative list of known purposes, subject to future amendment. Other voting
purposes will be defined as required, by either an update to this CIP or a
future CIP listing currently allocated voting purposes.

### BlockDate

```ts
interface BlockDate {
  epoch: number,
  slot: number
}
```

* epoch - an epoch value on the voting blockchain.
* slot - a block slot number on the voting blockchain.

### Proposal

```ts
interface Proposal {
  votePlanId: string,
  voteOptions: number[],
  votePublic: boolean
}
```
A Catalyst proposal.

* votePlanId - Hex-encoded string representing the vote plan unique identifier. This will be the same as anchored on the voting blockchain.
* voteOptions - List of possible "choices" which can be made for this proposal.
* votePublic - Boolean indicating whether the vote is public or private.

### Vote

```ts
interface Vote {
  proposal: Proposal,
  choice: number,
  expiration: BlockDate,
  purpose: VotingPurpose,
  spendingCounter: number
}
```

An individual raw unsigned vote record.

* proposal - The [`Proposal`](#proposal) being voted on.
* choice - The choice, this value must match one of the available `voteOptions` in the `proposal` element. An `UnknownChoiceError` should be thrown if the value is not one of the valid `voteOptions` of the `proposal`.
* expiration - Voting blockchain epoch \& slot for when the vote will expire. This value is supplied and maintained by the dApp, and forms a necessary component of a [Vote](#vote).
* purpose - The [`VotingPurpose`](#votingpurpose) being voted on.
* spendingCounter - The spending counter is used to prevent double voting. The spending counter for the vote transaction will be supplied and maintained by the dApp. The dApp will manage supplying this in the correct order, thus this should not be enforced by the wallet.

See [Jormungandr Voting](<https://input-output-hk.github.io/jormungandr/jcli/vote.html#voting>) for more information on the data required to construct votes.

## Delegation

```ts
interface Delegation {
    delegations: GovernanceKey[],
    purpose: VotingPurpose
}
```

The record of a voter's delegation.

* delegations - List of [`GovernanceKeys`](#governancekey) denoting the `votingKey` and `weight` of each delegation. As described within [CIP-36](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0036#motivation), we make no distinction between delegations to one's own voting key (registration) or another's voting key. 
* purpose - The associated [`VotingPurpose`](#voting-purpose). A voter may have multiple active delegations for different purposes, but only one active delegation per unique `VotingPurpose`.

## DelegatedCertificate

```ts
interface DelegatedCertificate {
  delegations: GovernanceKey[],
  stakingPub: string,
  rewardAddress: string,
  nonce: number,
  purpose: VotingPurpose
}
```

See [CIP-36](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0036) for an explanation of these fields. **Note:** this object is not in exactly the same format as CIP-36, but the information it contains is the same.

## SignedDelegationMetadata

```ts
interface SignedDelegationMetadata {
  certificate: DelegatedCertificate,
  signature: string,
  txHash: string
}
```

* certificate - The [`DelegatedCertificate`](#delegatedcertificate) that has been signed by the wallet.
* signature - The signature on the delegation certificate, as described in [CIP-36](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0036#example---registration).
* txHash - The transaction hash of the submitted transaction which contained the certificate.

## Error Types

### Extended APIError

```ts
APIErrorCode {
  UnsupportedVotingPurpose: -100,
  InvalidArgumentError: -101,
  UnknownChoiceError: -102,
  InvalidBlockDateError: -103,
  InvalidVotePlanError: -104,
  InvalidVoteOptionError: -105
}
```

```ts
APIError {
  code: APIErrorCode,
  info: string,
  votingPurpose: Purpose[]
}
```

These are the extended API error codes used by this specification. See
[CIP-30 Errors](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0030#apierror) for the standard
API error codes, which continue to be valid for this API extension.

* UnsupportedVotingPurpose - The wallet does not support one of the requested
  voting purposes. The [`votingPurpose`](#votingpurpose) element will be present in the `APIError` object and will list all the voting purposes requested, which are unsupported by the wallet.

  Eg: If voting purposes 0, 5 and 9 were requested and only 0 was supported by the
  wallet. Then the error will be:

  ```ts
  {
    code: -100,
    info: "Unsupported Voting Purpose 5 & 9",
    votingPurpose: [5,9]
  }
  ```

* InvalidArgumentError - Generic error for errors in the formatting of the arguments.
* UnknownChoiceError - If a `choice` is not within the
  [`Proposal`](#proposal)'s `voteOptions`.
* InvalidBlockDateError - If a [`BlockDate`](#blockdate) is invalid.
* InvalidVotePlanError - If the `votePlanId` is not a valid vote plan.
* InvalidVoteOptionError - If the `index` is not a valid vote option.

#### APIError Elements

* code - The `APIErrorCode` which is being reported.
* info - A human readable description of the error.
* votingPurpose (*OPTIONAL*) - Only present if the error relates to a voting purpose, and will list all the `VotingPurpose`s which are involved in the error. See the individual error code descriptions for more information.
* rejectedVotes (*OPTIONAL*) - In a voting transaction, there may be multiple votes being signed simultaneously.

### Extended TxSignError

```ts
TxSignErrorCode {
  ProofGeneration: 1,
  UserDeclined: 2,
  VoteRejected: 3,
}
```

```ts
type TxSignError = {
  code: TxSignErrorCode,
  info: String,
  rejectedVotes: number[]
}
```

All `TxSignErrors` defined in [CIP-30](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0030#txsignerror) are unchanged.

* UserDeclined - Raised when the user declined to sign the entire transaction, in the case of a vote submission, this would be returned if the user declined to sign all of the votes.
* VoteRejected - On a vote transaction, where there may be multiple votes.  If the user accepted some votes, but rejected others, then this error is raised, and `rejectedVotes` is present in the error instance.

## Governance Extension to CIP-30

### cardano.{walletName}.governance.apiVersion: String

The version number of the Governance Extension API that the wallet supports.

### cardano.{walletName}.governance.enable(purpose: VotingPurpose[]): Promise\<API>

Errors: [`APIError`](#extended-apierror)

The `cardano.{walletName}.governance.enable()` method is used to enable the
governance API. It should request permission from the Wallet to enable the API for the requested voting purposes.

If permission is granted, the rest of the API will be available. The Wallet
should maintain a specific whitelist of allowed clients and voting purposes for
this API. This whitelist can be used to avoid asking for permission every time.

This API, being an extension of
[CIP-30](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0030),
expects that `cardano.{walletName}.enable()` to be enabled and added to CIP-30
whitelist implicitly when this API is enabled.

When both this API and [CIP-30](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0030) are being
enabled, it is up to the Wallet to decide the number of prompts requesting
permissions to be displayed to the user.

* `purpose` - This is a list of `VotingPurposes` that the dApp is advising that it will be using on the API. The Wallet must respond with an `UnsupportedVotingPurpose` APIError if the purpose is not supported by the Wallet.

  **Note**: Currently only Voting Purpose 0 (Catalyst) is defined. The wallet should reject any other Voting Purpose requested.

### Returns

Upon successful connection via
[`cardano.{walletName}.governance.enable()`](#cardanowalletnamegovernanceenablepurpose-votingpurpose-promiseapi),
a javascript object we refer to as `API` (type) / `api` (instance) is
returned to the dApp with the following methods.

## Governance API

All methods (all but those requiring signing functionality) should not require any user
interaction as the user has already consented to the dApp reading information
about the wallet's governance state when they agreed to
[`cardano.{walletName}.governance.enable()`](#cardanowalletnamegovernanceenablepurpose-votingpurpose-promiseapi).
The remaining methods
[`api.signVotes()`](#apisignvotesvotes-vote-promisebytes) and
[`api.submitDelegation()`](#apisubmitdelegationdelegation-delegation-promisesigneddelegationmetadata)
must request the user's consent in an informative way for each and every API
call in order to maintain security.

The API chosen here is the minimum API necessary for dApp, wallet
interactions without convenience functions that don't strictly need the wallet's
state to work.

## api.signVotes(votes: Vote[]): Promise\<Bytes>[]

Errors: [`APIError`](#extended-apierror), [`TxSignError`](#extended-txsignerror)

* `votes` - An array of up to 10 votes to be validated with the wallet user, and if valid, signed.

If the wallet user declines some of the votes, a [`TxSignError`](#extended-txsignerror) should be raised with `code` set to `VoteRejected` and the optional `rejectedVotes` array specifying the votes rejected.

However, if the wallet user declines the entire set of votes, the wallet should raise a [`TxSignError`](#extended-txsignerror) with the `code` set to `UserDeclined`.

In either case, where the user declines to sign at least 1 of the votes, no signed votes are returned.

### Returns

`Bytes[]` - An array of the hex-encoded strings of the fully encoded and signed vote transactions. The dApp will submit these votes on behalf of the wallet to the voting blockchain.

## api.getVotingKey(): Promise\<cbor<PublicKey\>\>

Should return the voting [`PublicKey`](#publickey). The wallet should use `address_index` of 0 and return the public key for that index.
### Returns
CBOR hex-encoded representation of the voting `PublicKey`.

## api.submitDelegation(delegation: Delegation): Promise\<SignedDelegationMetadata>

Errors: [`APIError`](#extended-apierror),
[`TxSendError`](https://cips.cardano.org/cips/cip30/#txsenderror),
[`TxSignError`](https://cips.cardano.org/cips/cip30/#txsignerror)

This endpoint should construct the CBOR encoded delegation certificate according to the specs in [CIP-36 Example](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0036#example---registration).

The wallet should then sign the certificate with the private staking key as described in the same example as above.

The implementation of this endpoint can make use of the already existing [CIP-30](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0030#full-api) `api.signData` and `api.submitTx` to perform the broadcasting of the transaction containing the metadata.

Upon submission of the transaction containing the delegation certificate as part of metadata, the Wallet should store the delegation in its local storage and return an object that contains the delegation certificate, the signature and the hash of the transaction that the certificate was submitted with.

* `delegation` - The voter registration [`delegation`](#delegation) record.

This should be a call that implicitly CBOR encodes the `delegation` object and uses the already existing [CIP-30](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0030) `api.submitTx` to submit the transaction. The resulting transaction hash should be returned.

This should trigger a request to the user of the wallet to approve the signing of the transaction.

### Returns

The [SignedDelegationMetadata](#signeddelegationmetadata) of the voter delegation passed in the `delegation` parameter.

# Rationale

## Extension

It was decided to propose this as a new CIP rather than a change to [CIP-30](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0030), so that this was clearly optional (and could be referred to independently). Going forwards, we would expect various kinds of (possibly incompatible) dApp support.

## API Connection

A competing design option was to extend the existing enable function (in [CIP-30](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0030)) to take in a list of CIP APIs (30, 62, etc.) and have the wallet return back the list of accepted APIs. This was not pursued as individual `.enable()` functions allow for custom payloads to be passed at time of connection. This payload flexibility allows for greater variation in dApp connections; allowing for acceptance of part of APIs.

## Voting Key Derivation

Past iterations of this specification proposed three functions for voting key manipulation; get all used voting keys, rotate current voting key and get the current voting key. Discussions led to the removal of these functions, replacing them with `.getVotingKey()`. It was decided that such key control should be decided by wallets, not dApps. Furthermore the introduction of regular voting key rotation would add significant complexities without substantial utility gained.

## Delegation Certificate Flow

A competing design stipulated two separate functions for delegation flow, one to generate delegation certificates and the second to submit the delegation to the chain within a transaction. Having two functions gives the advantage that one wallet can create the delegation certificate whilst another is able to package this with a signature into a transaction. This was ultimately decided as undesirable since it would likely be a rare use case. Furthermore, this design would introduce significant complexity for hardware wallets users doubling their signing burden.

Discussion was had around if dApps should be able to pass in the rewards address field to the delegation certificate, ultimately it was decided that the dApp should not be able to do this. Allowing users to select reward addresses via connected dApps would allow voting rewards to be passed to another wallet’s control. But this would also allow malicious dApps to substitute in their own rewards address. By passing the reward address burden onto the wallet limits the damage a malicious dApp is able to do. This does not prevent wallets from implementing their own methods to allow users to pass rewards addresses.

## Voting Flow

Originally it was proposed that wallets would solely bear the responsibility of packing vote objects into transactions and submitting these to the Jormungandr sidechain. This raised concerns as running Jormungandr nodes would introduce cost for wallet providers with considerable complexity for a narrow use case; Catalyst vote submission. It was concluded that it would be better for dApps to submit voting transactions themselves, this significantly reduces the complexity of wallet implementations. Only requiring wallets to convert vote objects into signed vote transactions. This has the added benefit of making the specification less Catalyst specific.

# Examples of Message Flows and Processes

## Delegation

### Catalyst Registration

Recall from [CIP-36](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0036) a registration is a self-delegation, allocating one's voting power to one's own voting key, to be used on the voting blockchain.

1. **Get Voting Key** - dApp calls the method `api.getCurrentVotingKey()` to return the connected wallet account's public `VotingKey`.

2. **Construct Delegation** - The dApp constructs `Delegation` using the Wallet's public `VotingKey`, `weight` of 1 and `VotingPurpose` of 0 to denote Catalyst.   

3. **Submit Delegation** - The dApp passes the `Delegation` object to the Wallet to build a metadata transcation and submit this to Cardano blockchain. Wallets are able employ the already existing [`api.submitTx()`](https://cips.cardano.org/cips/cip30/#apisubmittxtxcbortransactionpromisehash32), available from [CIP-30](https://cips.cardano.org/cips/cip30/).

### Delegation to Catalyst dReps 

1. **Collect Voting Keys** - Catalyst Voting Center dApp connects to dRep wallet's and using `api.getCurrentVotingKey()` and collects dRep's public `VotingKey`.

2. **Construct Delegation** - A prospective delegating user browses dReps on the Catalyst Voting Center dApp selecting their choice of delegation and weight. The dApp uses the chosen dRep choices and weight to constuct `Delegation`.

3. **Submit Delegation** - The dApp passes the `Delegation` object to the Wallet to build a metadata transcation and submit this to Cardano blockchain. Wallets are able employ the already existing [`api.submitTx()`](https://cips.cardano.org/cips/cip30/#apisubmittxtxcbortransactionpromisehash32), available from [CIP-30](https://cips.cardano.org/cips/cip30/).

## Voting

### Casting Public Catalyst Votes

TODO: once voting design is fully spec-ed 

1. **Collect User Choices** - 
2. **Submit Votes** - 

### Casting Private Catalyst Votes

1. **Collect User Choices** - 
2. **Submit Votes** -


## Test Vector

See [test vector file](./test-vector.md).