
  # 1. Introduction
  this Repport is a part of Sperax hackaton challenge 2 which consists to find any implementation issues and try to fix it regarding to the white-paper included as a material.
  ## 1.1. Background
  The implementation was tested using the following dependencies:
  ----------------------------------------------------------------------------------
  |             dependency              |                 Version                  |
  | :---------------------------------: | :--------------------------------------: |
  |       @openzepplin/contracts        |              Version: 4.3.0              |
  | @openzeppelin/contracts-upgradeable |              Version: 4.3.0              |
  |          Solidity Compiler          | Version: 0.8.0+commit.c7dfd78e.Linux.g++ |
  |               Truffle               |              Version: 5.4.0              |
  |               Node.js               |             Version: 14.17.4             |
  |                 npm                 |             Version: 7.20.5              |
  ----------------------------------------------------------------------------------
 In order to update the contracts implementations with new dependencies versions, the fixes will be based on above dependencies versions.

 you can find the fixed version on this  [link](https://github.com/bayou020/USDs-Gitcoin/)

  # 2. Implementation Issues:
  ## 2.1. Warnings:
  ### 2.1.1. SPDX License:
  those low level warnings includes the SPDX license include as follows: 
  ```
  SPDX license identifier not provided in source file. Before publishing, consider adding a comment containing "SPDX-License-Identifier: <SPDX-License>" to each source file. Use "SPDX-License-Identifier: UNLICENSED" for non-open-source code. Please see https://spdx.org for more information.
  ```
  this warning was found on all files.

### 2.1.2. Unused Local Variables:
 on the file `contracts/oracle/oracle.sol` There was a lot of unused local variables as follows:
 
 (xxx) is used to pin the unused variable.
 `line 214`
 ```javascript
function getETHPrice() public view override returns (uint) {
		(
		xxx	uint80 roundID,
			int price,
		xxx	uint startedAt,
		xxx	uint timeStamp,
		xxx	uint80 answeredInRound
		) = priceFeedETH.latestRoundData();
		return uint(price);
	}


 ```
 `line 224`
 
 ```javascript
 function getAssetPrice(address assetAddress) public view override returns (uint) {
		(
		xxx	uint80 roundID,
			int price,
		xxx	uint startedAt,
		xxx	uint timeStamp,
		xxx	uint80 answeredInRound
		) = priceFeeds[assetAddress].latestRoundData();
		return uint(price);
	}

 ```
 `line 239`
```javascript
 	function getSPAPrice() external view override returns (uint) {
		uint ETHPrice = getETHPrice();

    xxx uint token0PriceMA_NoPrec = token0PriceMA.div(2**112);
        //failing case: 1 ETH > 10^25 SPA or token0PriceMA/2**112 < ETHPrice
        return ETHPrice.mul(2**112).div(token0PriceMA);
	}
```
`line 324`
```javascript
  xxx  uint USDsInflowOneDay = USDsInflow[indexNew].sub(USDsInflow[indexOld]);
```
 ` contracts/strategies/AaveStrategy.sol`
`Line 141`

 the `_aToken` variable was unused here inside the function `_abstractSetPToken`
 ```javascript
  function _abstractSetPToken(address _asset, address _aToken) internal override {
        address lendingPoolVault = _getLendingPoolCore();
        ERC20Upgradeable(_asset).safeApprove(lendingPoolVault, 0);
        ERC20Upgradeable(_asset).safeApprove(lendingPoolVault, uint256(-1));

```
here is the [fix](###-3.1.2.-Unused-Local-Variables:)
## 2.2. Errors:
### 2.2.1. Dependencies import Errors:
Since we use the last version of OpenZepplin contracts library few dependencies path were changed as follows:

`"@openzeppelin/contracts-upgradeable/math/SafeMathUpgradeable.sol";`

**import error found on those files:**
```
contracts/libraries/BancorFormula.sol
contracts/libraries/StableMath.sol
contracts/oracle/Oracle.sol
contracts/strategies/InitializableAbstractStrategy.sol
contracts/token/USDs.sol 
contracts/vault/VaultCore.sol
```
`"@openzeppelin/contracts-upgradeable/token/ERC20/SafeERC20Upgradeable.sol"`

**import error found on those files**

```
contracts/strategies/InitializableAbstractStrategy.sol
contracts/vault/VaultCore.sol 
```

`"@openzeppelin/contracts-upgradeable/proxy/Initializable.sol"`

**import error found on those files**

```
contracts/vault/VaultStorage.sol
contracts/vault/VaultCore.sol
contracts/token/USDs.sol
contracts/strategies/InitializableAbstractStrategy.sol
contracts/oracle/Oracle.sol 
```

# 3. Implementation Fixes:
## 3.1. Warnings:
  ### 3.1.1. SPDX License:
  just add the appropriate license in line 1 of the contract:
  ```
  // SPDX-License-Identifier: <LICENSE_TYPE>
  ```
### 3.1.2. Unused Local Variables:
 `oracle.sol`

The unused local variables will take more gas for executing the (op) code based on the datatype
So if there is a need to use them later just change the state by pushing these variables to some kind of array or mapping. if not just delete them it will take less gas during the operations.[1](https://ethereum.stackexchange.com/questions/38420/compile-error-warning-unused-local-variable)

 ` contracts/strategies/AaveStrategy.sol` `Line 153`

 Replaced `lendingPoolVault` variable with `_aToken`

 ```javascript
    //Fixed unused _aToken Variable by deleting lendingPoolVault variable and assigning it to _aToken one for approval
    function _abstractSetPToken(address _asset, address _aToken) 
        internal
        override
    {
        _aToken = _getLendingPoolCore();
        ERC20Upgradeable(_asset).safeApprove(_aToken, 0);
        ERC20Upgradeable(_asset).safeApprove(_aToken, type(uint256).max); 
```
## 3.2. Errors:
### 3.2.1. Dependencies import Errors:
The above Errors were fixed by changing dependency path as follow:

`"@openzeppelin/contracts-upgradeable/utils/math/SafeMathUpgradeable.sol";`

```
contracts/libraries/BancorFormula.sol
contracts/libraries/StableMath.sol
contracts/oracle/Oracle.sol
contracts/strategies/InitializableAbstractStrategy.sol
contracts/token/USDs.sol 
contracts/vault/VaultCore.sol
```
`"@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol"`

```
contracts/strategies/InitializableAbstractStrategy.sol
contracts/vault/VaultCore.sol 
```

`"@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol"`


```
contracts/vault/VaultStorage.sol
contracts/vault/VaultCore.sol
contracts/token/USDs.sol
contracts/strategies/InitializableAbstractStrategy.sol
contracts/oracle/Oracle.sol 
```

  # 4. Sources:
  
  1- (https://ethereum.stackexchange.com/questions/38420/compile-error-warning-unused-local-variable)

