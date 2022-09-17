## ctf takehomes

- know what the inital values if state variables are in a smart contract, and how they change overtime, to be able to exploit them.

  - how the internal functions in theb contract affect those initial varaibles (pen and paper works right here)
  - how the internal varaibles change overtime and how can inbalances occur with functions that rely little or heavily on those values and variables
  - you can check the test files for the initial values
  - in short know the initial values, how they change and what can go wrong now or overtime
  - like in finance

- APY -> means :annual percentage yield

## Cross contract reentrancy && Attack thinking level

- think, and kmow your contract first
- a vault(check the shares logic)

## NoteStart

- selfdestruct is big red button that lets you abandon your smart contract and move all remaining Ethers to another address

- constructor too can be payable

- Delegatecall calls a external contract function but updates your own state varaible
- If Alice invokes Bob who does delegatecall to Charlie, the msg.sender in the delegatecall is Alice.

- So, delegatecall just uses the code of the target contract, but the storage of the current contract.

## note

- . If you’re looking at an auction contract and there’s a section where it returns funds to users after being outbid, is there a way to make it return more funds than a user’s balance? No? Is there a way to make the user’s balance higher than what was deposited? No? Is there a way to make the amount deposited be read incorrectly? No? Okay, expand into other problems there could be. Is there a way to make the transaction fail when it returns funds so that no one can bid above you? No? Is there a way to make it so that when it returns funds you immediately call back into the contract and outbid the person who put a bid above you? Yes? Great.

## Gas on code4rena

- look at operations with more calculations, memory , storage

## code4rena

### Questions to ask after understanding the codebase

- who calls what
- who executes code
- when is code executed
- where is the storage and state variables
- why can a contract call an api
- fucking think deeply
- what could go wrong
- can eth or something be locked forever
- what should the owner not be able to do, but is doing

### sturdy

- understand the whole codebase first
- Read line by line and ask for how each operations can be optimized, ask why and why
- Look for wrong error messages
- what can be an issue, what access are we giving the owner or user taht we shouldn't be giving em
- what can, and will go wrong
- what the owner shouldn't be able to do
- perform some fuzzing in your brain, put values in variable and parameters and placeholders, and see what can go wrong
- check for integer overflow

## Gas Optimizations

- MULTIPLICATION/DIVISION BY TWO SHOULD USE BIT SHIFTING
- USE CUSTOM ERRORS RATHER THAN REVERT()/REQUIRE() STRINGS TO SAVE DEPLOYMENT GAS
- FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE
- USING > 0 COSTS MORE GAS THAN != 0 WHEN USED ON A UINT IN A REQUIRE() STATEMENT, This change saves 6 gas per instance
- NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS
- +I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS
- ARRAY>.LENGTH SHOULD NOT BE LOOKED UP IN EVERY LOOP OF A FOR-LOOP
- USE UNCHECKED {} FOR CALCULATIONS THAT CANNOT OVERFLOW
- CHEAPER TO SPLIT STRUCT IF ONLY PART OF IT IS UPDATED FREQUENTLY The currentStrike and currentExpiration fields are updated frequently so they should be in a separate struct rather than re-writing the whole struct every time

- If a variable is assigned at it's declaration and never reassigned, set it as immutable to save gas.

```
contract ExpensiveStateVar {
  uint public MAX_SUPPLY = 10000;
}
```

```
// pragma solidity ^0.8.0;

contract CheapImmutableStateVar {
  uint public immutable MAX_SUPPLY = 10000;
}
```

- When comparing uints, you can save gas costs by strategically picking the optimal operator.

- For example, it is cheaper to use strict < or > operators over <= or >= operators. This is due to the additional EQ opcode which must be performed.

- In the case of a conditional statement, it is further optimal to use != when possible.

```
// pragma solidity ^0.8.0;

// 149 gas
function a() external pure returns(bool) {
  return 1 > 0;
}

// 193 gas
function b() external pure returns(bool) {
  return 1 >= 0;
}

// 237 gas
function c() external pure returns(bool) {
  return 1 != 0;
}

// 164 gas
function d() external pure {
  require(1 > 0);
}

// 208 gas
function e() external pure {
  require(1 >= 0);
}

// 120 gas
function f() external pure {
  require(1 != 0);
}
```

- Payable Functions

- Functions that are not explicitly marked as payable require an initial check that msg.value == 0, which costs 21 gas. In the case that this check does not provide any benefit, it may be optimal to mark your function as payable.

- **Note**: Do not use this pattern unless you understand the security trade-off being made.

- Keeping revert strings under 32-bytes prevents the string from being stored in more than one memory slot.

```
function expensiveRevertStrings() {
  require(a < b; "long revert string over 32 bytes");
}
```

Alternatively you can write comments to map short strings to longer ones in your contract, e.g.:

```
// a: long revert string over 32 bytes
function cheapRevertStrings() {
  require(a < b; "a");
}
```

Ideally, if you are using solidity >= 0.8.4, it is even better to use custom errors to further save on gas.

```
// pragma solidity ^0.8.0;

error CustomError();
contract CustomErrors {
  if (a < b) {
    revert CustomError();
  }
}
```
