# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Don't use `_msgSender()` if not supporting EIP-2771 | 3 |
| [GAS-2](#GAS-2) | `a = a + b` is more gas effective than `a += b` for state variables (excluding arrays and mappings) | 4 |
| [GAS-3](#GAS-3) | Using bools for storage incurs overhead | 3 |
| [GAS-4](#GAS-4) | For Operations that will not overflow, you could use unchecked | 70 |
| [GAS-5](#GAS-5) | Use Custom Errors instead of Revert Strings to save Gas | 32 |
| [GAS-6](#GAS-6) | Avoid contract existence checks by using low level calls | 6 |
| [GAS-7](#GAS-7) | Functions guaranteed to revert when called by normal users can be marked `payable` | 10 |
| [GAS-8](#GAS-8) | Using `private` rather than `public` for constants, saves gas | 15 |
| [GAS-9](#GAS-9) | Use shift right/left instead of division/multiplication if possible | 1 |
| [GAS-10](#GAS-10) | Use != 0 instead of > 0 for unsigned integer comparison | 4 |
| [GAS-11](#GAS-11) | WETH address definition can be use directly | 2 |
### <a name="GAS-1"></a>[GAS-1] Don't use `_msgSender()` if not supporting EIP-2771
Use `msg.sender` if the code does not implement [EIP-2771 trusted forwarder](https://eips.ethereum.org/EIPS/eip-2771) support

*Instances (3)*:
```solidity
File: ./src/LamboToken.sol

99:         address owner = _msgSender();

122:         address owner = _msgSender();

144:         address spender = _msgSender();

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboToken.sol)

### <a name="GAS-2"></a>[GAS-2] `a = a + b` is more gas effective than `a += b` for state variables (excluding arrays and mappings)
This saves **16 gas per instance.**

*Instances (4)*:
```solidity
File: ./src/LamboToken.sol

180:             _totalSupply += value;

200:                 _balances[to] += value;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboToken.sol)

```solidity
File: ./src/VirtualToken.sol

95:         loanedAmountThisBlock += amount;

116:         _debt[user] += amount;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

### <a name="GAS-3"></a>[GAS-3] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (3)*:
```solidity
File: ./src/LamboFactory.sol

22:     mapping(address => bool) public whiteList;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboFactory.sol)

```solidity
File: ./src/VirtualToken.sol

21:     mapping(address => bool) public whiteList;

22:     mapping(address => bool) public validFactories;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

### <a name="GAS-4"></a>[GAS-4] For Operations that will not overflow, you could use unchecked

*Instances (70)*:
```solidity
File: ./src/LamboFactory.sol

4: import {Clones} from "@openzeppelin/contracts/proxy/Clones.sol";

5: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

6: import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

7: import {LamboToken} from "./LamboToken.sol";

8: import {VirtualToken} from "./VirtualToken.sol";

9: import {LaunchPadUtils} from "./Utils/LaunchPadUtils.sol";

10: import {IPool} from "./interfaces/Uniswap/IPool.sol";

11: import {IPoolFactory} from "./interfaces/Uniswap/IPoolFactory.sol";

12: import {UniswapV2Library} from "./libraries/UniswapV2Library.sol";

13: import {LamboVEthRouter} from "./LamboVEthRouter.sol";

15: import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";

16: import {ReentrancyGuard} from "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboFactory.sol)

```solidity
File: ./src/LamboToken.sol

6: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

7: import {IERC20Metadata} from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

8: import {Context} from "@openzeppelin/contracts/utils/Context.sol";

9: import {IERC20Errors} from "@openzeppelin/contracts/interfaces/draft-IERC6093.sol";

10: import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";

12: import {LaunchPadUtils} from "./Utils/LaunchPadUtils.sol";

180:             _totalSupply += value;

188:                 _balances[from] = fromBalance - value;

195:                 _totalSupply -= value;

200:                 _balances[to] += value;

286:                 _approve(owner, spender, currentAllowance - value, false);

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboToken.sol)

```solidity
File: ./src/LamboVEthRouter.sol

4: import "./libraries/UniswapV2Library.sol";

5: import {LaunchPadUtils} from "./Utils/LaunchPadUtils.sol";

7: import {IPool} from "./interfaces/Uniswap/IPool.sol";

8: import {VirtualToken} from "./VirtualToken.sol";

9: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

10: import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

11: import {LamboFactory} from "./LamboFactory.sol";

12: import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";

13: import {ReentrancyGuard} from "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

68:         amountIn = amountIn - (amountIn * feeRate) / feeDenominator;

82:         amount = amount - (amount * feeRate) / feeDenominator;

132:         uint256 fee = (amountXOut * feeRate) / feeDenominator;

133:         amountXOut = amountXOut - fee;

152:         uint256 fee = (amountXIn * feeRate) / feeDenominator;

153:         amountXIn = amountXIn - fee;

180:         if (msg.value > (amountXIn + fee + 1)) {

181:             (bool success, ) = payable(msg.sender).call{value: msg.value - amountXIn - fee - 1}("");

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

```solidity
File: ./src/Utils/LaunchPadUtils.sol

13:     uint256 public constant TOTAL_AMOUNT_OF_QUOTE_TOKEN = 10 ** 8 * 1e18;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/Utils/LaunchPadUtils.sol)

```solidity
File: ./src/VirtualToken.sol

4: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

5: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

6: import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

7: import {LaunchPadUtils} from "./Utils/LaunchPadUtils.sol";

8: import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";

93:         require(loanedAmountThisBlock + amount <= MAX_LOAN_PER_BLOCK, "Loan limit per block exceeded");

95:         loanedAmountThisBlock += amount;

116:         _debt[user] += amount;

121:         _debt[user] -= amount;

145:         if (from != address(0) && balanceOf(from) < value + _debt[from]) {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

4: import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";

5: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

7: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

8: import "@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol";

9: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

11: import {VirtualToken} from "../VirtualToken.sol";

12: import {IWETH} from "../interfaces/IWETH.sol";

13: import {IQuoter} from "../interfaces/Uniswap/IQuoter.sol";

14: import {IDexRouter} from "../interfaces/OKX/IDexRouter.sol";

15: import {IMorpho} from "@morpho/interfaces/IMorpho.sol";

16: import {IMorphoFlashLoanCallback} from "@morpho/interfaces/IMorphoCallbacks.sol";

27:     uint256 private constant _BUY_MASK = 1 << 255; // Mask for identifying if the swap is one-for-zero

28:     uint256 private constant _SELL_MASK = 0; // Mask for identifying if the swap is one-for-zero

67:         uint256 profit = balanceAfter - balanceBefore;

103:         uint256 newBalance = address(this).balance - initialBalance;

135:         uint256 targetBalance = (wethBalance + vethBalance) / 2;

138:             amountIn = vethBalance - targetBalance;

142:             amountIn = wethBalance - targetBalance;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="GAS-5"></a>[GAS-5] Use Custom Errors instead of Revert Strings to save Gas
Custom errors are available from solidity version 0.8.4. Custom errors save [**~50 gas**](https://gist.github.com/IllIllI000/ad1bd0d29a0101b25e57c293b4b0c746) each time they're hit by [avoiding having to allocate and store the revert string](https://blog.soliditylang.org/2021/04/21/custom-errors/#errors-in-depth). Not defining the strings also save deployment gas

Additionally, custom errors can be used inside and outside of contracts (including interfaces and libraries).

Source: <https://blog.soliditylang.org/2021/04/21/custom-errors/>:

> Starting from [Solidity v0.8.4](https://github.com/ethereum/solidity/releases/tag/v0.8.4), there is a convenient and gas-efficient way to explain to users why an operation failed through the use of custom errors. Until now, you could already use strings to give more information about failures (e.g., `revert("Insufficient funds.");`), but they are rather expensive, especially when it comes to deploy cost, and it is difficult to use dynamic information in them.

Consider replacing **all revert strings** with custom errors in the solution, and particularly those that have multiple occurrences:

*Instances (32)*:
```solidity
File: ./src/LamboFactory.sol

36:         require(_lamboTokenImplementation != address(0), "Invalid token implementation address");

41:         require(whiteList[virtualLiquidityToken], "virtualLiquidityToken is not in the whitelist");

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboFactory.sol)

```solidity
File: ./src/LamboToken.sol

35:         require(_totalSupply == 0, "LamboToken: Already initialized");

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboToken.sol)

```solidity
File: ./src/LamboVEthRouter.sol

29:         require(_vETH != address(0), "Invalid vETH address");

36:         require(newFeeRate <= feeDenominator, "Fee rate must be less than or equal to feeDenominator");

48:         require(VirtualToken(vETH).isValidFactory(lamboFactory), "only Validfactory");

134:         require(amountXOut >= minReturn, "Insufficient output amount. MinReturn Error.");

138:         require(success, "Transfer to User failed");

142:         require(success, "Transfer to owner() failed");

149:         require(msg.value >= amountXIn, "Insufficient msg.value");

155:         require(success, "Transfer to Owner failed");

168:         require(amountYOut >= minReturn, "Insufficient output amount");

182:             require(success, "ETH transfer failed");

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

```solidity
File: ./src/VirtualToken.sol

35:         require(whiteList[msg.sender], "Only WhiteList");

40:         require(validFactories[msg.sender], "Only valid factory can call this function");

49:         require(_underlyingToken != address(0), "Invalid underlying token address");

74:             require(msg.value == amount, "Invalid ETH amount");

93:         require(loanedAmountThisBlock + amount <= MAX_LOAN_PER_BLOCK, "Loan limit per block exceeded");

120:         require(_debt[user] >= amount, "Decrease amount exceeds current debt");

126:             require(msg.value >= amount, "Invalid ETH amount");

134:             require(address(this).balance >= amount, "Insufficient ETH balance");

136:             require(success, "Transfer failed");

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

41:         require(_multiSign != address(0), "Invalid _multiSign address");

42:         require(_vETH != address(0), "Invalid _vETH address");

43:         require(_uniswap != address(0), "Invalid _uniswap address");

68:         require(profit > 0, "No profit made");

72:         require(msg.sender == address(morphoVault), "Caller is not morphoVault");

74:         require(amountIn == assets, "Amount in does not match assets");

86:         require(IERC20(weth).approve(address(morphoVault), assets), "Approve failed");

93:         require(IERC20(weth).approve(address(OKXTokenApprove), amountIn), "Approve failed");

112:         require(IERC20(veth).approve(address(OKXTokenApprove), amountIn), "Approve failed");

147:         require(amountIn > 0, "amountIn must be greater than zero");

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="GAS-6"></a>[GAS-6] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including `EXTCODESIZE` (**100 gas**), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

*Instances (6)*:
```solidity
File: ./src/LamboFactory.sol

80:         IERC20(pool).safeTransfer(address(0), IERC20(pool).balanceOf(address(this)));

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboFactory.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

56:         uint256 balance = IERC20(token).balanceOf(address(this));

63:         uint256 balanceBefore = IERC20(weth).balanceOf(address(this));

66:         uint256 balanceAfter = IERC20(weth).balanceOf(address(this));

129:         wethBalance = IERC20(weth).balanceOf(uniswapPool);

130:         vethBalance = IERC20(veth).balanceOf(uniswapPool);

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="GAS-7"></a>[GAS-7] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (10)*:
```solidity
File: ./src/LamboFactory.sol

45:     function addVTokenWhiteList(address virtualLiquidityToken) public onlyOwner {

50:     function removeVTokenWhiteList(address virtualLiquidityToken) public onlyOwner {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboFactory.sol)

```solidity
File: ./src/LamboVEthRouter.sol

35:     function updateFeeRate(uint256 newFeeRate) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

```solidity
File: ./src/VirtualToken.sol

57:     function updateFactory(address _factory, bool isValid) external onlyOwner {

62:     function addToWhiteList(address user) external onlyOwner {

67:     function removeFromWhiteList(address user) external onlyOwner {

82:     function cashOut(uint256 amount) external onlyWhiteListed {

105:     function repayLoan(address to, uint256 amount) external onlyValidFactory {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

53:     function _authorizeUpgrade(address newImplementation) internal override onlyOwner {}

55:     function extractProfit(address to, address token) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="GAS-8"></a>[GAS-8] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (15)*:
```solidity
File: ./src/LamboVEthRouter.sol

18:     address public constant NATIVE_TOKEN = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

19:     uint256 public constant feeDenominator = 10000;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

```solidity
File: ./src/Utils/LaunchPadUtils.sol

8:     uint256 public constant MAX_AMOUNT = 115792089237316195423570985008687907853269984665640564039457584007913129639935;

10:     address public constant NATIVE_TOKEN = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

13:     uint256 public constant TOTAL_AMOUNT_OF_QUOTE_TOKEN = 10 ** 8 * 1e18;

16:     address public constant WETH = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;

18:     address public constant CURVE_STABLE_NG_FACTORY = 0x6A8cbed756804B16E05E741eDaBd5cB544AE21bf;

21:     address public constant UNISWAP_POOL_FACTORY_ = 0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f;

24:     address public constant UNISWAP_ROUTER_ADDRESS = 0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/Utils/LaunchPadUtils.sol)

```solidity
File: ./src/VirtualToken.sol

18:     uint256 public constant MAX_LOAN_PER_BLOCK = 300 ether;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

30:     address public constant weth = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;

31:     address public constant morphoVault = 0xBBBBBbbBBb9cC5e90e3b3Af64bdAF62C37EEFFCb;

32:     address public constant quoter = 0x5e55C9e631FAE526cd4B0526C4818D6e0a9eF0e3;

33:     address public constant OKXRouter = 0x7D0CcAa3Fac1e5A943c5168b6CEd828691b46B36;

34:     address public constant OKXTokenApprove = 0x40aA958dd87FC8305b97f2BA922CDdCa374bcD7f;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="GAS-9"></a>[GAS-9] Use shift right/left instead of division/multiplication if possible
While the `DIV` / `MUL` opcode uses 5 gas, the `SHR` / `SHL` opcode only uses 3 gas. Furthermore, beware that Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting. Eventually, overflow checks are never performed for shift operations as they are done for arithmetic operations. Instead, the result is always truncated, so the calculation can be unchecked in Solidity version `0.8+`
- Use `>> 1` instead of `/ 2`
- Use `>> 2` instead of `/ 4`
- Use `<< 3` instead of `* 8`
- ...
- Use `>> 5` instead of `/ 2^5 == / 32`
- Use `<< 6` instead of `* 2^6 == * 64`

TL;DR:
- Shifting left by N is like multiplying by 2^N (Each bits to the left is an increased power of 2)
- Shifting right by N is like dividing by 2^N (Each bits to the right is a decreased power of 2)

*Saves around 2 gas + 20 for unchecked per instance*

*Instances (1)*:
```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

135:         uint256 targetBalance = (wethBalance + vethBalance) / 2;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="GAS-10"></a>[GAS-10] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (4)*:
```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

57:         if (balance > 0) {

68:         require(profit > 0, "No profit made");

104:         if (newBalance > 0) {

147:         require(amountIn > 0, "amountIn must be greater than zero");

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="GAS-11"></a>[GAS-11] WETH address definition can be use directly
WETH is a wrap Ether contract with a specific address in the Ethereum network, giving the option to define it may cause false recognition, it is healthier to define it directly.

    Advantages of defining a specific contract directly:
    
    It saves gas,
    Prevents incorrect argument definition,
    Prevents execution on a different chain and re-signature issues,
    WETH Address : 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2

*Instances (2)*:
```solidity
File: ./src/Utils/LaunchPadUtils.sol

16:     address public constant WETH = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/Utils/LaunchPadUtils.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

30:     address public constant weth = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Constants should be in CONSTANT_CASE | 6 |
| [NC-2](#NC-2) | `constant`s should be defined rather than using magic numbers | 3 |
| [NC-3](#NC-3) | Control structures do not follow the Solidity Style Guide | 2 |
| [NC-4](#NC-4) | Consider disabling `renounceOwnership()` | 4 |
| [NC-5](#NC-5) | Draft Dependencies | 1 |
| [NC-6](#NC-6) | Functions should not be longer than 50 lines | 46 |
| [NC-7](#NC-7) | Use a `modifier` instead of a `require/if` statement for a special `msg.sender` actor | 3 |
| [NC-8](#NC-8) | Consider using named mappings | 4 |
| [NC-9](#NC-9) | `address`s shouldn't be hard-coded | 11 |
| [NC-10](#NC-10) | Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`) | 1 |
| [NC-11](#NC-11) | Avoid the use of sensitive terms | 25 |
| [NC-12](#NC-12) | Use Underscores for Number Literals (add an underscore every 3 digits) | 2 |
### <a name="NC-1"></a>[NC-1] Constants should be in CONSTANT_CASE
For `constant` variable names, each word should use all capital letters, with underscores separating each word (CONSTANT_CASE)

*Instances (6)*:
```solidity
File: ./src/LamboVEthRouter.sol

19:     uint256 public constant feeDenominator = 10000;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

30:     address public constant weth = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;

31:     address public constant morphoVault = 0xBBBBBbbBBb9cC5e90e3b3Af64bdAF62C37EEFFCb;

32:     address public constant quoter = 0x5e55C9e631FAE526cd4B0526C4818D6e0a9eF0e3;

33:     address public constant OKXRouter = 0x7D0CcAa3Fac1e5A943c5168b6CEd828691b46B36;

34:     address public constant OKXTokenApprove = 0x40aA958dd87FC8305b97f2BA922CDdCa374bcD7f;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="NC-2"></a>[NC-2] `constant`s should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

*Instances (3)*:
```solidity
File: ./src/LamboToken.sol

73:         return 18;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboToken.sol)

```solidity
File: ./src/LamboVEthRouter.sol

31:         feeRate = 100;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

135:         uint256 targetBalance = (wethBalance + vethBalance) / 2;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="NC-3"></a>[NC-3] Control structures do not follow the Solidity Style Guide
See the [control structures](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures) section of the Solidity Style Guide

*Instances (2)*:
```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

27:     uint256 private constant _BUY_MASK = 1 << 255; // Mask for identifying if the swap is one-for-zero

28:     uint256 private constant _SELL_MASK = 0; // Mask for identifying if the swap is one-for-zero

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="NC-4"></a>[NC-4] Consider disabling `renounceOwnership()`
If the plan for your project does not include eventually giving up all ownership control, consider overwriting OpenZeppelin's `Ownable`'s `renounceOwnership()` function in order to disable it.

*Instances (4)*:
```solidity
File: ./src/LamboFactory.sol

18: contract LamboFactory is Ownable, ReentrancyGuard {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboFactory.sol)

```solidity
File: ./src/LamboToken.sol

14: contract LamboToken is Context, IERC20, IERC20Metadata, IERC20Errors, Ownable {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboToken.sol)

```solidity
File: ./src/LamboVEthRouter.sol

15: contract LamboVEthRouter is Ownable, ReentrancyGuard {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

```solidity
File: ./src/VirtualToken.sol

10: contract VirtualToken is ERC20, Ownable {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

### <a name="NC-5"></a>[NC-5] Draft Dependencies
Draft contracts have not received adequate security auditing or are liable to change with future developments.

*Instances (1)*:
```solidity
File: ./src/LamboToken.sol

9: import {IERC20Errors} from "@openzeppelin/contracts/interfaces/draft-IERC6093.sol";

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboToken.sol)

### <a name="NC-6"></a>[NC-6] Functions should not be longer than 50 lines
Overly complex code can make understanding functionality more difficult, try to further modularize your code to ensure readability 

*Instances (46)*:
```solidity
File: ./src/LamboFactory.sol

45:     function addVTokenWhiteList(address virtualLiquidityToken) public onlyOwner {

50:     function removeVTokenWhiteList(address virtualLiquidityToken) public onlyOwner {

55:     function _deployLamboToken(string memory name, string memory tickname) internal returns (address quoteToken) {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboFactory.sol)

```solidity
File: ./src/LamboToken.sol

34:     function initialize(string memory _name, string memory _symbol) public {

47:     function name() public view virtual returns (string memory) {

55:     function symbol() public view virtual returns (string memory) {

72:     function decimals() public view virtual returns (uint8) {

79:     function totalSupply() public view virtual returns (uint256) {

86:     function balanceOf(address account) public view virtual returns (uint256) {

98:     function transfer(address to, uint256 value) public virtual returns (bool) {

107:     function allowance(address owner, address spender) public view virtual returns (uint256) {

121:     function approve(address spender, uint256 value) public virtual returns (bool) {

143:     function transferFrom(address from, address to, uint256 value) public virtual returns (bool) {

160:     function _transfer(address from, address to, uint256 value) internal {

177:     function _update(address from, address to, uint256 value) internal virtual {

215:     function _mint(address account, uint256 value) internal {

237:     function _approve(address owner, address spender, uint256 value) internal {

258:     function _approve(address owner, address spender, uint256 value, bool emitEvent) internal virtual {

279:     function _spendAllowance(address owner, address spender, uint256 value) internal virtual {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboToken.sol)

```solidity
File: ./src/LamboVEthRouter.sol

35:     function updateFeeRate(uint256 newFeeRate) external onlyOwner {

59:     function getBuyQuote(address targetToken, uint256 amountIn) public view returns (uint256 amount) {

72:     function getSellQuote(address targetToken, uint256 amountIn) public view returns (uint256 amount) {

148:     function _buyQuote(address quoteToken, uint256 amountXIn, uint256 minReturn) internal returns (uint256 amountYOut) {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

```solidity
File: ./src/VirtualToken.sol

53:     function isValidFactory(address _factory) external view returns (bool) {

57:     function updateFactory(address _factory, bool isValid) external onlyOwner {

62:     function addToWhiteList(address user) external onlyOwner {

67:     function removeFromWhiteList(address user) external onlyOwner {

72:     function cashIn(uint256 amount) external payable onlyWhiteListed {

82:     function cashOut(uint256 amount) external onlyWhiteListed {

88:     function takeLoan(address to, uint256 amount) external payable onlyValidFactory {

105:     function repayLoan(address to, uint256 amount) external onlyValidFactory {

111:     function getLoanDebt(address user) external view returns (uint256) {

115:     function _increaseDebt(address user, uint256 amount) internal {

119:     function _decreaseDebt(address user, uint256 amount) internal {

124:     function _transferAssetFromUser(uint256 amount) internal {

132:     function _transferAssetToUser(uint256 amount) internal {

143:     function _update(address from, address to, uint256 value) internal override {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

40:     function initialize(address _multiSign, address _vETH, address _uniswap, uint24 _fee) public initializer {

53:     function _authorizeUpgrade(address newImplementation) internal override onlyOwner {}

55:     function extractProfit(address to, address token) external onlyOwner {

62:     function rebalance(uint256 directionMask, uint256 amountIn, uint256 amountOut) external nonReentrant {

71:     function onMorphoFlashLoan(uint256 assets, bytes calldata data) external {

89:     function _executeBuy(uint256 amountIn, uint256[] memory pools) internal {

109:     function _executeSell(uint256 amountIn, uint256[] memory pools) internal {

128:     function _getTokenBalances() internal view returns (uint256 wethBalance, uint256 vethBalance) {

133:     function _getTokenInOut() internal view returns (address tokenIn, address tokenOut, uint256 amountIn) {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="NC-7"></a>[NC-7] Use a `modifier` instead of a `require/if` statement for a special `msg.sender` actor
If a function is supposed to be access-controlled, a `modifier` should be used instead of a `require/if` statement for more readability.

*Instances (3)*:
```solidity
File: ./src/VirtualToken.sol

35:         require(whiteList[msg.sender], "Only WhiteList");

40:         require(validFactories[msg.sender], "Only valid factory can call this function");

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

72:         require(msg.sender == address(morphoVault), "Caller is not morphoVault");

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="NC-8"></a>[NC-8] Consider using named mappings
Consider moving to solidity version 0.8.18 or later, and using [named mappings](https://ethereum.stackexchange.com/questions/51629/how-to-name-the-arguments-in-mapping/145555#145555) to make it easier to understand the purpose of each mapping

*Instances (4)*:
```solidity
File: ./src/LamboFactory.sol

22:     mapping(address => bool) public whiteList;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboFactory.sol)

```solidity
File: ./src/VirtualToken.sol

20:     mapping(address => uint256) public _debt;

21:     mapping(address => bool) public whiteList;

22:     mapping(address => bool) public validFactories;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

### <a name="NC-9"></a>[NC-9] `address`s shouldn't be hard-coded
It is often better to declare `address`es as `immutable`, and assign them via constructor arguments. This allows the code to remain the same across deployments on different networks, and avoids recompilation when addresses need to change.

*Instances (11)*:
```solidity
File: ./src/LamboVEthRouter.sol

18:     address public constant NATIVE_TOKEN = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

```solidity
File: ./src/Utils/LaunchPadUtils.sol

10:     address public constant NATIVE_TOKEN = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

16:     address public constant WETH = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;

18:     address public constant CURVE_STABLE_NG_FACTORY = 0x6A8cbed756804B16E05E741eDaBd5cB544AE21bf;

21:     address public constant UNISWAP_POOL_FACTORY_ = 0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f;

24:     address public constant UNISWAP_ROUTER_ADDRESS = 0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/Utils/LaunchPadUtils.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

30:     address public constant weth = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;

31:     address public constant morphoVault = 0xBBBBBbbBBb9cC5e90e3b3Af64bdAF62C37EEFFCb;

32:     address public constant quoter = 0x5e55C9e631FAE526cd4B0526C4818D6e0a9eF0e3;

33:     address public constant OKXRouter = 0x7D0CcAa3Fac1e5A943c5168b6CEd828691b46B36;

34:     address public constant OKXTokenApprove = 0x40aA958dd87FC8305b97f2BA922CDdCa374bcD7f;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="NC-10"></a>[NC-10] Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`)
While this won't save gas in the recent solidity versions, this is shorter and more readable (this is especially true in calculations).

*Instances (1)*:
```solidity
File: ./src/Utils/LaunchPadUtils.sol

13:     uint256 public constant TOTAL_AMOUNT_OF_QUOTE_TOKEN = 10 ** 8 * 1e18;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/Utils/LaunchPadUtils.sol)

### <a name="NC-11"></a>[NC-11] Avoid the use of sensitive terms
Use [alternative variants](https://www.zdnet.com/article/mysql-drops-master-slave-and-blacklist-whitelist-terminology/), e.g. allowlist/denylist instead of whitelist/blacklist

*Instances (25)*:
```solidity
File: ./src/LamboFactory.sol

22:     mapping(address => bool) public whiteList;

32:     event VTokenWhiteListAdded(address virtualLiquidityToken);

33:     event VTokenWhiteListRemoved(address virtualLiquidityToken);

40:     modifier onlyWhiteListed(address virtualLiquidityToken) {

41:         require(whiteList[virtualLiquidityToken], "virtualLiquidityToken is not in the whitelist");

45:     function addVTokenWhiteList(address virtualLiquidityToken) public onlyOwner {

46:         whiteList[virtualLiquidityToken] = true;

47:         emit VTokenWhiteListAdded(virtualLiquidityToken);

50:     function removeVTokenWhiteList(address virtualLiquidityToken) public onlyOwner {

51:         whiteList[virtualLiquidityToken] = false;

52:         emit VTokenWhiteListRemoved(virtualLiquidityToken);

70:     ) public onlyWhiteListed(virtualLiquidityToken) nonReentrant returns (address quoteToken, address pool) {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboFactory.sol)

```solidity
File: ./src/VirtualToken.sol

21:     mapping(address => bool) public whiteList;

29:     event WhiteListAdded(address user);

30:     event WhiteListRemoved(address user);

34:     modifier onlyWhiteListed() {

35:         require(whiteList[msg.sender], "Only WhiteList");

62:     function addToWhiteList(address user) external onlyOwner {

63:         whiteList[user] = true;

64:         emit WhiteListAdded(user);

67:     function removeFromWhiteList(address user) external onlyOwner {

68:         whiteList[user] = false;

69:         emit WhiteListRemoved(user);

72:     function cashIn(uint256 amount) external payable onlyWhiteListed {

82:     function cashOut(uint256 amount) external onlyWhiteListed {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

### <a name="NC-12"></a>[NC-12] Use Underscores for Number Literals (add an underscore every 3 digits)

*Instances (2)*:
```solidity
File: ./src/LamboVEthRouter.sol

19:     uint256 public constant feeDenominator = 10000;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

```solidity
File: ./src/Utils/LaunchPadUtils.sol

8:     uint256 public constant MAX_AMOUNT = 115792089237316195423570985008687907853269984665640564039457584007913129639935;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/Utils/LaunchPadUtils.sol)


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | `approve()`/`safeApprove()` may revert if the current approval is not zero | 3 |
| [L-2](#L-2) | Use a 2-step ownership transfer pattern | 5 |
| [L-3](#L-3) | Division by zero not prevented | 4 |
| [L-4](#L-4) | External call recipient may consume all transaction gas | 5 |
| [L-5](#L-5) | Initializers could be front-run | 5 |
| [L-6](#L-6) | Use `Ownable2Step.transferOwnership` instead of `Ownable.transferOwnership` | 7 |
| [L-7](#L-7) | Unsafe ERC20 operation(s) | 3 |
| [L-8](#L-8) | Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions | 7 |
| [L-9](#L-9) | Upgradeable contract not initialized | 13 |
### <a name="L-1"></a>[L-1] `approve()`/`safeApprove()` may revert if the current approval is not zero
- Some tokens (like the *very popular* USDT) do not work when changing the allowance from an existing non-zero allowance value (it will revert if the current approval is not zero to protect against front-running changes of approvals). These tokens must first be approved for zero and then the actual allowance can be approved.
- Furthermore, OZ's implementation of safeApprove would throw an error if an approve is attempted from a non-zero value (`"SafeERC20: approve from non-zero to non-zero allowance"`)

Set the allowance to zero immediately before each of the existing allowance calls

*Instances (3)*:
```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

86:         require(IERC20(weth).approve(address(morphoVault), assets), "Approve failed");

93:         require(IERC20(weth).approve(address(OKXTokenApprove), amountIn), "Approve failed");

112:         require(IERC20(veth).approve(address(OKXTokenApprove), amountIn), "Approve failed");

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="L-2"></a>[L-2] Use a 2-step ownership transfer pattern
Recommend considering implementing a two step process where the owner or admin nominates an account and the nominated account needs to call an `acceptOwnership()` function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account. Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

*Instances (5)*:
```solidity
File: ./src/LamboFactory.sol

18: contract LamboFactory is Ownable, ReentrancyGuard {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboFactory.sol)

```solidity
File: ./src/LamboToken.sol

14: contract LamboToken is Context, IERC20, IERC20Metadata, IERC20Errors, Ownable {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboToken.sol)

```solidity
File: ./src/LamboVEthRouter.sol

15: contract LamboVEthRouter is Ownable, ReentrancyGuard {

28:     constructor(address _vETH, address _multiSign) public Ownable(_multiSign) {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

```solidity
File: ./src/VirtualToken.sol

10: contract VirtualToken is ERC20, Ownable {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

### <a name="L-3"></a>[L-3] Division by zero not prevented
The divisions below take an input parameter which does not have any zero-value checks, which may lead to the functions reverting when zero is passed.

*Instances (4)*:
```solidity
File: ./src/LamboVEthRouter.sol

68:         amountIn = amountIn - (amountIn * feeRate) / feeDenominator;

82:         amount = amount - (amount * feeRate) / feeDenominator;

132:         uint256 fee = (amountXOut * feeRate) / feeDenominator;

152:         uint256 fee = (amountXIn * feeRate) / feeDenominator;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

### <a name="L-4"></a>[L-4] External call recipient may consume all transaction gas
There is no limit specified on the amount of gas used, so the recipient can use up all of the transaction's gas, causing it to revert. Use `addr.call{gas: <amount>}("")` or [this](https://github.com/nomad-xyz/ExcessivelySafeCall) library instead.

*Instances (5)*:
```solidity
File: ./src/LamboVEthRouter.sol

137:         (bool success, ) = msg.sender.call{value: amountXOut}("");

141:         (success, ) = payable(owner()).call{value: fee}("");

154:         (bool success, ) = payable(owner()).call{value: fee}("");

181:             (bool success, ) = payable(msg.sender).call{value: msg.value - amountXIn - fee - 1}("");

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

```solidity
File: ./src/VirtualToken.sol

135:             (bool success, ) = msg.sender.call{value: amount}("");

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

### <a name="L-5"></a>[L-5] Initializers could be front-run
Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

*Instances (5)*:
```solidity
File: ./src/LamboFactory.sol

60:         LamboToken(quoteToken).initialize(name, tickname);

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboFactory.sol)

```solidity
File: ./src/LamboToken.sol

34:     function initialize(string memory _name, string memory _symbol) public {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboToken.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

40:     function initialize(address _multiSign, address _vETH, address _uniswap, uint24 _fee) public initializer {

45:         __Ownable_init(_multiSign);

46:         __ReentrancyGuard_init();

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="L-6"></a>[L-6] Use `Ownable2Step.transferOwnership` instead of `Ownable.transferOwnership`
Use [Ownable2Step.transferOwnership](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) which is safer. Use it as it is more secure due to 2-stage ownership transfer.

**Recommended Mitigation Steps**

Use <a href="https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol">Ownable2Step.sol</a>
  
  ```solidity
      function acceptOwnership() external {
          address sender = _msgSender();
          require(pendingOwner() == sender, "Ownable2Step: caller is not the new owner");
          _transferOwnership(sender);
      }
```

*Instances (7)*:
```solidity
File: ./src/LamboFactory.sol

15: import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboFactory.sol)

```solidity
File: ./src/LamboToken.sol

10: import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";

31:         _transferOwnership(address(0));

41:         _transferOwnership(address(0));

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboToken.sol)

```solidity
File: ./src/LamboVEthRouter.sol

12: import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

```solidity
File: ./src/VirtualToken.sol

8: import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

5: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="L-7"></a>[L-7] Unsafe ERC20 operation(s)

*Instances (3)*:
```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

86:         require(IERC20(weth).approve(address(morphoVault), assets), "Approve failed");

93:         require(IERC20(weth).approve(address(OKXTokenApprove), amountIn), "Approve failed");

112:         require(IERC20(veth).approve(address(OKXTokenApprove), amountIn), "Approve failed");

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="L-8"></a>[L-8] Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions
See [this](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

*Instances (7)*:
```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

4: import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";

5: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

7: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

8: import "@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol";

20:     UUPSUpgradeable,

21:     OwnableUpgradeable,

22:     ReentrancyGuardUpgradeable,

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

### <a name="L-9"></a>[L-9] Upgradeable contract not initialized
Upgradeable contracts are initialized via an initializer function rather than by a constructor. Leaving such a contract uninitialized may lead to it being taken over by a malicious user

*Instances (13)*:
```solidity
File: ./src/LamboFactory.sol

60:         LamboToken(quoteToken).initialize(name, tickname);

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboFactory.sol)

```solidity
File: ./src/LamboToken.sol

34:     function initialize(string memory _name, string memory _symbol) public {

35:         require(_totalSupply == 0, "LamboToken: Already initialized");

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboToken.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

4: import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";

5: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

7: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

8: import "@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol";

20:     UUPSUpgradeable,

21:     OwnableUpgradeable,

22:     ReentrancyGuardUpgradeable,

40:     function initialize(address _multiSign, address _vETH, address _uniswap, uint24 _fee) public initializer {

45:         __Ownable_init(_multiSign);

46:         __ReentrancyGuard_init();

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)


## Medium Issues


| |Issue|Instances|
|-|:-|:-:|
| [M-1](#M-1) | `block.number` means different things on different L2s | 2 |
| [M-2](#M-2) | Centralization Risk for trusted owners | 16 |
### <a name="M-1"></a>[M-1] `block.number` means different things on different L2s
On Optimism, `block.number` is the L2 block number, but on Arbitrum, it's the L1 block number, and `ArbSys(address(100)).arbBlockNumber()` must be used. Furthermore, L2 block numbers often occur much more frequently than L1 block numbers (any may even occur on a per-transaction basis), so using block numbers for timing results in inconsistencies, especially when voting is involved across multiple chains. As of version 4.9, OpenZeppelin has [modified](https://blog.openzeppelin.com/introducing-openzeppelin-contracts-v4.9#governor) their governor code to use a clock rather than block numbers, to avoid these sorts of issues, but this still requires that the project [implement](https://docs.openzeppelin.com/contracts/4.x/governance#token_2) a [clock](https://eips.ethereum.org/EIPS/eip-6372) for each L2.

*Instances (2)*:
```solidity
File: ./src/VirtualToken.sol

89:         if (block.number > lastLoanBlock) {

90:             lastLoanBlock = block.number;

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

### <a name="M-2"></a>[M-2] Centralization Risk for trusted owners

#### Impact:
Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

*Instances (16)*:
```solidity
File: ./src/LamboFactory.sol

18: contract LamboFactory is Ownable, ReentrancyGuard {

35:     constructor(address _lamboTokenImplementation) Ownable(msg.sender) {

45:     function addVTokenWhiteList(address virtualLiquidityToken) public onlyOwner {

50:     function removeVTokenWhiteList(address virtualLiquidityToken) public onlyOwner {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboFactory.sol)

```solidity
File: ./src/LamboToken.sol

14: contract LamboToken is Context, IERC20, IERC20Metadata, IERC20Errors, Ownable {

30:     constructor() Ownable(msg.sender) {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboToken.sol)

```solidity
File: ./src/LamboVEthRouter.sol

15: contract LamboVEthRouter is Ownable, ReentrancyGuard {

28:     constructor(address _vETH, address _multiSign) public Ownable(_multiSign) {

35:     function updateFeeRate(uint256 newFeeRate) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/LamboVEthRouter.sol)

```solidity
File: ./src/VirtualToken.sol

10: contract VirtualToken is ERC20, Ownable {

48:     ) ERC20(name, symbol) Ownable(msg.sender) {

57:     function updateFactory(address _factory, bool isValid) external onlyOwner {

62:     function addToWhiteList(address user) external onlyOwner {

67:     function removeFromWhiteList(address user) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/VirtualToken.sol)

```solidity
File: ./src/rebalance/LamboRebalanceOnUniwap.sol

53:     function _authorizeUpgrade(address newImplementation) internal override onlyOwner {}

55:     function extractProfit(address to, address token) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2024-12-lambowin/blob/main/./src/rebalance/LamboRebalanceOnUniwap.sol)

