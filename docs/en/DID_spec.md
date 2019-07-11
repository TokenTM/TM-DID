
Method Name| Status | DLT or Network | Authors | Link
---|---|---|---|---
did:ttm: | PROVISIONAL | TMChain | Token.TM | [TokenTM DID Method](https://github.com/TokenTM/TM-DID)


# TokenTM DID Method
The following document defines TokenTM's DID method.

The following specifications may change in the future, but they must conform to the latest version of general DID standardization established by the W3C credentials community group.

The functionality of this method DID is provided by smart contract in the data repository.

## 1. DID Format
This method uses ```ttm``` as identification. TokenTM DID has the following format:

```
did:ttm:<32 byte hexadecimal string>
```

For example:

```
did:ttm:0xe32df42865e97135acfb65f3bae71bdc86f4d49150ad6a440b6f15878109880a
```

<32 byte hexadecimal string> corresponds to keccak256 and the hash value of Ethereum address connected by random numbers generated in the DID contract.

DID is registered in the contract and controlled by a single Ethereum address, which is set by default to the address where the createDID method was originally called. Then, this address can transfer control to a different address, or update/delete the corresponding DID in the contract.
## 2. DID Generation
The identifier string is generated in the following line of the DID contract:

```
Bytes32 _hash = keccak256(abi.encodepc (msg.sender, nonce));
```

Where nonce increases in each call, so that the result is considered to be random, the address is able to create and control multiple DID.

## 3. Definition of DID Structure 
Each DID entry in the ledger shows the following structure:
```
Struct DID {
	address the owner;
	uint256 created;
	uint256 updated;
	bool revoked;
}
```

When DID Deactivate revoked shown as true.

## 4. CRUD Operations
The following sections define the operations supported by managing DID.
### Create
DID creation is completed by a transaction that invokes the following method through the TM_DID contract submission:

```
createDID ()
```

Record msg.sender of DID as owner, record creation time, revoked set to true.

### Read
The records of DID are read by a transaction that invokes the following method through the TM_DID contract submission:
```
getDID(address didAddress) public view returns(address, uint256, uint256, bool)
```

The return values need to be formatted as follows json strings:
```
{
}
```
### Update
By calling 
```
SetAttribute (address didAddress, bytes32 name, bytes memory value, uint validity)
```
to update attributes of DID.
```
RevokeAttribute (address didAddress, bytes32 name, bytes memory value)
```
to revoke attrubute.
### Delete
```
RevokeDID (address didAddress)
```
set revoked to true.

### Agent
#### Determine if the agent is valid

```
ValidDelegate(address didAddress, bytes32 delegateType, address delegate) returns(bool)
```
#### Add agent
```
AddDelegate (address didAddress, bytes32 delegateType, address delegate, uint validity)
```
#### Deactivate agent

```
RevokeDelegate (address didAddress, bytes32 delegateType, address delegate)
```

## 5. Extentions
The DID ledger is implemented as a simple layer of persistent identity registration on the Ethereum blockchain network. Meanwhile it can be extended to contain other data and functions. Scalability is achieved by using an identity contract as a controller for DID on the ledger. In particular, ERC725 combined with private key management contract, for example, ERC734 is expected to involve additional features with common conditions (such as defines service endpoint, private key rotation, delegation and licensing, etc.). At the meantime, it gives the permit of exploiting other standards, and even allows the owner of DID to transforms contract implementation to another without losing its identifier.
### Security Considerations
The following points should be taken into consideration and the community should discuss these general security issues:

- DID documents are not explicitly stored but dynamically generated, which represents they cannot be signed. Therefore, the dependent parties need to trust parser code to execute correctly and they are expected to use the validating (such as check and verify) version of the parser library code provided by TokenTM or other trusted sources in the community.
- Once controller address transfers the control of DID to a new address, it loses all ability to execute operations on the DID. Therefore, you must be careful to do this to avoid errors (transferring DID control to a wrong address or out of user control address).
- This method does not define delegation or recovery mechanisms at ledger level. Applicable recoverability must be implemented through private key management and proxy identity smart contracts (such as ERC725 / ERC734). This is expected to become a common practice for managing DID under this method.
### Privacy Considerations
With the help of the anonymity of ethereum address, the anonymity of DID is guaranteed (because DID is controlled by the private key corresponding to the address). Since it will not store any data of the user on the block chain ledger, then DID documents cannot do reverse information calculation and comprehensive information analysis to contact someone in the real world.
### Supplemental explanation
Other approaches have been proposed to provide decentralized identities on Ethereum platforms. However, the TokenTM DID method is based on the following principles:
- Ideally, DID should be "timestamped" and revocable. General Ethereum addresses do not meet this requirement.
- The public data associated with DID and the presentation structure of this data should be upgraded based on general specification or in this particular method. This is done by providing a simple identifier < -> address map and a parser, which can be interpreted as flexible metadata properties with different parsed ways. It is expected that DID under this method will be associated with proxy identity smart contracts, aggregated necessary data, while allowing simple accounts to control DID to deal with some simple situations.
- All participants should be able to announce statements in public about any arbitrary entity (whether it exists in the Ethernet network or not). As regarded, the bytes32 trusted declaration type can refer to TokenTM DID or a hash representing any other arbitrary thing on or off chain. While the description of trusted claims is beyond the scope of this method, public claims registration contracts are also being developed for practice with DID as defined in this document.
