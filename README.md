# Lambo.win audit details
- Total Prize Pool: $22,500 in USDC
  - HM awards: $17,952 in USDC
  - QA awards: $748 in USDC
  - Judge awards: $2,000 in USDC
  - Scout awards: $500 in USDC
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts November 27, 2024 20:00 UTC
- Ends December 4, 2024 20:00 UTC

**Note re: risk level upgrades/downgrades**

Two important notes about judging phase risk adjustments: 
- High- or Medium-risk submissions downgraded to Low-risk (QA)) will be ineligible for awards.
- Upgrading a Low-risk finding from a QA report to a Medium- or High-risk finding is not supported.

As such, wardens are encouraged to select the appropriate risk level carefully during the submission phase.

## Automated Findings / Publicly Known Issues

The 4naly3er report can be found [here](https://github.com/code-423n4/2024-11-lambowin/blob/main/4naly3er-report.md).

_Note for C4 wardens: Anything included in this `Automated Findings / Publicly Known Issues` section is considered a publicly known issue and is ineligible for awards._

1. Rebalance Contract ignore gas cost
The RebalanceOnUniswap contract is designed to maintain the VETH/WETH pool ratio at 1:1 rather than for profit. Gas costs are intentionally omitted to increase rebalancing frequency, accepting gas losses as a trade-off for improved price stability.

2. repayLoan function not used
The repayLoan function in the VirtualToken contract is currently not used. This function is designed to allow valid factories to repay loans by decreasing the debt of a user and 1. burning the corresponding amount of tokens.

We hope to provide services for subsequent liquidity additions. If the withdrawable feature of virtual liquidity can be realized, it will be able to solve the impermanent loss problem of Uniswap V2. We plan to release LamboFactory V2 and conduct audit again.

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

# Overview

[ ⭐️ SPONSORS: add info here ]

## Links

- **Previous audits:**  https://github.com/Lambo-Win/Lambo-Virtual-Liquidity-Code4rena/blob/main/audit/SlowMistAudit.pdf
  - ✅ SCOUTS: If there are multiple report links, please format them in a list.
- **Documentation:** https://docsend.com/view/f2tf4zkt2udaydwd
- **Website:** https://lambo.win/launchpool
- **X/Twitter:** https://x.com/lambodotwin

---

# Scope

[ ✅ SCOUTS: add scoping and technical details here ]

### Files in scope
- ✅ This should be completed using the `metrics.md` file
- ✅ Last row of the table should be Total: SLOC
- ✅ SCOUTS: Have the sponsor review and and confirm in text the details in the section titled "Scoping Q amp; A"

*For sponsors that don't use the scoping tool: list all files in scope in the table below (along with hyperlinks) -- and feel free to add notes to emphasize areas of focus.*

| Contract | SLOC | Purpose | Libraries used |  
| ----------- | ----------- | ----------- | ----------- |
| [contracts/folder/sample.sol](https://github.com/code-423n4/repo-name/blob/contracts/folder/sample.sol) | 123 | This contract does XYZ | [`@openzeppelin/*`](https://openzeppelin.com/contracts/) |

### Files out of scope
✅ SCOUTS: List files/directories out of scope

## Scoping Q &amp; A

### General questions
### Are there any ERC20's in scope?: No

✅ SCOUTS: If the answer above 👆 is "Yes", please add the tokens below 👇 to the table. Otherwise, update the column with "None".

### Are there any ERC777's in scope?: 

✅ SCOUTS: If the answer above 👆 is "Yes", please add the tokens below 👇 to the table. Otherwise, update the column with "None".



### Are there any ERC721's in scope?: No

✅ SCOUTS: If the answer above 👆 is "Yes", please add the tokens below 👇 to the table. Otherwise, update the column with "None".



### Are there any ERC1155's in scope?: No

✅ SCOUTS: If the answer above 👆 is "Yes", please add the tokens below 👇 to the table. Otherwise, update the column with "None".



✅ SCOUTS: Once done populating the table below, please remove all the Q/A data above.

| Question                                | Answer                       |
| --------------------------------------- | ---------------------------- |
| ERC20 used by the protocol              |       🖊️             |
| Test coverage                           | ✅ SCOUTS: Please populate this after running the test coverage command                          |
| ERC721 used  by the protocol            |            🖊️              |
| ERC777 used by the protocol             |           🖊️                |
| ERC1155 used by the protocol            |              🖊️            |
| Chains the protocol will be deployed on | Ethereum |

### ERC20 token behaviors in scope

| Question                                                                                                                                                   | Answer |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| [Missing return values](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#missing-return-values)                                                      |    |
| [Fee on transfer](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#fee-on-transfer)                                                                  |   |
| [Balance changes outside of transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#balance-modifications-outside-of-transfers-rebasingairdrops) |    |
| [Upgradeability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#upgradable-tokens)                                                                 |    |
| [Flash minting](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#flash-mintable-tokens)                                                              |    |
| [Pausability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#pausable-tokens)                                                                      |    |
| [Approval race protections](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#approval-race-protections)                                              |    |
| [Revert on approval to zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-approval-to-zero-address)                            |    |
| [Revert on zero value approvals](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-approvals)                                    |    |
| [Revert on zero value transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                    |    |
| [Revert on transfer to the zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-transfer-to-the-zero-address)                    |    |
| [Revert on large approvals and/or transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-large-approvals--transfers)                  |    |
| [Doesn't revert on failure](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#no-revert-on-failure)                                                   |    |
| [Multiple token addresses](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                          |    |
| [Low decimals ( < 6)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#low-decimals)                                                                 |    |
| [High decimals ( > 18)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#high-decimals)                                                              |    |
| [Blocklists](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#tokens-with-blocklists)                                                                |    |

### External integrations (e.g., Uniswap) behavior in scope:


| Question                                                  | Answer |
| --------------------------------------------------------- | ------ |
| Enabling/disabling fees (e.g. Blur disables/enables fees) | Yes   |
| Pausability (e.g. Uniswap pool gets paused)               |  No   |
| Upgradeability (e.g. Uniswap gets upgraded)               |   No  |


### EIP compliance checklist
N/A

✅ SCOUTS: Please format the response above 👆 using the template below👇

| Question                                | Answer                       |
| --------------------------------------- | ---------------------------- |
| src/Token.sol                           | ERC20, ERC721                |
| src/NFT.sol                             | ERC721                       |


# Additional context

## Main invariants

The the owner of VETH is invariants.

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

## Attack ideas (where to focus for bugs)
1.  The V3 Liquidity (VETH <-> WETH)'s safety
2. The ETH locked in VETH's safety
3. The complicated scenario with unsiwapV2, V3 and VETH's take loan 

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

## All trusted roles in the protocol

Uniswap V2 and Uniswap V3

✅ SCOUTS: Please format the response above 👆 using the template below👇

| Role                                | Description                       |
| --------------------------------------- | ---------------------------- |
| Owner                          | Has superpowers                |
| Administrator                             | Can change fees                       |

## Describe any novel or unique curve logic or mathematical models implemented in the contracts:

Virtual liquidity is a conditional ERC20 token. For each holder address of virtual liquidity, the transferable balance must satisfy the following formula to ensure that the issued virtual liquidity does not enter the market.

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

## Running tests

```
git clone git@github.com:Lambo-Win/Lambo-Virtual-Liquidity-Code4rena.git
forge build
forge test
```


✅ SCOUTS: Please format the response above 👆 using the template below👇

```bash
git clone https://github.com/code-423n4/2023-08-arbitrum
git submodule update --init --recursive
cd governance
foundryup
make install
make build
make sc-election-test
```
To run code coverage
```bash
make coverage
```
To run gas benchmarks
```bash
make gas
```

✅ SCOUTS: Add a screenshot of your terminal showing the gas report
✅ SCOUTS: Add a screenshot of your terminal showing the test coverage

## Miscellaneous
Employees of Lambo.win and employees' family members are ineligible to participate in this audit.

Code4rena's rules cannot be overridden by the contents of this README. In case of doubt, please check with C4 staff.




# Scope

*See [scope.txt](https://github.com/code-423n4/2024-11-lambowin/blob/main/scope.txt)*

### Files in scope


| File   | Logic Contracts | Interfaces | nSLOC | Purpose | Libraries used |
| ------ | --------------- | ---------- | ----- | -----   | ------------ |
| /src/LamboFactory.sol | 1| **** | 58 | |@openzeppelin/contracts/proxy/Clones.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol<br>@openzeppelin/contracts/access/Ownable.sol<br>@openzeppelin/contracts/utils/ReentrancyGuard.sol|
| /src/LamboToken.sol | 1| **** | 122 | |@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol<br>@openzeppelin/contracts/utils/Context.sol<br>@openzeppelin/contracts/interfaces/draft-IERC6093.sol<br>@openzeppelin/contracts/access/Ownable.sol|
| /src/LamboVEthRouter.sol | 1| **** | 115 | |@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol<br>@openzeppelin/contracts/access/Ownable.sol<br>@openzeppelin/contracts/utils/ReentrancyGuard.sol|
| /src/VirtualToken.sol | 1| **** | 117 | |@openzeppelin/contracts/token/ERC20/ERC20.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol<br>@openzeppelin/contracts/access/Ownable.sol|
| /src/rebalance/LamboRebalanceOnUniwap.sol | 1| **** | 131 | |@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol<br>@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol<br>@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol<br>@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol<br>@morpho/interfaces/IMorpho.sol<br>@morpho/interfaces/IMorphoCallbacks.sol|
| /src/Utils/LaunchPadUtils.sol | 1| **** | 10 | ||
| **Totals** | **6** | **** | **553** | | |

### Files out of scope

*See [out_of_scope.txt](https://github.com/code-423n4/2024-11-lambowin/blob/main/out_of_scope.txt)*

| File         |
| ------------ |
| ./script/0.deployTokens.s.sol |
| ./script/1.deployOthers.s.sol |
| ./script/2.ownerTransfer.s.sol |
| ./script/3.deployRebalacne.s.sol |
| ./script/quoter/DeployQuoter.s.sol |
| ./script/quoter/LamboMemeQuoter.sol |
| ./script/quoter/LamboQuoterForAggregator.sol |
| ./script/quoter/LamboQuoterPathFor1inchV6.sol |
| ./src/interfaces/1inchV6/IAggregator.sol |
| ./src/interfaces/Curve/IStableNGFactory.sol |
| ./src/interfaces/Curve/IStableNGPool.sol |
| ./src/interfaces/IApprove.sol |
| ./src/interfaces/IApproveProxy.sol |
| ./src/interfaces/IFactory.sol |
| ./src/interfaces/ILaunchpad.sol |
| ./src/interfaces/IPool.sol |
| ./src/interfaces/IPoolFactory.sol |
| ./src/interfaces/IPoolRegistery.sol |
| ./src/interfaces/IRouter.sol |
| ./src/interfaces/IWETH.sol |
| ./src/interfaces/OKX/IDexRouter.sol |
| ./src/interfaces/Uniswap/ILiquidityManager.sol |
| ./src/interfaces/Uniswap/INonfungiblePositionManager.sol |
| ./src/interfaces/Uniswap/IPool.sol |
| ./src/interfaces/Uniswap/IPoolFactory.sol |
| ./src/interfaces/Uniswap/IPoolInitializer.sol |
| ./src/interfaces/Uniswap/IQuoter.sol |
| ./src/interfaces/Uniswap/IUniswapV2Router01.sol |
| ./src/interfaces/Uniswap/IUniswapV3Pool.sol |
| ./src/libraries/1inchV6.sol |
| ./src/libraries/AddressLib.sol |
| ./src/libraries/ProtocolLib.sol |
| ./src/libraries/UniswapV2Library.sol |
| ./test/BaseTest.t.sol |
| ./test/GeneralTest.t.sol |
| ./test/LiquidityManage.t.sol |
| ./test/RebalanceTest.t.sol |
| ./test/tools/TestLamboMemeQuoter.t.sol |
| ./test/tools/TestLamboQuoterForAggregator.t.sol |
| Totals: 39 |

