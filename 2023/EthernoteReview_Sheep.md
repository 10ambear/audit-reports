__Project Name:__ Ethernote

__Prepared By:__ 10ambear

<br />

__Ethernote__ engaged __10ambear__ to review the security of its Smart Contract system. From __20 November 2023__ to __24 November 2023__ the source code in scope was reviewed. All findings have been recorded in the following report.

Notice that the examined smart contracts are not resistant to external/internal exploit. For a detailed understanding of risk severity, source code vulnerability, and potential attack vectors, refer to the complete audit report below.



# Project Overview

| Project Name | __Ethernote__                                                                                               |
|--------------|----------------------------------------------------------------------------------------------------------|
| Language     | Solidity                                                                                                 |
| Codebase     | https://github.com/Slot0x0f/PA-Ethernote-Foundry                             |
| Commit       | [_____](https://github.com/Slot0x0f/PA-Ethernote-Foundry/commit/c7fb03531e8bd43c2a2de0f165500106b55a1415) |


| Delivery Date     | 24 November 2023       |
|-------------------|--------------------------------|
| Audit Methodology | Static Analysis, Manual Review |


| Vulnerability Level | Total | Pending | Declined | Acknowledged | Partially Resolved | Resolved |
|---------------------|-------|---------|----------|--------------|--------------------|----------|
| [Critical](#Critical)| 0     | 0       | 0        | 0            | 0                  | 0        |
| [High](#High)        | 1     | 0       | 0        | 0            | 0                  | 0        |
| [Medium](#Medium)    | 2     | 0       | 0        | 0            | 0                  | 0        |
| [Low](#Low)          | 3     | 0       | 0        | 0            | 0                  | 0        |

# Audit Scope & Methodology

## Scope

| ID | File      | SHA-1 Hash                              |
|---------|------------|---------------------------------------|
| ETN      | src/ethernote.sol |fd9febd69c6c4ffb386784f0af4415bfdd409905  |

## Methodology

The auditing process pays special attention to the following considerations:
- Testing the smart contracts against both common and uncommon attack vectors.
- Assessing the codebase to ensure compliance with current best practices and industry standards.
- Ensuring contract logic meets the specifications and intentions of the client.
- Cross-referencing contract structure and implementation against similar smart contracts produced by industry leaders.
- Thorough line-by-line manual review of the entire codebase by community auditors.

## Vulnerability Classifications

| Vulnerability Level | Classification                                                                               |
|---------------------|----------------------------------------------------------------------------------------------|
| [Critical](#Critical)            | Easily exploitable by anyone, causing causing loss of assets or undermining of the protocol’s goals.                 |
| [High](#High)                | Arduously exploitable by a subset of addresses, causing loss of assets or undermining of the protocol’s goals. |
| [Medium](#Medium)              | Inherent risk of future exploits that may or may not impact the smart contract execution.    |
| [Low](#Low)                 | Minor deviation from best practices.                                                         |



# Findings & Resolutions

| ID      | Title                                                                                     | Category            | Severity | Status  |
|-------|-------------------------------------------------------------------------------------------|---------------------|----------|---------|
| [H-01](#H01)  | Duplicate notes for the same edition                                   | Architecture                 | HIGH     | Pending |
| [M-01](#M01)  | Missing pause/unpause functionality | Token integration   | MEDIUM   | Pending |
| [M-02](#M02)  | Consider using roles in stead of ownable                                                    | Architecture       | MEDIUM   | Pending |
| [L-01](#L01)  | Floating pragma                                                                       | Solidity | LOW      | Pending |
| [L-02](#L02)  | No zero address check for `validatorAddress`                                                                          | Validation          | LOW      | Pending |
| [L-03](#L02)  | Mapping for `editions` and `notes`                                                                          | Architecture          | LOW      | Pending |


## <a id="High"></a> High

### <a id="H01"></a> H-01 Duplicate notes for the same edition
https://github.com/Slot0x0f/PA-Ethernote-Foundry/blob/c7fb03531e8bd43c2a2de0f165500106b55a1415/src/ethernote.sol#L98C1-L108C6


#### PoC:
```solidity
    function test_DuplicateNotesForSameEdition() external {
        // id 0
        _createEdition("1st Edition", false, address(0), true, BASE_URL);
        // id 1
        _createEdition("2nd Edition", false, address(0), true, BASE_URL);

        // intended case: note intended for "1st Edition" i.e. editionId 0
        _createNote(0, 1e17, 300, true);

        // unintended case: note intended for "2nd Edition" i.e. editionId 1
        // but edition 0 was used
        _createNote(0, 1e17, 300, true);

        // check if both notes are created
        uint256 getNotesLength = ethernote.getNotesLength();
        assertEq(2, getNotesLength);

        // check that noteId 1 was created with editionId 0
        (uint256 edition,,,) = ethernote.notes(1);
        assertEq(edition, 0);
    }
```

#### Description:
Referring to the protocol readme and struct representing `Notes` "Each note type has an associated edition", we're assuming that each `note` should only have one `edition`. An owner could technically add the same `edition` to multiple `notes`. This could be dangerous considering `editions` and `notes` are not mapped, but added to a dynamic array. The owner could lose track of the `edition` and `note` ids, and map the incorrect `edition` to the incorrect `note`. The result of this could have rippling effects through the minting function as the `edition` also makes use of a `validatorAddress` i.e. the incorrect address could mint the incorrect edition for the incorrect price thus financially impacting the protocol.

#### Recommendation:
Map the `edition id to the `note` id via a mapping and use the mapping to check if the edition is already mapped to a note. 

#### Resolution:

-----------------

## <a id="Medium"></a> Medium

### <a id="M01"></a> M-01 Missing pause/unpause functionality
https://github.com/Slot0x0f/PA-Ethernote-Foundry/blob/c7fb03531e8bd43c2a2de0f165500106b55a1415/src/ethernote.sol#L24C8-L24C8

#### Description:
The OpenZeppelin `Pausable` contract was imported which assumes that the `Ethernote` contract should be `Pausable` however this is not the case. The `Ethernote` contract will have to override the `_pause` and '_unpause' functions from the `Pausable` contract to make use of the intended functionality.

#### Recommendation:
Override the `_pause` and '_unpause' functions to make use of the `Pausable` functionality.

#### Resolution:
This was upgraded to a high
-----------------

## <a id="Medium"></a> Medium

### <a id="M02"></a> M-02 Consider using roles in stead of ownable
https://github.com/Slot0x0f/PA-Ethernote-Foundry/blob/c7fb03531e8bd43c2a2de0f165500106b55a1415/src/ethernote.sol#L4

#### Description:
The `owner` has a myriad of "critical" responsibilities namely creating/editing the `notes` and `editions`. Ceasing the `notes`, withdrawing the fees, updating the `payout`address and pausing the contract (when correctly implemented). If the owner is not a multisig, gets compromised or renounces the ownership of the contract the protocol would not be able to function.

#### Recommendation:
Consider implementing role based access control as per the example: https://docs.openzeppelin.com/contracts/2.x/access-control

#### Resolution:

-----------------

## <a id="Low"></a> Low

### <a id="L01"></a> L-01 Floating pragma
https://github.com/Slot0x0f/PA-Ethernote-Foundry/blob/c7fb03531e8bd43c2a2de0f165500106b55a1415/src/ethernote.sol#L2C1-L2C1

#### Description:
Contracts should be deployed with the same compiler version and flags used during development and testing. Locking the pragma helps to ensure that contracts do not accidentally get deployed using another pragma. For example, an outdated pragma version might introduce bugs that affect the contract system negatively or recently released pragma versions may have unknown security vulnerabilities

#### Recommendation:
Consider locking the pragma to the latest solidity version 0.8.23

#### Resolution:

____

## <a id="Low"></a> Low

### <a id="L02"></a> L-03 No zero address check for `validatorAddress`
https://github.com/Slot0x0f/PA-Ethernote-Foundry/blob/c7fb03531e8bd43c2a2de0f165500106b55a1415/src/ethernote.sol#L89

#### Description:
There are no zero address checks for the `validatorAddress` when an edition is created. This could have unintended consequences. 

#### Recommendation:
Consider adding a require statement to make sure the owner cannot accidentally add the zero address. 

#### Resolution:

____

## <a id="Low"></a> Low

### <a id="L03"></a> L-03 Mapping for `editions` and `notes`
https://github.com/Slot0x0f/PA-Ethernote-Foundry/blob/c7fb03531e8bd43c2a2de0f165500106b55a1415/src/ethernote.sol#L59-L60

#### Description:
Referring to the check below: 

```
        if (editions[notes[_id].edition].validator) {
            require(editions[notes[_id].edition].validatorAddress == msg.sender, "[x] Not Validator");
        }
```

The code is quite hard to read which causes unnecessary complexity which that could lead to unintended consequences. The way the ids are stored currently also have no checks to make sure that the ids are unique. A mapping would make the code more readable and would make it easier to enforce unique id checks. 

#### Recommendation:
Consider using a mapping for `editions` and `notes`:

// editionId => Edition
mapping (uint256 => Edition) editions;
// noteId => Note
mapping (uint256 => Note) notes;

#### Resolution:




## Disclaimer
> 
> This report is not, nor should be considered, an “endorsement” or “disapproval” of any particular project or team. This report is not, nor should be considered, an indication of the economics or value of any “product” or “asset” created by any team or project that contracts the firm to perform a security assessment. This report does not provide any warranty or guarantee regarding the absolute bug-free nature of the technology analyzed, nor do they provide any indication of the technologies proprietors, business, business model or legal compliance.
> 
> This report should not be used in any way to make decisions around investment or involvement with any particular project. This report in no way provides investment advice, nor should be leveraged as investment advice of any sort. This report represents an extensive assessing process intending to help our customers increase the quality of their code while reducing the high level of risk presented by cryptographic tokens and blockchain technology.
> 
> Blockchain technology and cryptographic assets present a high level of ongoing risk. The firm’s position is that each company and individual are responsible for their own due diligence and continuous security. The firm’s goal is to help reduce the attack vectors and the high level of variance associated with utilizing new and consistently changing technologies, and in no way claims any guarantee of security or functionality of the technology we agree to analyze.
> 
> The assessment services provided by the firm is subject to dependencies and under continuing
> development. You agree that your access and/or use, including but not limited to any services, reports, and materials, will be at your sole risk on an as-is, where-is, and as-available basis. Cryptographic tokens are emergent technologies and carry with them high levels of technical risk and uncertainty. The assessment reports could include false positives, false negatives, and other unpredictable results. The services may access, and depend upon, multiple layers of third-parties.
> 
> Notice that smart contracts deployed on the blockchain are not resistant from internal/external exploit. Notice that active smart contract owner privileges constitute an elevated impact to any smart contract’s safety and security. Therefore, the firm does not guarantee the explicit security of the audited smart contract, regardless of the verdict.

<br/>

