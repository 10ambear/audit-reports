
__Project Name:__ Cd Pro

  

__Prepared By:__ 10ambear

  

<br />

  

__CDPro__ engaged __10ambear__ to review the security of its Smart Contract system. All findings have been recorded in the following report.

  

Notice that the examined smart contracts are not resistant to external/internal exploit. For a detailed understanding of risk severity, source code vulnerability, and potential attack vectors, refer to the complete audit report below.

  
  
  

# Project Overview

  

| Project Name | __CDPro_ |

|--------------|----------------------------------------------------------------------------------------------------------|

| Language | Solidity |

| Codebase | https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/tree/main|

| Commit | https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/commit/727d97a924b75c76c896d71661909b96e6a8ce57 |

  
  

| Delivery Date | 11 December 2023 |

|-------------------|--------------------------------|

| Audit Methodology | Static Analysis, Manual Review |

  
  

| Vulnerability Level | Total | Pending | Declined | Acknowledged | Partially Resolved | Resolved |

|---------------------|-------|---------|----------|--------------|--------------------|----------|

| [Critical](#Critical)| 5 | 0 | 0 | 0 | 0 | 0 |

| [High](#High) | 2 | 0 | 0 | 0 | 0 | 0 |

| [Medium](#Medium) | 4 | 0 | 0 | 0 | 0 | 0 |

| [Low](#Low) | 5 | 0 | 0 | 0 | 0 | 0 |

  

# Audit Scope & Methodology

  

## Scope

  

| ID | File | SHA-1 Hash |

|---------|------------|---------------------------------------|

| GOV | src/Gov.sol| |
| GOVT | src/GovToken.sol| |
| CDP | src/CDPro.sol| |
| CDP | src/StableCoin.sol| |

  

## Methodology

  

The auditing process pays special attention to the following considerations:

- Testing the smart contracts against both common and uncommon attack vectors.

- Assessing the codebase to ensure compliance with current best practices and industry standards.

- Ensuring contract logic meets the specifications and intentions of the client.

- Cross-referencing contract structure and implementation against similar smart contracts produced by industry leaders.

- Thorough line-by-line manual review of the entire codebase by community auditors.

  

## Vulnerability Classifications

| Vulnerability Level | Classification |

|---------------------|----------------------------------------------------------------------------------------------|

| [Critical](#Critical) | Easily exploitable by anyone, causing causing loss of assets or undermining of the protocol’s goals. |

| [High](#High) | Arduously exploitable by a subset of addresses, causing loss of assets or undermining of the protocol’s goals. |

| [Medium](#Medium) | Inherent risk of future exploits that may or may not impact the smart contract execution. |

| [Low](#Low) | Minor deviation from best practices. |


# Findings & Resolutions

| ID      | Title                                                                                     | Category            | Severity | Status  |
|-------|-------------------------------------------------------------------------------------------|---------------------|----------|---------|
| [C-01](#C01) | Liquidators can liquidate `healthy` positions | Logic error | Critical | Pending |
| [C-02](#C02) | Cannot liquidate all positions that break the LTV ratio | Logic error | Critical | Pending |
| [C-03](#C03) | Missing ownership checks on `GovToken` | Access Controls | Critical | Pending |
| [C-04](#C04) | Withdraw collateral doesn't take all collateral tokens into account | Logic error | Critical | Pending |
| [C-05](#C05) | Incorrect fee calculation | Decimals | Critical | Pending |
| [H-01](#H01) | Voters can `vote` multiple times | Access Controls | HIGH | Pending |
| [H-02](#H02) | Users with <1% governance tokens can propose | Access Controls | HIGH | Pending |
| [M-01](#M01) | Propose doesn't automatically count the proposer's votes | logic error | MEDIUM | Pending |
| [M-02](#M02) | Users can vote for proposals after the `endTime` | Validation | MEDIUM | Pending |
| [M-03](#M03) | Native tokens can get stuck in the protocol | Logic error | MEDIUM | Pending |
| [M-04](#M04) | Centralization risk | Centralization | MEDIUM | Pending |
| [L-01](#L01) | Missing event for the liquidate function | Access Controls | LOW | Pending |
| [L-02](#L02) | Unnecessary operation in propose | Gas optimization | LOW | Pending |
| [L-03](#L03) | Imported frameworks that are not in use | Gas optimization | LOW | Pending |
| [L-04](#L04) | Floating pragma | Architecture | LOW | Pending |
| [L-05](#L05) | Parameter not in use | Gas optimization | LOW | Pending |


## <a id="Critical"></a>Critical
### <a id="C01"></a> C-01 Liquidators can liquidate `healthy` positions

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/CDPro.sol#L533C1-L550C1

#### PoC:
```solidity
function test_liquidate_healthy_position() public {

	// set up
	
	uint256 etherAmount = 1 ether;
	
	uint256 cdproToBorrow = 600 * 1e18;
	
	address user = makeAddr("user");
	
	vm.deal(user, etherAmount);
	
	// ---
	
	  
	
	vm.startPrank(user);
	
	// this swaps our eth for weth and approved cdpro to spend it
	
	WETH.mint(user, etherAmount);
	
	WETH.approve(address(cdpro), etherAmount);
	
	  
	
	// deposits weth into cdpro
	
	cdpro.depositCollateral(address(WETH), etherAmount);
	
	  
	
	// check balances
	
	uint256 userWethCollateral = WETH.balanceOf(address(cdpro));
	
	console2.log("User weth collateral", userWethCollateral / 1e18);
	
	  
	
	// check the usd value of the ether
	
	uint256 valueOfWETHCollateral = cdpro.getUsdValueOfCollateral(
	
	user,
	
	etherAmount,
	
	address(WETH)
	
	);
	
	console2.log("Value of Weth collateral:", valueOfWETHCollateral/1e18);
	
	  
	
	// // check how much cdpro can be borrowed
	
	uint256 howMuchCanUserBorrowWeth = cdpro.howMuchCanUserBorrowWETH();
	
	console2.log(
	
	"How much cdpro can I borrow for eth:",
	
	howMuchCanUserBorrowWeth / 1e18
	
	);
	
	  
	
	console2.log("Total borrowable cdpro:", howMuchCanUserBorrowWeth/1e18);
	
	cdpro.borrow(cdproToBorrow);
	
	uint256 userBorrowedAmount = cdpro.getPositionForIndividualUser(user);
	
	console2.log("User borrowed amount", userBorrowedAmount / 1e18);
	
	  
	
	// position = 600 cdpro
	
	// collateral = 1 weth
	
	// collateral value = $2000
	
	// ltv = 70%
	
	// available collateral = 2000 * 70% = $1400
	
	// should not be able to liquidate, but can liquidate returns true
	
	  
	
	bool canliquidate = cdpro.canLiquidate(user);
	
	console2.log("Can liquidate:", canliquidate);
	
	assertEq(canliquidate, true);

}
```
  
  

#### Description:
The `canLiquidate` function in the `CDPro.sol` contract will return `true` for certain positions that do not violate the LTV ratio. This could result in `liquidations` of health positions. It is also worth noting that the code will never make use of the last `else` in the function. The `liquidation` function is not feature complete even though it does rely on `canLiquidate`. 

#### Recommendation:
Recalculate the `availableDebtInUsd` & `maxDebtInUsd` to always return true when a position can be liquidated.

#### Resolution:


----
### <a id="C02"></a> C-02 Cannot liquidate all positions that break the LTV ratio

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/CDPro.sol#L493-L531

#### PoC:
```solidity
function test_liquidate_revert() public {

	// set up
	
	uint256 etherAmount = 1 ether;
	
	uint256 cdproToBorrow = 1400 * 1e18;
	
	address user = makeAddr("user");
	
	vm.deal(user, etherAmount);
	
	// ---
	
	  
	
	vm.startPrank(user);
	
	// this swaps our eth for weth and approved cdpro to spend it
	
	WETH.mint(user, etherAmount);
	
	WETH.approve(address(cdpro), etherAmount);
	
	  
	
	// deposits weth into cdpro
	
	cdpro.depositCollateral(address(WETH), etherAmount);
	
	  
	
	// check balances
	
	uint256 userWethCollateral = WETH.balanceOf(address(cdpro));
	
	console2.log("User weth collateral", userWethCollateral / 1e18);
	
	  
	
	// check the usd value of the ether
	
	uint256 valueOfWETHCollateral = cdpro.getUsdValueOfCollateral(
	
	user,
	
	etherAmount,
	
	address(WETH)
	
	);
	
	console2.log("Value of Weth collateral:", valueOfWETHCollateral/1e18);
	
	  
	
	// // check how much cdpro can be borrowed
	
	uint256 howMuchCanUserBorrowWeth = cdpro.howMuchCanUserBorrowWETH();
	
	console2.log(
	
	"How much cdpro can I borrow for eth:",
	
	howMuchCanUserBorrowWeth / 1e18
	
	);
	
	  
	
	console2.log("Total borrowable cdpro:", howMuchCanUserBorrowWeth/1e18);
	
	cdpro.borrow(cdproToBorrow);
	
	uint256 userBorrowedAmount = cdpro.getPositionForIndividualUser(user);
	
	console2.log("User borrowed amount", userBorrowedAmount / 1e18);
	
	  
	
	updateOracle_WethPriceDrop();
	
	  
	  
	
	/*
	
	position = 1400 cdpro
	
	collateral = 1 weth
	
	collateral value = $2000
	
	ltv = 70%
	
	available collateral = 2000 * 70% = $1400
	
	---------- price drop ----------
	
	position = 1400 cdpro
	
	collateral = 1 weth
	
	collateral value = $1999
	
	ltv = 70%
	
	available collateral = 1999 * 70% = $1399.3
	
	  
	
	---------- liquidate ----------
	
	breaks ltv so we should be able to liquidate
	
	alice liquidates by paying 1400 cdpro
	
	liquidatation reward = 1400 * 0.05 = 70 cdpro
	
	alice gets $1470 in weth
	
	bobs collateral = $1999 - $1470 = $529 in weth
	
	*/
	
	  
	// the liquidation reverts, thus an 
	// unhealthy position is still active
	vm.expectRevert();
	
	cdpro.canLiquidate(user);
	
	  

}
```
  
  

#### Description:
The liquidators would not be able to `liquidate` certain unhealthy positions on the protocol. This is due to the fact that the `liquidation` function is not feature complete, and has a myriad of issues, such as underflows, incorrect/missing mathematical calculations and no fee considerations. 

#### Recommendation:
We recommend that the function should be retested after all the features have been added and thoroughly tested. 
  
#### Resolution:

  


----
### <a id="C03"></a> C-03 Missing ownership checks on `GovToken`

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/GovToken.sol#L5-L17

#### PoC:
```solidity
function test_mint_gov_token() public {

	address userOne = makeAddr("userOne");
	
	address userTwo = makeAddr("userTwo");
	
	address userThree = makeAddr("userThree");
	
	address userFour = makeAddr("userFour");
	
	  
	
	// userOne
	
	vm.startPrank(userOne);
	
	govTkn.mint(userOne, 1000);
	
	assertEq(govTkn.balanceOf(userOne), 1000);
	
	vm.stopPrank();
	
	  
	
	// userTwo
	
	vm.startPrank(userTwo);
	
	govTkn.mint(userTwo, 1000);
	
	assertEq(govTkn.balanceOf(userTwo), 1000);
	
	vm.stopPrank();
	
	  
	
	// userThree
	
	vm.startPrank(userThree);
	
	govTkn.mint(userThree, 1000);
	
	assertEq(govTkn.balanceOf(userThree), 1000);
	
	vm.stopPrank();
	
	  
	
	// userFour
	
	vm.startPrank(userFour);
	
	govTkn.mint(userFour, 1000);
	
	assertEq(govTkn.balanceOf(userFour), 1000);
	
	vm.stopPrank();

}
```
  
  

#### Description:
The `GovToken.sol` contract does not have `ownership` or `role` checks to ensure that only privilege users/contracts can mint governance tokens. The current architecture allows any user to `mint` any amount of governance tokens for themselves.

#### Recommendation:
Consider adding access control or ownership checks to the contract to ensure the only privilege users or contracts can `mint` or `burn` tokens. 
  
#### Resolution:

  


----
### <a id="C04"></a> C-04 Withdraw collateral doesn't take all collateral tokens into account

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/CDPro.sol#L168C2-L191

#### PoC:
```solidity
function test_withdraw_underflow() public {

	// set up
	
	uint256 etherAmount = 1 ether;
	
	uint256 usdcAmount = 20_000 * 1e6;
	
	uint256 cdproToBorrow = 12_001 * 1e18;
	
	address user = makeAddr("user");
	
	vm.deal(user, etherAmount);
	
	// ---
	
	  
	
	vm.startPrank(user);
	
	// this swaps our eth for weth and approved cdpro to spend it
	
	WETH.mint(user, etherAmount);
	
	WETH.approve(address(cdpro), etherAmount);
	
	  
	
	USDC.mint(user, usdcAmount);
	
	USDC.approve(address(cdpro), usdcAmount);
	
	  
	
	// deposits weth into cdpro
	
	cdpro.depositCollateral(address(WETH), etherAmount);
	
	  
	
	// deposits usdc into cdpro
	
	cdpro.depositCollateral(address(USDC), usdcAmount);
	
	  
	
	// check balances
	
	uint256 userWethCollateral = WETH.balanceOf(address(cdpro));
	
	uint256 userUsdcCollateral = USDC.balanceOf(address(cdpro));
	
	console2.log("User weth collateral", userWethCollateral / 1e18);
	
	console2.log("User Usdc collateral", userUsdcCollateral / 1e6);
	
	  
	
	// check the usd value of the ether
	
	uint256 valueOfWETHCollateral = cdpro.getUsdValueOfCollateral(
	
	user,
	
	etherAmount,
	
	address(WETH)
	
	);
	
	  
	
	// check the usd value of the usdc
	
	uint256 valueOfUSDCCollateral = cdpro.getUsdValueOfCollateral(
	
	user,
	
	usdcAmount,
	
	address(USDC)
	
	);
	
	console2.log("Value of Weth collateral:", valueOfWETHCollateral/1e18);
	
	console2.log("Value of Usdc collateral:", valueOfUSDCCollateral/1e18);
	
	  
	
	// // check how much cdpro can be borrowed
	
	uint256 howMuchCanUserBorrowWeth = cdpro.howMuchCanUserBorrowWETH();
	
	console2.log(
	
	"How much cdpro can I borrow for eth:",
	
	howMuchCanUserBorrowWeth / 1e18
	
	);
	
	  
	
	uint256 howMuchCanUserBorrowUsdc = cdpro.howMuchCanUserBorrowUSDC();
	
	console2.log(
	
	"How much cdpro can I borrow for usdc:",
	
	howMuchCanUserBorrowUsdc / 1e18
	
	);
	
	  
	
	console2.log("Total borrowable cdpro:", (howMuchCanUserBorrowWeth + howMuchCanUserBorrowUsdc)/1e18);
	
	cdpro.borrow(cdproToBorrow);
	
	uint256 userBorrowedAmount = cdpro.getPositionForIndividualUser(user);
	
	console2.log("User borrowed amount", userBorrowedAmount / 1e18);
	
	vm.expectRevert();
	
	cdpro.withdrawCollateral(address(WETH), 1);
	
	  
	
	// what's happening here is the user is trying to withdraw 1 weth, but the function
	
	// doesn't take the usdc collateral into account when checking if the user has enough collateral
	
	// to cover the amount of cdpro borrowed
	
	  
	
	// user position = 12_001 cdpro
	
	// user collateral = 1 weth + 20_000 usdc
	
	// user collateral value = $2000 in weth and $20_0000 in usdc = 22000 usd
	
	// Taking ltv into account the user should be able to borrow $1400(weth) & $16_000(usdc) = $17400
	
	// the postion is only $12_001 so the user should be able to withdraw 1 weth
	
	// if the user could withdraw 1 weth, the borrowable amount is $16_000 which is still
	
	// way higer than the $12_001 borrowed, the ltv (80%) for usdc should mean that
	
	// the user should be able to borrow $12_800 cdpro, so the position is still healthy
	
	  
	
	// the account is done like this hence the underflow:
	
	// availableCollateral = userCollateralTotalUsdBalance - ((userBorrowedAmount * collateralTypes[collateralToken].ltv) / 100);
	
	// availableCollateral = 2000 - (12001*70)/100
	
	// availableCollateral = 2000 - 8400.7
	
	// availableCollateral = -6400.7 <- UNDERFLOW

}
```
  
  

#### Description:
The protocol does not take the full `collateral` value into account when confirming if a user is allowed to withdraw `collateral`. The user's `collateral` could get stuck as it would force users to settle their debts to withdraw `collateral` even if the remaining collateral is enough to cover the position. 

Consider the following scenario:

- Bob deposits 1 WETH as collateral
- Bob deposits 20_000 USDC as collateral
- Bob should be able to borrow 1400 (WETH LTV%) + 16_000(USDC LTV%) = 17_400 CDPro
- Bob borrows 12_001 CDPro
- Bob's wants to withdraw 1 WETH
- After the withdrawal, Bob's position would be healthy as they can borrow up to 16_000 CDPro with only the USDC collateral
- Bob should be able to withdraw the 1 WETH, but the protocol does not allow this

#### Recommendation:
When calculating if the remaining collateral is sufficient to cover the debt, take the value and LTV ratio of every token into account.

#### Resolution:
### <a id="C05"></a> C-05 Incorrect fee calculation

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/CDPro.sol#L468

#### PoC:
```solidity
function test_multiple_borrows_not_working() public {
	
	// set up
	
	uint256 depositAmount = 10 ether;
	
	uint256 cdproToBorrow = 1 * 1e18;
	
	uint256 secondCdproToBorrow = 1 * 1e18;
	
	address user = makeAddr("user");
	
	vm.deal(user, depositAmount);
	
	// ---
	
	  
	
	vm.startPrank(user);
	
	// this swaps our eth for weth and approved cdpro to spend it
	
	WETH.mint(user, depositAmount);
	
	WETH.approve(address(cdpro), depositAmount);
	
	  
	
	// deposits weth into cdpro
	
	cdpro.depositCollateral(address(WETH), depositAmount);
	
	  
	
	// check balances
	
	uint256 userWethCollateral = WETH.balanceOf(address(cdpro));
	
	console2.log("User weth collateral", userWethCollateral / 1e18);
	
	  
	
	// check the usd value of the ether
	
	uint256 valueOfCollateral = cdpro.getUsdValueOfCollateral(
	
	user,
	
	depositAmount,
	
	address(WETH)
	
	);
	
	console2.log("Value of the collateral", valueOfCollateral / 1e18);
	
	  
	
	// check how much cdpro can be borrowed
	
	uint256 howMuchCanUserBorrow = cdpro.howMuchCanUserBorrowWETH();
	
	console2.log(
	
	"How much cdpro can I borrow:",
	
	howMuchCanUserBorrow / 1e18
	
	);
	
	  
	
	// try and borrow 1 cdpro (1e18)
	
	cdpro.borrow(cdproToBorrow);
	
	uint256 userBorrowedAmount = cdpro.getPositionForIndividualUser(user);
	
	console2.log("1st Borrow", userBorrowedAmount / 1e18);
	
	  
	
	vm.stopPrank();
	
	vm.warp(94_608_000 seconds);
	
	updateOracle();
	
	console2.log("Time warped by 3 years");
	
	  
	
	// try and borrow 1 cdpro again (1e18)
	
	console2.log("Transaction reverts after we attempt to borrow 2 cdpro");
	
	vm.expectRevert();
	
	cdpro.borrow(secondCdproToBorrow);

}
```

#### Description:
The `_updatePositionInterest` adds a 36 decimal amount to the `borrowedAmount` when updating the interest of a position for fee calculations. This is then used in `borrow` and `repay` where an 18 decimal amount is expected. This causes underflows and unintended `reverts` in the `borrow` function when users attempt to borrow `CDPro` against any position with a `borrowed` amount. Users would not be able to increase the amount of `CDPro` borrowed until the position is settled. This issue also affects every function where `_updatePositionInterest` is called, such as `repay` & `withdrawCollateral`.

  
#### Recommendation:
Divide the `interestAmount` by `1e18` before adding the interest to the `userPosition.borrowedAmount`.

#### Resolution:
-----------------

## <a id="High"></a> High
### <a id="H01"></a> H-01 Voters can `vote` multiple times

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/Gov.sol#L108-L122

#### PoC:

```solidity
function test_propose_double_vote() public {

	// set up --->
	
	address alice = makeAddr("alice");
	
	address bob = makeAddr("bob");
	
	uint256 _ltv = 60;
	
	// alice
	
	vm.startPrank(alice);
	
	govTkn.mint(alice, 1);
	
	govTkn.approve(address(gov), 1);
	
	vm.stopPrank();
	
	// bob
	
	  
	
	// mint 500 token
	
	vm.startPrank(bob);
	
	govTkn.mint(bob, 500);
	
	govTkn.approve(address(gov), 200);
	
	vm.stopPrank();
	
	// -----
	
	  
	
	// propose with a single token
	
	vm.startPrank(alice);
	
	gov.propose(address(WETH), _ltv);
	
	(address proposer,,,,,,) = gov.getProposal(1);
	
	vm.stopPrank();
	
	  
	
	vm.warp(10 days);
	
	  
	
	vm.startPrank(bob);
	
	gov.vote(0);
	
	vm.stopPrank();
	
	vm.startPrank(bob);
	
	gov.vote(0);
	
	vm.stopPrank();
	
	// the votes for the proposal sits at 1000

}
```
  
  

#### Description:
The same user can call the `vote` function multiple times in `Gov.sol`. This is due to the fact that the `vote` function does not check if a user has already voted. 

#### Recommendation:
Consider adding checks such as a mapping that tracks if the user has already voted for the proposal.:`mapping(uint256 proposalId => address user) public voted;`.
  
#### Resolution:

  
  
  
----
### <a id="H02"></a> H-02 Users with <1% governance tokens can propose

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/Gov.sol#L108-L123C6

#### PoC:
```solidity
function test_propose_with_one_token() public {

	// set up --->
	
	address alice = makeAddr("alice");
	
	address bob = makeAddr("bob");
	
	uint256 _ltv = 60;
	
	// alice
	
	vm.startPrank(alice);
	
	// mint one token for alice
	
	govTkn.mint(alice, 1);
	
	govTkn.approve(address(gov), 1);
	
	vm.stopPrank();
	
	  
	  
	
	// mint 500 tokens for bob
	
	vm.startPrank(bob);
	
	govTkn.mint(bob, 500);
	
	govTkn.approve(address(gov), 500);
	
	vm.stopPrank();
	
	// -----
	
	// since alice only has one token and bob 500
	
	// she as less than 1% of the total supply
	
	// she should not be allowed to propose
	
	vm.startPrank(alice);
	
	gov.propose(address(WETH), _ltv);
	
	(address proposer,,,,,,) = gov.getProposal(0);
	
	vm.stopPrank();
	
	  
	
	uint256 totalSupply = govTkn.totalSupply();
	
	console2.log("TotalSupply:", totalSupply);
	
	assertEq(totalSupply, 501);
	
	  
	
	console2.log("Proposer", proposer);
	
	assertEq(address(alice), proposer);

}
```
  

#### Description:
Referencing the official specification: "Holders of the governance token with more than 1% of the `totalSupply` will be able to submit proposals for a given `collateralToken` and `ltv` configuration.", it is however possible for users with less than 1% of the total supply to submit proposals. This is possible due to missing validation that checks the balance of the proposer compared to the `totalSupply()`.

#### Recommendation:
Check the balance of the user compared to the `totalSupply()` before creating a proposal;

#### Resolution:

-----------------

## <a id="Medium"></a> Medium
### <a id="M01"></a> M-01 Propose doesn't automatically count the proposer's votes

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/Gov.sol#L83

#### Description:
The propose function does not follow the intended behaviour as noted by the natspec: `@notice proposer automatically votes for the proposal with their entire balance.`. The `votesFor` variable is currently set to zero when the `proposal` is created which means that the proposer would have to call the `vote` function after the `proposal` is made.

```solidity
	Proposal memory proposal = Proposal({
	proposer: msg.sender,
	collateralToken: _collateralToken,
	ltv: _ltv,
	votesFor: 0,
	startTime: block.timestamp,
	endTime: block.timestamp + 5 days,
	active: true
});
```
#### Recommendation:
Add the user balance to the `votesFor` when the `proposal` is created. 
  
#### Resolution:

----
### <a id="M02"></a> M-02 Users can vote for proposals after the `endTime`

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/Gov.sol#L108C2-L123C6

#### Description:
The `vote` function does not check if the `endTime` of the proposal has passed. This means that users can vote for proposals after the `endTime`. Since there is no `execute` implementation or documentation protocol the impact is speculative, but users can influence the `proposal` results after the `endTime` has been reached.
  
```
function test_propose_vote_after_end() public {

	// set up --->
	
	address alice = makeAddr("alice");
	
	address bob = makeAddr("bob");
	
	uint256 _ltv = 60;
	
	// alice
	
	vm.startPrank(alice);
	
	govTkn.mint(alice, 1);
	
	govTkn.approve(address(gov), 1);
	
	vm.stopPrank();
	
	// bob
	
	  
	
	// mint 500 token
	
	vm.startPrank(bob);
	
	govTkn.mint(bob, 500);
	
	govTkn.approve(address(gov), 200);
	
	vm.stopPrank();
	
	// -----
	
	  
	
	// propose with a single token
	
	vm.startPrank(alice);
	
	gov.propose(address(WETH), _ltv);
	
	(address proposer,,,,,,) = gov.getProposal(1);
	
	vm.stopPrank();
	
	  
	
	vm.warp(10 days);
	
	  
	
	vm.startPrank(bob);
	
	gov.vote(0);
	
	vm.stopPrank();

}
```
  

#### Recommendation:
Consider adding a check in the `vote` function that checks if the proposal has ended. 

#### Resolution:

  

-----------------
### <a id="M03"></a> M-03 Native tokens can get stuck in the protocol


https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/CDPro.sol#L131-L133

#### Description:
It is possible for native tokens to get stuck in the protocol. The `receive` and `fallback` functions are `payable`, but empty. If a user sends native ETH to the contract the contract will not revert and accept the ether. The native ether will be stuck in the contract.
  
  
``` solidity
function test_native_tokens_stuck() public {

	// set up
	
	uint256 depositAmount = 2 ether;
	
	address user = makeAddr("user");
	
	vm.deal(user, depositAmount);
	
	// ---
	
	  
	
	vm.startPrank(user);
	
	// transfer native eth to cdpro
	
	(bool success, ) = address(cdpro).call{value: 1 ether}("");
	
	require(success, "Failed to send Ether");
	
	vm.stopPrank();
	
	  
	
	// user balance dropped after
	
	uint256 userBalanceNativeEth = user.balance;
	
	console2.log(
	
	"User balance after transfer",
	
	userBalanceNativeEth / 1e18
	
	);
	
	assertEq(userBalanceNativeEth, 1 ether);

}
```
  

#### Recommendation:
  Implement the `receive` and `fallback` functions to wrap the native ETH and open a position for the user (`depositCollateral`) . 
  
  
  

#### Resolution:

  

-----------------
### <a id="M04"></a> M-04 Centralization risk

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/CDPro.sol#L235
https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/CDPro.sol#L115C52-L115C52

#### Description:
The `deployer` of the contract can set the `LTV` values to any value as the `setCollateralTokenLTV` function has an `onlyOwner` modifier. This supersedes the `governance` proposals. It's also worth noting that `CDPro.sol` uses the `Ownable(msg.sender)` in the constructor. If the `msg.sender` is not a multisig and the `owner/deployer` loses access to their private key or accidentally revokes their ownership it could have unintended consequences. We increased the severity to a medium due to the fact that there is currently no way to update the `LTV` via the `Gov.sol` contract, thus if the `owner/deployer` loses access to their private key, it wouldn't be possible to update the `LTV` for the collateral tokens in the protocol. 

#### Recommendation:
- Ensure the protocol uses a two step ownership process
- Consider using a multisig
- Consider adding an execution pipeline from `Gov.sol` to update the `LTV` and remove the `setCollateralTokenLTV` function. 

#### Resolution:
____

## <a id="Low"></a> Low
### <a id="L01"></a> L-01 Missing event for the liquidate function

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/CDPro.sol#L493-L549

#### Description:
The `liquidate` function does not emit an event to record on chain changes when called successfully. Events could potentially be used to notify users when on chain events occur, a case could be made that users may miss critical changes(such as liquidating their positions) when events are omitted. 

#### Recommendation:
Consider adding a liquidation event. Example: `emit PositionLiquidated(user, "Position Liquidated at:", block.timestamp);`
  
#### Resolution:

----
### <a id="L02"></a> L-02 Unnecessary operation in propose

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/Gov.sol#L98-L99

#### Description:
The `proposalId` in the `Propose` function for the `Gov.sol` contract increments the `proposalId` by one, then decreases the `proposalId` by -1 before returning the variable. The same end state could be accomplished by returning `return proposalId++;`. This would improve the readability and remove the unnecessary added complexity.
  
#### Recommendation:
Replace `proposalId += 1;`&`return proposalId-1;` with `return proposalId++;`.

#### Resolution:

____
### <a id="L03"></a> L-03 Imported frameworks that are not in use

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/CDPro.sol#L10

#### Description:
Importing libraries and frameworks that are not being used could cause significant increases on gas cost for the protocol. In this case the `forge` testing framework was imported, which could also lead to unintended behaviour for the protocol.

#### Recommendation:
Remove `import "../lib/forge-std/src/Test.sol";`

#### Resolution:

____
### <a id="L04"></a> L-04 Floating pragma

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/CDPro.sol#L2C25-L2C25
https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/Gov.sol#L3
https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/GovToken.sol#L3C1-L3C1
https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/727d97a924b75c76c896d71661909b96e6a8ce57/src/StableCoin.sol#L2
  
#### Description:
Contracts should be deployed with the same compiler version and flags used during development and testing. Locking the pragma helps to ensure that contracts do not accidentally get deployed using another pragma. For example, an outdated pragma version might introduce bugs that affect the contract system negatively or recently released pragma versions may have unknown security vulnerabilities.

#### Recommendation:
Consider locking the pragma to the latest solidity version 0.8.23`

#### Resolution:
____  
### <a id="L05"></a> L-05 Parameter not in use

https://github.com/owenThurm/CDPro-maxgrok-sheepghosty/blob/e46e02a4a5bd652caad6bbce0438158b607bfca6/src/CDPro.sol#L427
  
#### Description:
The `address user` parameter is not in use and could be removed. 

#### Recommendation:
Remove the `address user` parameter.

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