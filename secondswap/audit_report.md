# SecondSwap Audit
- Date
    - 10 Dec 2024 - 20 Dec 2024  
- Scope
    - ./contracts/SecondSwap_Marketplace.sol  
    - ./contracts/SecondSwap_MarketplaceSetting.sol  
    - ./contracts/SecondSwap_StepVesting.sol  
    - ./contracts/SecondSwap_VestingDeployer.sol  
    - ./contracts/SecondSwap_VestingManager.sol  
    - ./contracts/SecondSwap_Whitelist.sol
    - ./contracts/SecondSwap_WhitelistDeployer.sol
- Findings
    - Medium: 1  
    [M-01: Calling `SecondSwap_Marketplace::setMarketplaceSettingAddress` may unintentionally unfreeze the marketplace](#med-01)  
    - Low/QA: 8  
    [L-01: Floating pragma used in several contracts](#low-01)  
    [L-02: `SecondSwap_Marketplace::addCoin()` missing emit event](#low-02)  
    [L-03: Remove unused comment codes](#low-03)  
    [L-04: `SecondSwap_MarketplaceSetting::setBuyerFee()` and `SecondSwap_MarketplaceSetting::setSellerFee()` missing zero checks](#low-04)  
    [L-05: Hardcoded variables in `SecondSwap_MarketplaceSetting` constructor](#low-05)  
    [L-06: Redundant casting to uint256() in `SecondSwap_Marketplace::_getFees()`](#low-06)  
    [L-07: Missing remove whitelist function in `SecondSwap_Whitelist`](#low-07)  
    [L-08: Missing documentation and typo in comments](#low-08)  
    - Gas: 1  
    [G-01: Reduce repeated external calls in `SecondSwap_Marketplace::unlistVesting()`](#gas-01)

- Tools  
    - [Foundry](https://github.com/foundry-rs/foundry)

# Findings

## <a id="med-01"></a>M-01: Calling `SecondSwap_Marketplace::setMarketplaceSettingAddress` may unintentionally unfreeze the marketplace

### Summary
If an admin calls `SecondSwap_Marketplace::setMarketplaceSettingAddress` to upgrade the marketplace settings when the marketplace is freezed, it will cause `SecondSwap_MarketplaceSettings::isMarketplaceFreeze` to be set to false. This may cause an unintentional unpause of the protocol during a case of emergency and users will be able to continue calling functions like spotPurchase(), unlistVesting(), listVesting().

### Vulnerability Details
When a new SecondSwap_MarketplaceSetting contract is deployed and `SecondSwap_Marketplace::setMarketplaceSettingAddress` is called to update to the new address, it will point to the new `SecondSwap_MarketplaceSettings::isMarketplaceFreeze` which is set to false by default during deployment. 

Affected [code](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_Marketplace.sol#L560-L564)
```
function setMarketplaceSettingAddress(address _marketplaceSetting) external {
    require(msg.sender == IMarketplaceSetting(marketplaceSetting).s2Admin(), "SS_Marketplace: Unauthorized user");
    require(_marketplaceSetting != address(0), "SS_Marketplace: Address cannot be null");
    marketplaceSetting = _marketplaceSetting;
}
```
Affected [code](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_MarketplaceSetting.sol#L147)
```
constructor(
        address _feeCollector,
        address _s2Admin,
        address _whitelistDeployer,
        address _vestingManager,
        address _usdt
    ) {
        //...
        isMarketplaceFreeze = false;
        // ...  
    }
```

### Impact/Proof of Concept
```
function testMarketplaceSettingsUpgradeCanUnfreeze() public {
        vm.startPrank(s2Admin);
        // Freeze marketplace and set status to True
        marketplaceSetting.setMarketplaceStatus(true);
        assertEq(IMarketplaceSetting(marketplace.marketplaceSetting()).isMarketplaceFreeze(), true, "Marketplace is not freezed");

        // Deploy new marketplaceSetting and call setMarketplaceSettingAddress() to upgrade to new contract
        marketplaceSetting = new SecondSwap_MarketplaceSetting(
            feeCollector,
            s2Admin,
            address(whitelistDeployer),
            address(manager),
            address(usdt)
        );
        marketplace.setMarketplaceSettingAddress(address(marketplaceSetting));

        // Check marketplace status and see if it has change to false
        assertEq(IMarketplaceSetting(marketplace.marketplaceSetting()).isMarketplaceFreeze(), false, "Marketplace is still freezed");
        vm.stopPrank();
    }
```
Results
```
[PASS] testMarketplaceSettingsUpgradeCanUnfreeze() (gas: 974169)
Logs:
  tokens deployed
  vesting manager & deployer deployed
  vesting deployed
  marketplace & settings deployed

Traces:
  [974169] SecondSwapTest::testMarketplaceSettingsUpgradeCanUnfreeze()
    ├─ [0] VM::startPrank(s2Admin: [0x8D7DfAe223fBBBfc0F2A185d54406bF87E49ba7B])
    │   └─ ← [Return] 
    ├─ [8609] SecondSwap_MarketplaceSetting::setMarketplaceStatus(true)
    │   ├─ emit MarketplaceStatusUpdated(status: true)
    │   └─ ← [Stop] 
    ├─ [2397] SecondSwap_Marketplace::marketplaceSetting() [staticcall]
    │   └─ ← [Return] SecondSwap_MarketplaceSetting: [0x5991A2dF15A8F6A256D3Ec51E99254Cd3fb576A9]
    ├─ [438] SecondSwap_MarketplaceSetting::isMarketplaceFreeze() [staticcall]
    │   └─ ← [Return] true
    ├─ [0] VM::assertEq(true, true, "Marketplace is not freezed") [staticcall]
    │   └─ ← [Return] 
    ├─ [892614] → new SecondSwap_MarketplaceSetting@0xb8c748B0c9892CD7db9B1E4A4B98026fad4a8015
    │   └─ ← [Return] 3350 bytes of code
    ├─ [4593] SecondSwap_Marketplace::setMarketplaceSettingAddress(SecondSwap_MarketplaceSetting: [0xb8c748B0c9892CD7db9B1E4A4B98026fad4a8015])
    │   ├─ [626] SecondSwap_MarketplaceSetting::s2Admin() [staticcall]
    │   │   └─ ← [Return] s2Admin: [0x8D7DfAe223fBBBfc0F2A185d54406bF87E49ba7B]
    │   └─ ← [Return] 
    ├─ [397] SecondSwap_Marketplace::marketplaceSetting() [staticcall]
    │   └─ ← [Return] SecondSwap_MarketplaceSetting: [0xb8c748B0c9892CD7db9B1E4A4B98026fad4a8015]
    ├─ [438] SecondSwap_MarketplaceSetting::isMarketplaceFreeze() [staticcall]
    │   └─ ← [Return] false
    ├─ [0] VM::assertEq(false, false, "Marketplace is still freezed") [staticcall]
    │   └─ ← [Return] 
    ├─ [0] VM::stopPrank()
    │   └─ ← [Return] 
    └─ ← [Return] 

Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 1.68ms (124.98µs CPU time)
```

### Recommendations
Add an input parameter in `SecondSwap_MarketplaceSettings` constructor to allow admin to set `isMarketplaceFreeze` during deployment
```diff
constructor(
        address _feeCollector,
        address _s2Admin,
        address _whitelistDeployer,
        address _vestingManager,
        address _usdt
+        bool _isMarketplaceFreeze
    ) {
        // ...
+        isMarketplaceFreeze = _isMarketplaceFreeze;
        // ...
    }
```


## <a id="low-01"></a>L-01: Floating pragma used in several contracts

### Summary
Currently, the protocol uses a floating pragma, allowing the contract to be compiled with any 0.8.x Solidity version higher than 0.8.24. The security best practice is to set the pragma to a specific version, so that the contract is not accidentally compiled to a version which breaks the contract's operability.

### Vulnerability Details
|Contracts|
|:--:|
|[SecondSwap_Marketplace.sol](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_Marketplace.sol#L2)|
|[SecondSwap_MarketplaceSetting.sol](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_MarketplaceSetting.sol#L2)|
|[SecondSwap_StepVesting.so](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_StepVesting.sol#L2)|
|[SecondSwap_VestingDeployer.sol](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_VestingDeployer.sol#L2)|
|[SecondSwap_VestingManager.sol](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_VestingManager.sol#L2)|
|[SecondSwap_Whitelist.sol](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_Whitelist.sol#L2)|
|[SecondSwap_WhitelistDeployer.sol](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_WhitelistDeployer.sol#L2)|

### Recommendations
Set the pragma to a specific version.

## <a id="low-02"></a>L-02: `SecondSwap_Marketplace::addCoin()` missing emit event

### Summary
`addCoin()` is missing CoinAdded event emits, which prevents the intended data from being observed easily by off-chain interfaces.

### Vulnerability Details
Affected [code](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_Marketplace.sol#L205-L218)
```
function addCoin(address _token) external {
        require(msg.sender == IMarketplaceSetting(marketplaceSetting).s2Admin(), "SS_Marketplace: Unauthorized user");
        require(!isTokenSupport[_token], "SS_Marketplace: Token is currently supported");
        // try IERC20Extended(_token).decimals() returns (uint8 decimals) {   
        //     require(decimals <= 18, "SS_Marketplace: Token decimals too high");
        //     require(decimals > 0, "SS_Marketplace: Token decimals must be greater than 0");

        //     isTokenSupport[_token] = true;
        //     emit CoinAdded(_token); // Emit event when coin is added
        // } catch {
        //     revert("SS_Marketplace: Token must implement decimals function");
        // }                            
        isTokenSupport[_token] = true; 
    }
```

### Recommendations
Add emit event CoinAdded(address indexed token)

## <a id="low-03"></a>L-03: Remove unused comment codes

### Summary
Various functions has unused commented codes. These should be removed to provide better code quality.

### Vulnerability Details
1. https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_Marketplace.sol#L208-L216
2. https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_VestingDeployer.sol#L112

### Recommendations
Remove unused commented codes

## <a id="low-04"></a>L-04: `SecondSwap_MarketplaceSetting::setBuyerFee()` and `SecondSwap_MarketplaceSetting::setSellerFee()` missing zero checks

### Summary
`SecondSwap_MarketplaceSetting::setBuyerFee()` and `SecondSwap_MarketplaceSetting::setSellerFee()` do not include checks to ensure the fee amount are not zero. Additionally,  `SecondSwap_VestingManager::vestingSettings[_vestingPlan].buyerFee` and `SecondSwap_VestingManager::vestingSettings[_vestingPlan].sellerFee` can be set as -1, which triggers the fallback to the default fees set in the MarketplaceSettings. This could potentially lead to unexpected behavior if the intent is for these fees to be of positive values.

### Vulnerability Details
Affected [code](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_MarketplaceSetting.sol#L168-L184)
```
SecondSwap_MarketplaceSetting.sol
...
function setBuyerFee(uint256 _amount) external onlyAdmin {
        require(_amount <= 5000, "SS_Marketplace_Settings: Buyer fee cannot be more than 50%");
        buyerFee = _amount;
        emit DefaultBuyerFeeUpdated(_amount);
    }

function setSellerFee(uint256 _amount) external onlyAdmin {
        require(_amount <= 5000, "SS_Marketplace_Settings: Seller fee cannot be more than 50%");
        sellerFee = _amount;
        emit DefaultSellerFeeUpdated(_amount);
    }
```

### Recommendations
Add zero checks in `SecondSwap_MarketplaceSetting::setBuyerFee()` and `SecondSwap_MarketplaceSetting::setSellerFee()` 

## <a id="low-05"></a>L-05: Hardcoded variables in `SecondSwap_MarketplaceSetting` constructor

### Summary
The following variables `buyerFee`, `sellerFee`, `penaltyFee`, `minListingDuration`, `referralFee` and `isMarketplaceFreeze` are hardcoded in the constructor. This may implement inefficiency in the event the users want to set the variables to their custom values and need to perform additional calls to other functions to set them.

### Vulnerability Details
Affected [code](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_MarketplaceSetting.sol#L142-L147)
```
constructor(
        address _feeCollector,
        address _s2Admin,
        address _whitelistDeployer,
        address _vestingManager,
        address _usdt
    ) {
        require(_feeCollector != address(0), "SS_Marketplace_Settings: Invalid fee collector address");
        require(_s2Admin != address(0), "SS_Marketplace_Settings: Invalid admin address");
        feeCollector = _feeCollector;
        s2Admin = _s2Admin;
        buyerFee = 250; // 2.5% fee
        sellerFee = 250; // 2.5% fee
        penaltyFee = 10 ether;
        minListingDuration = 120;
        referralFee = 1000;
        isMarketplaceFreeze = false;
        whitelistDeployer = _whitelistDeployer;
        vestingManager = _vestingManager;
        usdt = IERC20(_usdt);
    }
```

### Recommendations
Add user input parameters for setting the variables

## <a id="low-06"></a>L-06: Redundant casting to uint256() in `SecondSwap_Marketplace::_getFees()`

### Summary
The returned `vpbf` and `vpsf` are already uint256 type. Hence, it is not needed to cast them into uint256() types.

### Vulnerability Details
Affected [code](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_Marketplace.sol#L434-L435)
```
function _getFees(address _vestingPlan) private view returns (uint256 bfee, uint256 sfee) {
        (int256 vpbf, int256 vpsf) = IMarketplaceSetting(marketplaceSetting).getVestingFees(_vestingPlan);
        bfee = vpbf > -1 ? uint256(vpbf) : IMarketplaceSetting(marketplaceSetting).buyerFee(); 
        sfee = vpsf > -1 ? uint256(vpsf) : IMarketplaceSetting(marketplaceSetting).sellerFee();
    }
```
### Recommendations
Remove type casting and use variable directly

## <a id="low-07"></a>L-07: Missing remove whitelist function in `SecondSwap_Whitelist`

### Summary
The `SecondSwap_Whitelist` contract allows users to add themselves into the whitelist and the whitelist simply serves as a mechanism to limit the number of users that have access to private listings. However, the contract is missing a function to allow lotOwner to remove addresses from the whitelist in the event that it has reached the limit of maxWhitelist and spam/grief/fake addresses cannot be removed. This will then prevent other legitimate users from buying.

### Vulnerability Details
https://github.com/code-423n4/2024-12-secondswap/blob/main/contracts/SecondSwap_Whitelist.sol

### Recommendations
Add a function to allow lotOwner to remove addresses from whitelist

## <a id="low-08"></a>L-08: Missing documentation and typo in comments

### Summary
The following comments are missing documentation comments or has typo.

### Vulnerability Details
1. Typo in comments "token address ot the new token"  
Affected [code](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_Marketplace.sol#L170)
```
/**
* @notice Emitted when a new token is added
* @param token address ot the new token
*/
event CoinAdded(address indexed token);
```
2. Missing documentation comment explaining about vestingPlan parameter  
Affected [code](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_Marketplace.sol#L53-L83)
```
/**
* @notice Structure containing all listing information
* @dev Used to store and manage listing details
* @param seller Address of the token seller
* @param total Total amount of tokens initially listed
* @param balance Current remaining amount of tokens
* @param pricePerUnit Price per token unit
* @param listingType Type of listing (PARTIAL or SINGLE)
* @param discountType Type of discount applied
* @param discountPct Discount percentage (0-10000)
* @param listTime Timestamp when listing was created
* @param whitelist Address of whitelist contract if private listing
* @param minPurchaseAmt Minimum amount that can be purchased
* @param status Current status of the listing
* @param currency Address of token used for payment
*/
struct Listing {
    address seller;
    uint256 total;
    uint256 balance;
    uint256 pricePerUnit;
    ListingType listingType;
    DiscountType discountType;
    uint256 discountPct;
    uint256 listTime;
    address whitelist;
    uint256 minPurchaseAmt;
    Status status;
    address currency;
    address vestingPlan;
}
```
### Recommendations
Add in documentation comment and fix the spelling typo

## <a id="gas-01"></a>G-01: Reduce repeated external calls in `SecondSwap_Marketplace::unlistVesting()`

### Summary
`SecondSwap_Marketplace::unlistVesting()` makes multiple calls to the marketplaceSetting contract to retrieve the same data `penaltyFee()`, `s2Admin()` `and usdt()` multiple times. Repeating these calls increases the gas cost unnecessarily.

### Vulnerability Details
Affected [code](https://github.com/code-423n4/2024-12-secondswap/blob/214849c3517eb26b31fe194bceae65cb0f52d2c0/contracts/SecondSwap_Marketplace.sol#L339-L373)
```
function unlistVesting(address _vestingPlan, uint256 _listingId) external isFreeze {
        Listing storage listing = listings[_vestingPlan][_listingId];
        require(listing.status == Status.LIST, "SS_Marketplace: Listing not active");
        require(
            listing.seller == msg.sender || msg.sender == IMarketplaceSetting(marketplaceSetting).s2Admin(),
            "SS_Marketplace: Not the seller"
        );
        uint256 _penaltyFee = 0;
        if (msg.sender != IMarketplaceSetting(marketplaceSetting).s2Admin()) {
            //  3.4. The s2Admin is unable to unlist vesting
            if ((listing.listTime + IMarketplaceSetting(marketplaceSetting).minListingDuration()) > block.timestamp) {
                require(
                    (IMarketplaceSetting(marketplaceSetting).usdt()).balanceOf(msg.sender) >=
                        IMarketplaceSetting(marketplaceSetting).penaltyFee(),
                    "SS_Marketplace: Penalty fee required for early unlisting"
                ); // 3.7. Value difference caused by the same penalty fee
                (IMarketplaceSetting(marketplaceSetting).usdt()).safeTransferFrom(
                    msg.sender,
                    IMarketplaceSetting(marketplaceSetting).feeCollector(), // 3.7. Value difference caused by the same penalty fee
                    IMarketplaceSetting(marketplaceSetting).penaltyFee()
                ); //  3.6. DOS caused by the use of transfer and transferFrom functions
                _penaltyFee = IMarketplaceSetting(marketplaceSetting).penaltyFee();
            }
        }
        IVestingManager(IMarketplaceSetting(marketplaceSetting).vestingManager()).unlistVesting(
            listing.seller,
            _vestingPlan,
            listing.balance
        ); //  3.4. The s2Admin is unable to unlist vesting

        listing.status = Status.DELIST; // 3.3. Buyer can choose listing price
        listing.balance = 0; // 3.3. Buyer can choose listing price

        emit Delisted(_vestingPlan, _listingId, _penaltyFee, msg.sender);
    }
```

### Recommendations
The values returned for `penaltyFee()`, `s2Admin()` `and usdt()` should be cached in local variables at the beginning of the function.
