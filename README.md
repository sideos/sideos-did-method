# sideos DID Method Specification
v0.1, Marcus Nasarek, sideos GmbH

## Introduction

sideos is a service middleware providing SSI services to manage DIDs and Verifiable Credentials. The goal is to ensure interoperability between multiple SSI technologies and improve the privacy and efficiency of SSI protocols. 

sideos supports private and public DIDs depending on the privacy requirements of a specific use case. For a private DID we are using the `did:key` method which is described [here](https://w3c-ccg.github.io/did-method-key/). 

For the public DIDs we are using our `did:sideos` method described in the sections below.

### Why a sideos DID method?

To support basic business requirements appropriately there need to be certain elements part of the DID management layer which are beyond the DID core feature set. This said, to ensure interoperability and availability we need to consider the business layer and taking into account the following objectives: 
* cross-chain support: businesses may have certain constraints preferring to us a specific ledger. To consider also legal, regulatory and business requirements, the sideos DID method is ledger-agnostic
* bidirectional trust: The interaction between two entities may require certain conditions to set up constraints for the interaction. Mutual establishment of trust may be necessary before the exchange of data is allowed. 
* multi-layer privacy: Privacy comes in multiple flavors and in a business environment it needs to consider regional requirements to decide what conditions need to be applied before certain data may be requested or provided.     
* responsiveness: The speed of transactions is crucial for businesses and user experience. As many blockchain-related transactions may be unbearably slow for certain ledgers. DID services need to responde and finalize transactions as fast as possible. 
* business-readiness: Interoperability ensures reachability which is among the most crucial business requirements. But even more important is the support of business workflows, the user experience, legal compliance and the predictability of interactions. 

To support the business layer accordingly we introduced our sideos DID method making SSI features easier to integrate into business applications and handling DIDs as efficiently as possible from a user perspective.     

## Overview

DIDs need to be resolvable und following the standards it is required to implement [DID Resolution](https://www.w3.org/TR/did-core/#resolution). 

## sideos Specifications

### DID Scheme

sideos DIDs are identifiable by `did:sideos` method-name and conform to the [Generic DID Scheme](https://w3c-ccg.github.io/did-spec/#the-generic-did-scheme).

### Syntax

Using the following ABNF rules for definition:

did                = "did:" method-name ":" method-specific-id\
method-name        = "sideos"\
method-specific-id = [ version ":" ] 36\*36base58-char\
version            = v 3\*3(lower-char / DIGIT)\
base58-char        = ALPHA / DIGIT ; A-Z / a-z / 0-9\
lower-char         = %x61-7A  ; a-z\
v                  = "v"

### DID Creation

The method-specific-id component is created as the following:

1. generate 256 random bits 
2. create a key pair from the secp256k1 elliptic curve
4. BLAKE2 hash of the public key from the SPKI structure in a DER format with a length of 256 bits
3. base58 encode the hash

The method-specific-id is used to create the DID following the syntax above. 

The DID created is globally unique. 

### DID Documents

sideos DID documents follow the [DID Core Specification](https://www.w3.org/TR/did-core/#abstract) utlizing the JSON-LD format. 

## CRUD Operations

The sideos DID method supports the CRUD lifecycle of a DID providing functionalities to create, resolve, update and deactivate a DID.

sideos DIDs are registered on a ledger which is publicly reachable and identified by the `version` element of the DID. 

### CREATE

Creating a DID Document entry: 

1. Presuming a DID document
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

Option 2 is to perform a deletion event and delete or deactivate access to the data record on the ledger, e.g. dropping decryption keys or deleting the record. This is applied to mutable ledgers or ledgers with encrypted records, which cannot be read without the respective decryption key. 

Both options perform the following steps to deactivate a DID Document (Reference):

1. Provide a DID 
2. Look-up the respective DID Document
3. Retrieve the Identity Holders permission to delete
4. Perform the deactivation of the DID Document reference

## Key Management

### Key Recovery 

The keys are generated within the premise of the Identity Holder or on its behalf. That implies that recovery of key material is subject to the business layer and not in the scope of this document. 

### Key Revocation

If the private key of a DID is compromised or becomes or invalid, DIDs can be identified when created and registered on a public ledger. The DEACTIVATE operation applies. Only the Identity Holder can grant permissions to perform an DEACTIVATE operation. 

## Privacy and Security Considerations

### Privacy

As sideos distinguishes private and public DIDs a general privacy decision is made at the time of creating a DID. Private DIDs are following the `did:key` [method](https://w3c-ccg.github.io/did-method-key/) and are not discussed here. 

Public DIDs are per definition publicly reachable and stored on a public ledger. The DID Document can be resolved by anyone w/o restrictions. As this is the purpose of a public DID any user or business requiring a public DID should be aware of that fact. 

#### surveillance

The Verifiable Credentials managed by the Identity Holder of a public DID are not necessarily stored on the public ledger. Still, the Identity Holder should be aware that surveillance may be possible tracking the DID through captured transactions. 

#### stored data compromise

Data stored on a public ledger may be compromised. sideos supports only ledgers with inherent integrity protection, e.g. blockchains or permissioned distributed ledgers. The integrity protection may fail, so we are utilizing open source ledgers which are supported actively by the big community.

#### unsolicited traffic

The protection against unsolicited traffic or denial-of-service messages are subject to the infrastructure layer and not scope of the method-specific privacy considerations.

#### misattribution

The interaction protocols processing sideos DIDs are protected on multiple layers against misattribution and account-take-over. On an identity layer SSI ensures the authenticity of datagrams, whereas the interaction layer, e.g. within a mobile phone's environment or on cloud agents acting on behalf of a user are subject of the environmental security layer.

#### correlation

The correlation of data captured during the CRUD operations of a sideos DID transaction or during related SSI-based interactions may lead to insights affecting the privacy. Although the communication channel is encrypted the parties involved in the communication may try to correlate data elements to other information available. The correlation is addressed in the way a sideos presentation is created by processing only data relevant in an interaction. No additional data is involved, not in plain nor encrypted or hashed.  

#### identification

The topic identification as addressed in the section 5.2.2 of the [RFC6973](https://datatracker.ietf.org/doc/html/rfc6973#section-5.2.2) is what SSI is about. The inherent characteristics of the technology ensure that the identity of interacting individuals can be validated. The natural person or physical object to be identified by SSI credentials have to be linked on the layer above SSI and out of scope of this document.

#### secondary use

The secondary use of the information contained in a Verifiable Credential provided via SSI can not be excluded. Data fields from a Verifiable Credential presented as a JWT can be extracted as plain data records and be used elsewhere. The data records of a Verifiable Credential are usually not encrypted. The values of a data field may be protected by encryption which implies that an additional permission layer and key management protocol is required to ensure confidentiality. 

#### disclosure

The structure, naming convention and number of data fields reveal information that allows other parties receiving DIDs and Verifiable Credentials to make certain conclusions. The sideos method creates presentations in a way that it implements the need-to-know principle. Only data elements which are required for the respective purpose are captured for the specific identity presentation. 

#### exclusion

The nature of SSI enforces the involvement of the Identity Holder into the handling and management of its data. Specifically, the focus of SSI is the secure management of data describing the context of an identity. This said, the sideos DID methods allow the Identity Holder to be involved in the management of identity data in the context of SSI protocols.

### Security

Security requires multiple layers of defense. The software implementation of the resolver incorporates cryptographic libraries and security features provided by many vendors, mostly supervised by the open source community. In most of the cases the community detects vulnerabilities quickly and provides fixes to security flaws. Keeping up with the maturity process and applying a secure software lifecycle is crucial for the resolver but also for the connecting services. 

Cryptographic weaknesses may occur if new technologies are able to break through the protection layer or conceptual shortcuts in the cryptographic algorithms are discovered. The sideos DID method is using a versioning system to be able to migrate to updates of the underlying cryptographic primitives. 

DID documents may be tempered with if the storage of the DID document has certain vulnerabilities with respect to the access permissions or integrity protection. It is part of the defense strategy to protect DID Document on the storage and transmission layer. 

If the Identity Holder loses control over the secret keys there may be unauthorized changes to the DID Document via the UPDATE operation or a key revocation via the DEACTIVATION operation is not possible in the case the keys are lost or destroyed. 