#### GAS-1 Consider using Solady's token contracts to save gas

Solady contracts generally more gas optimized than OpenZeppelin 

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/SimpleERC20Clone.sol

#### GAS-2 Consider not using ERC721Enumerable and implementing functionality in contract 

Reduces need to import all functionality and contracts, can just implement an own  counter

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L11

#### GAS-3 Functions guaranteed to revert when called by normal users can be marked payable

Consider marking payable all access controlled functions in contracts e.g. This makes the function call by the caller cheaper due to ow OPCODES resolve with the msg.value checking if value is sent but contract function dont need value 

e.g this onlyReserveRole function 
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L99

check all other access controlled and implement payable


#### GAS-4 Initializers can be marked as payable 

Just like marking constructors as payable this leads to gas savings 

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L682

#### GAS-5 Reduce deployment costs by tweaking contracts' metadata

You can tweak metadata to get bytecode with 00's in certain format that results in cheaper deployment of the smart contracts 

#### GAS-6 Use assembly to emit events 
Emit events in assembly using logs; this reduces overhead costs of solidity 

e.g check all other events 
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L46

