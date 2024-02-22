__Project Name:__ 

__Prepared By:__ 10amBear

<br />

__Protocol Name__ engaged __10amBear__ to review the security of its Smart Contract system. From __Begining date__ to __Ending date__ the source code in scope was reviewed. All findings have been recorded in the following report.

Notice that the examined smart contracts are not resistant to external/internal exploit. For a detailed understanding of risk severity, source code vulnerability, and potential attack vectors, refer to the complete audit report below.

# Project Overview

| Project Name | __Project__                                                                                               |
|--------------|----------------------------------------------------------------------------------------------------------|
| Language     | Solidity                                                                                                 |
| Codebase     | https://github.com/___                             |
| Commit       | [_____](https://github.com/___) |
| Delivery Date     | ____       |

| Vulnerability Level | Total | Pending | Declined | Acknowledged | Partially Resolved | Resolved |
|---------------------|-------|---------|----------|--------------|--------------------|----------|
| [Critical](#Critical)| 0     | 0       | 0        | 0            | 0                  | 0        |
| [High](#High)        | 0     | 0       | 0        | 0            | 0                  | 0        |
| [Medium](#Medium)    | 0     | 0       | 0        | 0            | 0                  | 0        |
| [Low](#Low)          | 0     | 0       | 0        | 0            | 0                  | 0        |

# Audit Scope & Methodology

## Scope

| ID | File      |
|---------|------------|
| DPT      | contracts/DividendPayingToken.sol |
| TOK      | contracts/Token.sol | 
| PRE      | contracts/Presale.sol | 
| AIR     | contracts/Airdropper.sol | 

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
| [H-01](#H01)  | A DOS can happen on XXXXXXX                                     | DOS                 | HIGH     | Pending |
| [H-02](#H02)  | Lack of access controls                                                        | Access Controls         | HIGH     | Pending |
| [M-01](#M01)  | Inherent risk of future exploits that may or may not impact the smart contract execution. | Token integration   | MEDIUM   | Pending |
| [M-02](#M02)  | Minor deviation from best practices.                                                      | Logic error         | MEDIUM   | Pending |
| [L-01](#L01)  | Low error 01                                                                              | Centralization risk | LOW      | Pending |
| [L-02](#L02)  | Low error 02                                                                              | Validation          | LOW      | Pending |




## <a id="Critical"></a>Critical


### <a id="C01"></a> C-01 Title issue

https://github.com/___/Contract.sol#L146-L160


#### PoC:


#### Description:





#### Recommendation:





#### Resolution:


----


## <a id="High"></a> High

### <a id="H01"></a> H-01 Title issue

https://github.com/___/Contract.sol#L146-L160


#### PoC:


#### Description:





#### Recommendation:





#### Resolution:



### <a id="H02"></a> H-02 Title issue

https://github.com/___/Contract.sol#L146-L160


#### PoC:

#### Description:





#### Recommendation:





#### Resolution:


-----------------



## <a id="Medium"></a> Medium

### <a id="M01"></a> M-01 Title issue

https://github.com/___/Contract.sol#L146-L160


#### Description:





#### Recommendation:





#### Resolution:

-----------------


## <a id="Low"></a> Low


### <a id="L01"></a> L-01 Title issue


https://github.com/___/Contract.sol#L146-L160

#### Description:





#### Recommendation:





#### Resolution:

____

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

