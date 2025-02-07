# Lambo.win Audit
- Date
    - 03 Dec 2024 - 10 Dec 2024  
- Scope
    - ./src/LamboFactory.sol
    - ./src/LamboToken.sol
    - ./src/LamboVEthRouter.sol
    - ./src/VirtualToken.sol
    - ./src/rebalance/LamboRebalanceOnUniwap.sol
    - ./src/Utils/LaunchPadUtils.sol
    - ./src/libraries/UniswapV2Library.sol
- Findings
    - High: 1  
    [H-01: Incorrect amount used for minting in `cashIn()`](#high-01) 
    - Medium: 1    
    [M-01: `_buyQuote` retains 1 wei in the contract for certain transactions, leading to fund accumulated to be stucked](#med-02)  
    - Low: 7  
    [L-01: Floating pragma used in several contracts](#low-01)  
    [L-02: Inconsistent use of _msgSender() and msg.sender](#low-02)  
    [L-03: Efficient use of constants from LaunchPadUtils.sol and remove duplicated constant variable](#low-03)  
    [L-04: `updateFeeRate()` lacks input validation](#low-04)  
    [L-05: Unnecessary casting of variable to address](#low-05)  
    [L-06: Unnecessary check inside `_transferAssetFromUser()`](#low-06)  
    [L-07: Misleading error message in `_getTokenInOut()`](#low-07)  
- Tools
		- [Foundry](https://github.com/foundry-rs/foundry)

# Findings

## <a id="high-01"></a>H-01: Incorrect amount used for minting in `cashIn()`

### Summary
If the underlyingToken is not Ether, it is suppose to transfer the amount of underlyingToken (Other ERC20 token) from user to the contract. However, in such scenario, the amount that is minted uses msg.value instead of the given input amount and this causes wrong amount to be minted and the transferred ERC20 token to be lost as `cashOut()` would revert.

### Vulnerability Details
Affected [code](https://github.com/code-423n4/2024-12-lambowin/blob/874fafc7b27042c59bdd765073f5e412a3b79192/src/VirtualToken.sol#L78)

1. If underlyingToken is not Ether, it calls `_transferAssetFromUser()` and transfer the amount of underlyingToken(other ERC20 token) from user to contract.
2. When it completes, `_mint()` is called but the minted amount uses msg.value instead of amount. In the event msg.value could be 0, a user can transfer 5 underlyingToken and get 0 minted because msg.value is 0.


```
function cashIn(uint256 amount) external payable onlyWhiteListed {
    if (underlyingToken == LaunchPadUtils.NATIVE_TOKEN) {
        require(msg.value == amount, "Invalid ETH amount");
    } else {
        _transferAssetFromUser(amount); 
    }
    _mint(msg.sender, msg.value);
    emit CashIn(msg.sender, msg.value);
}

function _transferAssetFromUser(uint256 amount) internal {
    if (underlyingToken == LaunchPadUtils.NATIVE_TOKEN) {
        require(msg.value >= amount, "Invalid ETH amount");
    } else {
        IERC20(underlyingToken).safeTransferFrom(msg.sender, address(this), amount);
    }
}
```

### Impact/Proof of Concept
This will cause wrong amount of tokens to be minted if user uses other ERC20 token. In the following test case, we test a user cash in 5 USDC token to mint 5 vUSDC. However, due to the vulnerability, no vUSDC was minted. The 5 USDC tokens that were cashed in will be lost, as there were no vUSDC minted. Hence, cashOut() will also revert.


*Put this test in GeneralTest.t.sol*
```solidity
contract MockERC20 is ERC20 {
    uint8 private _decimals;

    constructor(string memory name, string memory symbol, uint8 decimals_) ERC20(name, symbol) {
        _decimals = decimals_;
    }

    function mint(address to, uint256 amount) external {
        _mint(to, amount);
    }

    function decimals() public view override returns (uint8) {
        return _decimals;
    }
}

function test_cashInMintWrongAmount() public {
    address owner = makeAddr("owner");
    vm.startPrank(owner);
    uint256 INITIAL_SUPPLY = 100 * 10**6;
    uint256 USDC_AMOUNT = 5 * 10**6;

    // Deploy a mock USDC token
    MockERC20 usdcToken = new MockERC20("USDC", "USDC", 6);
    usdcToken.mint(owner, INITIAL_SUPPLY);

    // Deploy VirtualToken using USDC as the underlying token
    VirtualToken vToken = new VirtualToken("vUSDC", "vUSDC", address(usdcToken));
    address underlyingToken = vToken.underlyingToken();
    require(underlyingToken == address(usdcToken), "Underlying token mismatch");

    // Add to whitelist so the owner can call cashIn()
    vToken.addToWhiteList(owner);
    
    // Print before balances of USDC and vUSDC
    console2.log("USDC beforeBalance:", usdcToken.balanceOf(owner));
    console2.log("vUSDC beforeBalance:", vToken.balanceOf(owner));

    // Approve and cash in 5 USDC tokens
    usdcToken.approve(address(vToken), USDC_AMOUNT);
    vToken.cashIn(USDC_AMOUNT);

    // Print after balances of USDC and vUSDC
    console2.log("USDC afterBalance:", usdcToken.balanceOf(owner));
    console2.log("vUSDC afterBalance:", vToken.balanceOf(owner));

    // Try cashOut() and expect revert
    vm.expectRevert(abi.encodeWithSelector(VirtualToken.DebtOverflow.selector, owner, 0, USDC_AMOUNT));
    vToken.cashOut(USDC_AMOUNT);

    vm.stopPrank();
}
```

Results:  
After cashIn(), the vUSDC balance is still 0.
```solidity
[PASS] test_cashInMintWrongAmount() (gas: 1726387)
Logs:
  USDC beforeBalance: 100000000
  vUSDC beforeBalance: 0
  USDC afterBalance: 95000000
  vUSDC afterBalance: 0

Traces:
  [1726387] GeneralTest::test_cashInMintWrongAmount()
    ├─ [0] VM::addr(<pk>) [staticcall]
    │   └─ ← [Return] owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266]
    ├─ [0] VM::label(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], "owner")
    │   └─ ← [Return] 
    ├─ [0] VM::startPrank(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266])
    │   └─ ← [Return] 
    ├─ [438126] → new MockERC20@0x88F59F8826af5e695B13cA934d6c7999875A9EeA
    │   └─ ← [Return] 1848 bytes of code
    ├─ [46488] MockERC20::mint(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], 100000000 [1e8])
    │   ├─ emit Transfer(from: 0x0000000000000000000000000000000000000000, to: owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], value: 100000000 [1e8])
    │   └─ ← [Stop] 
    ├─ [1087246] → new VirtualToken.0.8.23@0xCeF98e10D1e80378A9A74Ce074132B66CDD5e88d
    │   ├─ emit OwnershipTransferred(previousOwner: 0x0000000000000000000000000000000000000000, newOwner: owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266])
    │   └─ ← [Return] 5082 bytes of code
    ├─ [370] VirtualToken.0.8.23::underlyingToken() [staticcall]
    │   └─ ← [Return] MockERC20: [0x88F59F8826af5e695B13cA934d6c7999875A9EeA]
    ├─ [23874] VirtualToken.0.8.23::addToWhiteList(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266])
    │   ├─ emit WhiteListAdded(user: owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266])
    │   └─ ← [Stop] 
    ├─ [552] MockERC20::balanceOf(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266]) [staticcall]
    │   └─ ← [Return] 100000000 [1e8]
    ├─ [0] console::log("USDC beforeBalance:", 100000000 [1e8]) [staticcall]
    │   └─ ← [Stop] 
    ├─ [2717] VirtualToken.0.8.23::balanceOf(GeneralTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496]) [staticcall]
    │   └─ ← [Return] 0
    ├─ [0] console::log("vUSDC beforeBalance:", 0) [staticcall]
    │   └─ ← [Stop] 
    ├─ [24339] MockERC20::approve(VirtualToken.0.8.23: [0xCeF98e10D1e80378A9A74Ce074132B66CDD5e88d], 5000000 [5e6])
    │   ├─ emit Approval(owner: owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], spender: VirtualToken.0.8.23: [0xCeF98e10D1e80378A9A74Ce074132B66CDD5e88d], value: 5000000 [5e6])
    │   └─ ← [Return] true
    ├─ [34419] VirtualToken.0.8.23::cashIn(5000000 [5e6])
    │   ├─ [25420] MockERC20::transferFrom(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], VirtualToken.0.8.23: [0xCeF98e10D1e80378A9A74Ce074132B66CDD5e88d], 5000000 [5e6])
    │   │   ├─ emit Transfer(from: owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], to: VirtualToken.0.8.23: [0xCeF98e10D1e80378A9A74Ce074132B66CDD5e88d], value: 5000000 [5e6])
    │   │   └─ ← [Return] true
    │   ├─ emit Transfer(from: 0x0000000000000000000000000000000000000000, to: owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], value: 0)
    │   ├─ emit CashIn(user: owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266], amount: 0)
    │   └─ ← [Stop] 
    ├─ [717] VirtualToken.0.8.23::balanceOf(GeneralTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496]) [staticcall]
    │   └─ ← [Return] 0
    ├─ [552] MockERC20::balanceOf(owner: [0x7c8999dC9a822c1f0Df42023113EDB4FDd543266]) [staticcall]
    │   └─ ← [Return] 95000000 [9.5e7]
    ├─ [0] console::log("USDC afterBalance:", 95000000 [9.5e7]) [staticcall]
    │   └─ ← [Stop] 
    ├─ [717] VirtualToken.0.8.23::balanceOf(GeneralTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496]) [staticcall]
    │   └─ ← [Return] 0
    ├─ [0] console::log("vUSDC afterBalance:", 0) [staticcall]
    │   └─ ← [Stop] 
    ├─ [0] VM::expectRevert(DebtOverflow(0x7c8999dC9a822c1f0Df42023113EDB4FDd543266, 0, 5000000 [5e6]))
    │   └─ ← [Return] 
    ├─ [3403] VirtualToken.0.8.23::cashOut(5000000 [5e6])
    │   └─ ← [Revert] DebtOverflow(0x7c8999dC9a822c1f0Df42023113EDB4FDd543266, 0, 5000000 [5e6])
    ├─ [0] VM::stopPrank()
    │   └─ ← [Return] 
    └─ ← [Return] 
```

### Recommendations
Use amount instead of msg.value to mint
```diff
function cashIn(uint256 amount) external payable onlyWhiteListed {
        if (underlyingToken == LaunchPadUtils.NATIVE_TOKEN) {
            require(msg.value == amount, "Invalid ETH amount");
        } else {
            _transferAssetFromUser(amount); 
        }
-        _mint(msg.sender, msg.value);
+        _mint(msg.sender, amount);
        emit CashIn(msg.sender, msg.value);
    }
```


## <a id="med-01"></a>M-01: `_buyQuote` retains 1 wei in the contract for certain transactions, leading to fund accumulated to be stucked

### Summary
When `_buyQuote` is called and if msg.value is more than amountXIn, the router contract will retain 1 wei in the contract. This accumulated ether can get stucked in the contract and owner will not be able to withdraw. Over a prolonged period, the wei can be accumulated to significant amount.

### Vulnerability Details
Affected [code](https://github.com/code-423n4/2024-12-lambowin/blob/874fafc7b27042c59bdd765073f5e412a3b79192/src/LamboVEthRouter.sol#L181)

In the following scenarios, the contract may retain 1 wei:  
**Scenario 1**  
msg.value = 100  
amountXIn = 100  
Retain 1 wei = No as msg.value is same as amountXIn  

**Scenario 2**  
msg.value = 101  
amountXIn = 100  
Retain 1 wei = Yes, as msg.value is 1 more than amountXIn but not bigger than amountXIn + 1  

**Scenario 3**  
msg.value = 102 or more  
amountXIn = 100  
Retain 1 wei = Yes, 1 wei gets retained and the excess will transferred to msg.sender

```
function _buyQuote(address quoteToken, uint256 amountXIn, uint256 minReturn) internal returns (uint256 amountYOut) {
    rest of code.....

    if (msg.value > (amountXIn + fee + 1)) {
        (bool success, ) = payable(msg.sender).call{value: msg.value - amountXIn - fee - 1}("");
        require(success, "ETH transfer failed");
    }

    emit BuyQuote(quoteToken, amountXIn, amountYOut);
}
```

### Impact/Proof of Concept
Over a prolonged period with high buy quote transaction volume, the amount of wei will increase and funds will get stuck in this contract.

### Recommendations
1. Implement a withdraw function
2. Change the function to not retain 1 wei if msg.value is more than amountXIn


## <a id="low-01"></a>L-01: Floating pragma used in several contracts

### Summary
Currently, the contract uses a floating pragma, allowing the contract to be compiled with any 0.8.x Solidity version higher than 0.8.20. The security best practice is to set the pragma to a specific version, so that the contract is not accidentally compiled to a version which breaks the contract's operability.


### Vulnerability Details
|Contracts|
|:--:|
|[./src/LamboFactory.sol](https://github.com/code-423n4/2024-12-lambowin/blob/main/src/LamboFactory.sol)|
|[./src/LamboToken.sol](https://github.com/code-423n4/2024-12-lambowin/blob/main/src/LamboToken.sol)|
|[./src/LamboVEthRouter.sol](https://github.com/code-423n4/2024-12-lambowin/blob/main/src/LamboVEthRouter.sol)|
|[./src/VirtualToken.sol](https://github.com/code-423n4/2024-12-lambowin/blob/main/src/VirtualToken.sol)|
|[./src/rebalance/LamboRebalanceOnUniwap.sol](https://github.com/code-423n4/2024-12-lambowin/blob/main/src/rebalance/LamboRebalanceOnUniwap.sol)|

### Recommendations
Set the pragma to a specific version.

## <a id="low-02"></a>L-02: Inconsistent use of _msgSender() and msg.sender

### Summary
The _mint() inside `initialize()` uses msg.sender while other parts of the protocol uses _msgSender() which may not seem consistent.

### Vulnerability Details
Affected [code](https://github.com/code-423n4/2024-12-lambowin/blob/874fafc7b27042c59bdd765073f5e412a3b79192/src/LamboToken.sol#L40)  

_mint() in `initialize()` uses msg.sender instead of _msgSender()
```
function initialize(string memory _name, string memory _symbol) public { 
    rest of code.....

    _mint(msg.sender, LaunchPadUtils.TOTAL_AMOUNT_OF_QUOTE_TOKEN);
    _transferOwnership(address(0));
}
```

### Recommendations
Change msg.sender to _msgSender() and ensure consistency

## <a id="low-03"></a>L-03: Efficient use of constants from LaunchPadUtils.sol and remove duplicated constant variable

### Summary
LamboVEthRouter.sol and LamboRebalanceOnUniwap.sol contain constant variables that:
1. Are already declared inside LaunchPadUtils.sol, hence are duplicates.
2. Should be declared inside LaunchPadUtils.sol to ensure consistency

### Vulnerability Details
1. Affected [code](https://github.com/code-423n4/2024-12-lambowin/blob/874fafc7b27042c59bdd765073f5e412a3b79192/src/LamboVEthRouter.sol#L18)  

`NATIVE_TOKEN` is already declared inside LaunchPadUtils.sol and should not be declared again inside LamboVEthRouter
```
contract LamboVEthRouter is Ownable, ReentrancyGuard {
    address public constant NATIVE_TOKEN = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;
    rest of code.....
}
```

2. Affected [code](https://github.com/code-423n4/2024-12-lambowin/blob/874fafc7b27042c59bdd765073f5e412a3b79192/src/rebalance/LamboRebalanceOnUniwap.sol#L30) 

`weth` is already declared inside LaunchPadUtils.sol and should not be declared again inside LamboRebalanceOnUniwap
```
contract LamboRebalanceOnUniwap{
    address public constant weth = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;
    address public constant morphoVault = 0xBBBBBbbBBb9cC5e90e3b3Af64bdAF62C37EEFFCb;
    address public constant quoter = 0x5e55C9e631FAE526cd4B0526C4818D6e0a9eF0e3;
    address public constant OKXRouter = 0x7D0CcAa3Fac1e5A943c5168b6CEd828691b46B36;
    address public constant OKXTokenApprove = 0x40aA958dd87FC8305b97f2BA922CDdCa374bcD7f;
    rest of code.....
}
```

3. Affected [code](https://github.com/code-423n4/2024-12-lambowin/blob/874fafc7b27042c59bdd765073f5e412a3b79192/src/rebalance/LamboRebalanceOnUniwap.sol#L31-L34) 

Constants are all declared inside LaunchPadUtils.sol, hence these constants should also be declared inside the same library for consistency: `morphoVault`, `quoter`, `OKXRouter`, `OKXTokenApprove`
```
contract LamboRebalanceOnUniwap{
    address public constant morphoVault = 0xBBBBBbbBBb9cC5e90e3b3Af64bdAF62C37EEFFCb;
    address public constant quoter = 0x5e55C9e631FAE526cd4B0526C4818D6e0a9eF0e3;
    address public constant OKXRouter = 0x7D0CcAa3Fac1e5A943c5168b6CEd828691b46B36;
    address public constant OKXTokenApprove = 0x40aA958dd87FC8305b97f2BA922CDdCa374bcD7f;
    rest of code.....
}
```
### Recommendations
Declare all constants inside LaunchPadUtils.sol and use the variables inside LaunchPadUtils.sol instead of declaring them again in individual contracts. This will provide cleaner code and consistency.


## <a id="low-04"></a>L-04: `updateFeeRate()` lacks input validation

### Summary
The `updateFeeRate()` lacks input validation. If the `newfeeRate` is 0, there will be an opportunity window for users to abuse the free buying or selling of tokens.

### Vulnerability Details
Affected [code](https://github.com/code-423n4/2024-12-lambowin/blob/874fafc7b27042c59bdd765073f5e412a3b79192/src/LamboVEthRouter.sol#L35-L39)  

`updateFeeRate()` does not validate the input to check if `newFeeRate` is zero.
```
function updateFeeRate(uint256 newFeeRate) external onlyOwner {
    require(newFeeRate <= feeDenominator, "Fee rate must be less than or equal to feeDenominator");
    feeRate = newFeeRate;
    emit UpdateFeeRate(newFeeRate);
}
```

### Recommendations
Include a check to ensure that the fee is not 0.


## <a id="low-05"></a>L-05: Unnecessary casting of `vETH` variable to address

### Summary
the `vETH` variable is already an address variable. Hence, the casting of `vETH` inside `createLaunchPadAndInitialBuy()` to address variable is unnecessary.

### Vulnerability Details
Affected [code](https://github.com/code-423n4/2024-12-lambowin/blob/874fafc7b27042c59bdd765073f5e412a3b79192/src/LamboVEthRouter.sol#L53)  

```
createLaunchPadAndInitialBuy(){
    rest of code.....
    (quoteToken, pool) = LamboFactory(lamboFactory).createLaunchPad(
        name,
        tickname,
        virtualLiquidityAmount,
        address(vETH)
    );
}
```

### Recommendations
Remove unnecessary address casting
```diff
createLaunchPadAndInitialBuy(){
    rest of code.....
    (quoteToken, pool) = LamboFactory(lamboFactory).createLaunchPad(
        name,
        tickname,
        virtualLiquidityAmount,
-        address(vETH)
+        vETH
    );
}
```


## <a id="low-06"></a>L-06: Unnecessary check inside `_transferAssetFromUser()`

### Summary
The `_transferAssetFromUser()` checks if underlyingToken == LaunchPadUtils.NATIVE_TOKEN. However, this function is only called from `cashIn()` and the check is already done beforehand. Hence, this check inside `_transferAssetFromUser()` is redundant as it will never return True.

### Vulnerability Details
Affected [code](https://github.com/code-423n4/2024-12-lambowin/blob/874fafc7b27042c59bdd765073f5e412a3b79192/src/VirtualToken.sol#L125-L126)  

(underlyingToken == LaunchPadUtils.NATIVE_TOKEN) will never return True as the check was already done in `cashIn()`
```
function cashIn(uint256 amount) external payable onlyWhiteListed {
        if (underlyingToken == LaunchPadUtils.NATIVE_TOKEN) {
            require(msg.value == amount, "Invalid ETH amount");
        } else {
            _transferAssetFromUser(amount); 
        }
        _mint(msg.sender, msg.value);
        emit CashIn(msg.sender, msg.value);
    }

function _transferAssetFromUser(uint256 amount) internal {
        if (underlyingToken == LaunchPadUtils.NATIVE_TOKEN) { 
            require(msg.value >= amount, "Invalid ETH amount");
        } else {
            IERC20(underlyingToken).safeTransferFrom(msg.sender, address(this), amount);
        }
    }
```

### Recommendations
Remove the unnecessary check of (underlyingToken == LaunchPadUtils.NATIVE_TOKEN) in `_transferAssetFromUser()`


## <a id="low-07"></a>L-07: Misleading error message in `_getTokenInOut()`

### Summary
In the scenario of no rebalancing needed (equal balance of wethBalance and vethBalance) when users call `previewRebalance()`, the `_getTokenInOut()` function reverts with error message "amountIn must be greater than zero", which can be misleading to users.

### Vulnerability Details
Affected [code](https://github.com/code-423n4/2024-12-lambowin/blob/874fafc7b27042c59bdd765073f5e412a3b79192/src/rebalance/LamboRebalanceOnUniwap.sol#L147)  

If no rebalancing is needed, it reverts with misleading error message "amountIn must be greater than zero" when users call `previewRebalance()`.
```
function _getTokenInOut() internal view returns (address tokenIn, address tokenOut, uint256 amountIn) {
        (uint256 wethBalance, uint256 vethBalance) = _getTokenBalances();
        uint256 targetBalance = (wethBalance + vethBalance) / 2;

        if (vethBalance > targetBalance) {
            amountIn = vethBalance - targetBalance;
            tokenIn = weth;
            tokenOut = veth;
        } else {
            amountIn = wethBalance - targetBalance;
            tokenIn = veth;
            tokenOut = weth;
        }

        require(amountIn > 0, "amountIn must be greater than zero");
    }
```

### Recommendations
Display a clearer error message such as "No rebalancing needed"