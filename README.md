# sideos DID Method Specification
v0.1, Marcus Nasarek, sideos GmbH

## Introduction

sideos is a service middleware providing SSI services to manage DIDs and Verifiable Credentials. The goal is to ensure interoperability between multiple SSI technologies and improve the privacy and efficiency of SSI protocols. 

sideos supports private and public DIDs depending on the privacy requirements of a specific use case. For a private DID we are using the `did:key` method which is described [here](https://w3c-ccg.github.io/did-method-key/).

For the public DIDs we are using our `did:sideos` method described in the sections below.

### Why a sideos DID method?

To support basic business requirements appropriately there need to be certain elements part of the DID management layer which are beyond the DID core feature set. This said, to ensure interoperability and availability we need to consider the business layer and taking into account the following objectives: 
* cross-chain support: businesses may have certain constraints preference to us a specific ledger. To consider also legal, regulatory and business requirements, the sideos DID method is ledger-agnostic
* bidirectional trust: The interaction between to entities may require certain conditions to set up constraints for the interaction. Mutual establishment of trust may be necessary before the exchange of data is allowed. 
* multi-layer privacy: Privacy comes in multiple flavors and in a business environment is needs to consider regional requirements to decide what conditions need to be applied before certain data may be requested or provided.     
* responsivness: The speed of transactions is crucial for businesses and user experience. As many blockchain-related transactions may be unbearable slow for certain ledgers. DID services need to responde and finalize transactions as fast as possible. 
* business-readiness: Interoperabilty ensures reacheabilty which is among the most crucial business requirements. But even more important is the support of business workflows, the user experience, legal compliance and the predictability of interactions. 

To support the business layer accordingly we introduced our sideos DID method making SSI features easier to integrate into business applications and handling DIDs as efficient as possible from a user perspective.     

## Overview

DIDs need to be resolvable und following the standards it is required to implement [DID Resolution](https://www.w3.org/TR/did-core/#resolution). 

## sideos Specifications

### DID Scheme

sideos DIDs are identifiable by `did:sideos` method-name and conform to the [Generic DID Scheme](https://w3c-ccg.github.io/did-spec/#the-generic-did-scheme).

### Syntax

did                = "did:" method-name ":" method-specific-id\
method-name        = "sideos:"  
method-specific-id = [ version ":" ] 36*36base58-char
version            = v 3*3(lower-char / DIGIT)
base58-char        = ALPHA / DIGIT ; A-Z / a-z / 0-9
lower-char         = %x61-7A  ; a-z
v                  = "v"

### DID Creation

The method-specific-id component is created as the following:
1. generate 256 random bits and a 3 byte crypto marker
2. create a Ed25519 key pair as a PKCS8 
4. BLAKE2 hash of the public key from the PKCS8 with a length of 160 bits
3. base58 encode the hash prefixed with 3 bytes of the crypto marker

## CRUD Operations

The sideos DID method supports the CRUD lifecycle of a DID providing functionalities to create, resolve, update and deactivate a DID.

sideos DIDs are registered on a ledger which publically reacheable and identified by the `version` element of the DID. 

### CREATE

Creating a DID: 

1. Create a DID document
2. Register the DID on the ledger, e.g. through a smart contract or other means of mapping the DID to the DID Document
3. Create off-chain DID document updates if required

### RESOLVE (READ)

Resolving a DID Document from a given DID:
1. Provide a DID 
2. Lookup the respective DID Document 
3. Apply possible off-chain DID Document updates

### UPDATE

There are 2 options for DID Document updates. Option 1 is the offline update and no need to make changes to the registry. Option 2 is an update to the ledger:
1. Provide a DID
2. Look-up the respective DID Document and prepare the versioning
3. Retrieve the Identity Holders permission to update
4. Perform the versioning of the DID Document reference

### DEACTIVATE OR DELETE

There are 2 options for the deactivation. 

Option 1 is to perform an update with an status indicating the DID Document and the respective DID are invalid from a certain point in time on. This is applied for immutable ledgers, e.g. blockchain ledgers.

Option 2 is to perform a deletion event and delete or deactivate access to the data record on the ledger, e.g. dropping decryption keys or deleting the record. This is applied to mutable ledgers or ledgers with encrypted records, which cannot be read without the respecitve decryption key. 


## Key Management

### Key Recovery 

The keys are generated within the premise of the Identity Holder or on its behalf. That implies that recovery of key material is subject of business layer and not in the scope of this document. 

### Key Revocation

Expired or invalid DIDs can be identified when created and registered on a public ledger. The DEACTIVATE operation applies. Only the Identity Holder can grant permissions to perform an DEACTIVATE operation. 

## Privacy and Security Considerations

### Privacy

As sideos distingueshes private and public DIDs a general privacy decision is made at the time of creating a DID. Private DIDs are following the `did:key` [method](https://w3c-ccg.github.io/did-method-key/) and not discussed here. Public DIDs are per definition publically reachable and stored on a public ledger. The DID Document can be resolved by anyone w/o restrictions. As this is the purpose of a public DID any user or business requiring a public DID should be aware of that fact. 

The Verifiable Credentials managed by the Identity Holder of a public DID are not necessarely stored on the public ledger. Still, the Identity Holder should be aware that survillance may be possible tracking the DID through captured transactions. 

### Security

Security requires multiple layers of defense. The software implementation of the resolver incorporates cryptographic libraries and security features provided by many vendors, mostly supervised by the open source community. In most of the cases the community detects vulnerabilties quickly and provides fixes to security flaws. Keeping up with the maturity process and apply a secure software lifecycle is crucial for the resolver but also for the connecting services. 

Cryptographic weaknesses may occure if new technologies are able to break through the protection layer or conceptual shortcuts in the cryptographic algorithms are discovered. The sideos DID method is using a versioning system to be able to migrate to updates of the underlying cryptographic primitives. 

DID documents may be tempared with if the storage of the DID document has certain vulnerabilities with respect to the access permissions or integrity protection. It is part of the defense strategy to protect DID Document on the storage and transmission layer. 

If the Identity Holder looses control over the secret keys there may be unauthorized changes to the DID Document via the UPDATE operation or a key revocation via the DEACTIVATION operation is not possible inthe case the keys are lost or destroyed. 








