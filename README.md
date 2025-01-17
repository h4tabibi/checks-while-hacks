## Checks while Hacks
This repository contains my ultimate solidity attack vectors compilation. <br>
I will be compiling all solidity attack vectors that I come across with.<br><br>
thanks to `transmisions11/Solcurity` for a kickstart :)

## To-Do List
- [x] [`Solcurity`](https://github.com/transmissions11/solcurity)
- [x] [Solidity Attack Vectors `Quillhash`](https://github.com/Quillhash/Solidity-Attack-Vectors)
- [x] [Defi Attack Vectors `Quillhash`](https://github.com/Quillhash/DeFi-Attack-Vectors)
- [x] [NFT Attack Vectors `Quillhash`](https://github.com/Quillhash/NFT-Attack-Vectors)
- [x] [smart-contract-vulnerabilities by @0xKaden](https://github.com/kadenzipfel/smart-contract-vulnerabilities)
- [ ] [`Secureum` Audit Findings 101](https://secureum.substack.com/p/audit-findings-101)
- [ ] [`Secureum` Audit Findings 201](https://secureum.substack.com/p/audit-findings-201)
- [ ] [`Solidity Lab` by Guardian Audits](https://lab.guardianaudits.com/encyclopedia-of-solidity-attack-vectors)

Previous Audits : 
- [x] Caviar AMM December 2022
- [x] Caviar AMM April 2023
- [x] ENS November 2022

## Sections 
1. Approach : Contains the general points during auditing.
2. General Entity : Common questions arise while observing specific term in the smart contract.
3. Variables : Points related to state variables.
4. Structs : Points related to structs.
5. Functions : Points related to functions.
6. Modifiers : Points related to modifiers.
7. Code : Some good practices to avoid any vulnerability.
8. External Call : Points related to External Call.
9. Static Call : Points related to Static Call.
10. Events : Points related to Events.
11. Contract : Some points related to the whole contract
12. Project : Best practices during building a project.
13. Defi : Contains some DeFi related vulnerabilities.
14. After Transaction : Security issues that can arise after the transaction has been submitted to the mempool.
15. NFT : Issues specific to NFTs.

## Approach
- Read the project's docs, specs, and whitepaper to understand what the smart contracts are meant to do.
- Construct a mental model of what you expect the contracts to look like before checking out the code.
- Glance over the contracts to get a sense of the project's architecture.
- Compare the architecture to your mental model. Look into areas that are surprising.
- Identify relevant global and state variables, functions, equations that are involved in the contract.
- List all the invariants related to them and try to find a way to break them to get a loop hole in the implementation.
- Look at areas that interface with external contracts and ensure all assumptions about them are valid.
- Do a generic line-by-line review of the contracts.
- Do another review from the perspective of every actor in the threat model.
- Glance over the project's tests + code coverage and look deeper at areas lacking coverage.
- Run static analysers and review their output.
- Look at related projects and their audits to check for any similar issues or oversights.
- Try to figure out as many as expected invariants in the contract after getting its context.
- Try to avoid `transaction order dependence` in the code or find a way to deal with it.
- Try to anticipate what will occur when governance turns evil (this may be the case of the RUG PULL, EXIT SCAMS).
- Comment the "why" as much as possible. 
- Comment the "what" if using obscure syntax or writing unconventional code.
- Comment explanations + example inputs/outputs next to complex and fixed point math.
- Comment explanations wherever optimizations are done, along with an estimate of much gas they save.
- Comment explanations wherever certain optimizations are purposely avoided, along with an estimate of much gas they would/wouldn't save if implemented.

## Common questions to ask when we come across any general entity
1. Will the contract run the same if this entity is removed?
2. Will this entity be replaced with some alternative code?
3. Will this entity be used by the admin to do some exploit(making the protocol apparently centralised)?
4. Is the entity opening a path for arbitrary interaction with the contract?

## Variables

1. Is the visibility set? Can it be more specific such as `external`, `internal`, `private`? 
2. Can it be `constant`,`immutable`?
3. Is the purpose of the variable and other important information documented using `natspec`?
4. Can it be packed with an adjacent storage variable?
5. Can it be packed in a struct with more than 1 other variable?
6. Use full 256 bit types unless packing with other variables.
7. If it's a public array, is a separate function provided to return the full array?
8. Check that the size of the array to be limited, otherwise it may lead to gas shortage to complete the transaction.
9. Only use `private` to intentionally prevent child contracts from accessing the variable, prefer `internal` for flexibility.
10. Uninitialized local storage variables(variables that take their value from a state variable) can point to unexpected storage locations in the contract, which can lead to intentional or unintentional vulnerabilities, so mark them as memory, calldata and storage as per the requirement.


## Functions

1. Should it be `external`or `internal`?
2. Should it be `payable`?
3. Can it be combined with another similar function?
4. Validate all parameters are within safe bounds, even if the function can only be called by a trusted users.
5. Always make sure that the argument passed is a valid argument/ behaves as expected in its full range of taking values.
6. Are the multiple arrays taken have same length?
7. Is the `checks` before `effects` pattern followed? (SWC-107)
8. Is the `update` before `call` pattern followed? (Reentrancy) Sometimes even the modifier can not save from reentrancy.
9. Check for front-running possibilities, such as the approve function.(SWC-114)
10. Are the correct modifiers applied, such as `onlyOwner`/`requiresAuth`?
11. Are the `modifiers`(if more than one) written in funtion in correct order, because the change in order will change the code?
12. Write down and test invariants about state before a function can run correctly.
13. Write down and test invariants about the return or any changes to state after a function has run and try to include all edge cases as input.
14. Take care when naming functions, because people will assume behaviour based on the name.
15. If a function is intentionally unsafe (to save gas, etc), use an unwieldy name to draw attention to its risk.
16. Are all arguments, return values, side effects and other information documented using `natspec`?
17. Only use `private` to intentionally prevent child contracts from calling the function, prefer `internal` for flexibility.
18. Use `virtual` if there are legitimate (and safe) instances where a child contract may wish to override the function's behaviour.
19. Are return values always assigned?, sometimes not assigning values is better.
20. Try not to use `msg.value`, after its value has been used as this can cause the loss of funds of the contract. `msg.value` can be used in case of fees payment which is very small and protocol exclusive.

## Modifiers

1. Are no storage updates made (except in a reentrancy lock)?
2. Are `external calls` avoided?
3. Is the purpose of the modifier and other important information documented using `natspec`?
4. Always remember that `modifiers` increase the codesize so use them wisely.


## Code

1. Using SafeMath or 0.8 checked math? (SWC-101)
2. Are any storage slots read multiple times?
3. Implementation should be consistent with the documentation, whats written in docs should be implemented in the contract.
4. Are any unbounded loops/arrays used that can cause DoS? (SWC-128)
5. Use `block.timestamp` only for long intervals. (SWC-116)
6. Don't use block.number for elapsed time. (SWC-116)
7. Do not update the length of an array while iterating over it.
8. Don't use `blockhash()`, etc for randomness. (SWC-120)
9. Are signatures protected against replay with a nonce and `block.chainid`? (SWC-121)
10. Ensure all signatures use EIP-712. (SWC-117 SWC-122)
11. Output of `abi.encodePacked()` shouldn't be hashed if using >2 dynamic types. Prefer using `abi.encode()` in general. (SWC-133)
12. Don't use any arbitrary data while using the assembly. (SWC-127)
13. Don't assume a specific ETH balance. (SWC-132)
14. Private data isn't private, it can be accessed. (SWC-136)
15. Updating a struct/array in memory won't modify it in storage.
16. Never shadow state variables. (SWC-119)
17. Try not to mutate function parameters.
18. Is calculating a value on the fly cheaper than storing it?
19. Are all state variables read from the correct contract (master vs. clone)?
20. Are comparison operators used correctly (`>`, `<`, `>=`, `<=`), especially to prevent off-by-one errors?
21. Are logical operators used correctly (`==`, `!=`, `&&`, `||`, `!`), especially to prevent off-by-one errors?
22. Always multiply before dividing, unless the multiplication could overflow.
23. Are magic numbers replaced by a constant with an intuitive name?
24. If the recipient of ETH had a fallback function that reverted, could it cause DoS? (SWC-113)
25. Use SafeERC20 or check return values safely.
26. Don't use `msg.value` if recursive delegatecalls are possible (like if the contract inherits `Multicall`/`Batchable`).
27. Don't assume `msg.sender` is always a relevant user.
28. Don't use `assert()` unless for fuzzing or formal verification. (SWC-110)
29. Don't use `tx.origin` for authorization. (SWC-115)
30. Don't use `address.transfer()` or `address.send()`. Use `.call.value(...)("")` instead. (SWC-134)
31. When using low-level calls, ensure the contract exists before calling.
32. When calling a function with many parameters, use the named argument syntax.
33. Do not use assembly for create2. Prefer the modern salted contract creation syntax.
34. Do not use assembly to access chainId or contract code/size/hash. Prefer the modern Solidity syntax.
35. Use the `delete` keyword when setting a variable to a zero value (`0`, `false`, `""`, etc).
36. Use `unchecked` blocks where overflow/underflow is impossible, or where an overflow/underflow is unrealistic on human timescales (counters, etc). Comment explanations wherever `unchecked` is used, along with an estimate of how much gas it saves (if relevant).
37. Do not depend on Solidity's arithmetic operator precedence rules. In addition to the use of parentheses to override default operator precedence, parentheses should also be used to emphasise it.
38. Expressions passed to logical/comparison operators (`&&`/`||`/`>=`/`==`/etc) should not have side-effects.
39. Wherever arithmetic operations are performed that could result in precision loss, ensure it benefits the right actors in the system, and document it with comments. 
40. Document the reason why a reentrancy lock is necessary whenever it's used with an inline or `@dev` natspec comment.
41. When fuzzing functions that only operate on specific numerical ranges use modulo to tighten the fuzzer's inputs (such as `x = x % 10000 + 1` to restrict from 1 to 10,000).
42. Use ternary expressions to simplify branching logic wherever possible.
43. When operating on more than one address, ask yourself what happens if they're the same.
44. Can someone without spending other then gas fees change the state of the contract.
45. Always check the number of loop iterations should be bounded by a small finite number other wise the transaction will run out of gas.
46. Always check for the return datatype of the called contract function. Example: in ERC20 implementations, the transfer functions are not consistent with             the value they return(some return the bool while others revert which can cause problems)
47. You can always convert `bool` to `revert` by using `require`.
48. Similar to the above, global `transfer` method reverts while the `send` gives the bool value which sometimes causes problems
49. Don't use `extcodesize` to gain the knowledge of whether the `msg.sender` is EOA as any contract calling the function while staying in the constructor can easily act as an EOA.
50. Try to monitor the expected and actual length of the array.
51. Always try to be consistent with the interface contract otherwise the call will lead to the fallback.
52. Making a new owner is a crucial thing, so a new function to accept the ownership should be made so that the ownership don't go in the hands of some wrong person or a smart contract which can not do anything.
53. In Solidity any address can be casted into specific contract, even if the contract at the address is not the one being casted. This can be exploited to hide malicious code.
54. don't use `erecover` and `signature` to verify the user as these cause signature malleability.
55. delete every entry of the mapping before deleting the mapping itself, otherwise the getter function will still work by giving all the mapping values
56. Look out for signature replay attacks.
57. Use underscores or constants for number literals for better readability and also for unexpected human error.
58. Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4
59. Any inconsistency in formula for calculation may cause the loss of the funds and also minting additional funds,<br> 
          example can be use of Math.min(a, b) which change suddenly when the condition changes.
60. Don't assume the implementations of ERC20, ERC721 tokens in their contracts, such as decimals, approve functions etc., coding using this assumption will lead to the casting errors
61. Look for the statements that can be skipped and still takes to the same blockchain state, for example some external call without any return values, some non-relevant require statements.
62. Try to read all the ERC20 Implementations in scope as their definitions can be different from what is expected.

## External Calls

- `X1` - Is an external contract call actually needed?
- `X2` - Avoid delegatecall wherever possible, especially to external (even if trusted) contracts. (SWC-112)
- `X2` - If there is an error, could it cause DoS? Like `balanceOf()` reverting. (SWC-113)
- `X3` - Would it be harmful if the call reentered into the current function?
- `X4` - Would it be harmful if the call reentered into another function?
- `X5` - Is the result checked and errors dealt with? (SWC-104)
- `X6` - What if it uses all the gas provided?
- `X7` - Could it cause an out-of-gas in the calling contract if it returns a massive amount of data?
- `X8` - If you are calling a particular function, do not assume that `success` implies that the function exists (phantom functions).
- `X9` - Its best to be stateless while doing an external delegate call.
- `X10` - Always assume that the external call will fail, now code accordingly.
- `X11` - Try avoiding taking arbitrary input or calldata input for a function that does external call which can make the EOA make the calls in the behalf of the contract.
- `X12` - The external calls from a contract can be made to be failed and still be made the function continue to act if the external call returns a bool, the attacker can just give very enough gas to make the sub-call(call from a contract function to another contract) fail.(Insufficient Gas Greifing)


## Static Calls

1. Is an external contract call actually needed?
2. Is it actually marked as view in the interface?
3. If there is an error, could it cause DoS? Like `balanceOf()` reverting. (SWC-113)
4. If the call entered an infinite loop, could it cause DoS?
5. Is this call supporting Reentrancy? Is this call reading the updated values OR outdated values?

## Events

- `E1` - Should any fields be indexed?
- `E2` - Is the creator of the relevant action included as an indexed field?
- `E3` - Do not index dynamic types like strings or bytes.
- `E4` - Is when the event emitted and all fields documented using natspec?
- `E5` - Are all users/ids that are operated on in functions that emit the event stored as indexed fields?
- `E6` - Avoid function calls and evaluation of expressions within event arguments. Their order of evaluation is unpredictable.
- `E7` - Events should be made for every important change in state made through the contract.


## Contract

- `T1` - Use an SPDX license identifier.
- `T2` - Are events emitted for every storage mutating function?
- `T3` - Check for correct inheritance, keep it simple and linear. (SWC-125)
- `T4` - Use a `receive() external payable` function if the contract should accept transferred ETH.
- `T5` - Write down and test invariants about relationships between stored state.
- `T6` - Is the purpose of the contract and how it interacts with others documented using natspec?
- `T7` - The contract should be marked `abstract` if another contract must inherit it to unlock its full functionality.
- `T8` - Emit an appropriate event for any non-immutable variable set in the constructor that emits an event when mutated elsewhere.
- `T9` - Avoid over-inheritance as it masks complexity and encourages over-abstraction.
- `T10` - Always use the named import syntax to explicitly declare which contracts are being imported from another file.
- `T11` - Group imports by their folder/package. Separate groups with an empty line. Groups of external dependencies should come first, then mock/testing contracts (if relevant), and finally local imports.
- `T12` - Summarize the purpose and functionality of the contract with a `@notice` natspec comment. Document how the contract interacts with other contracts inside/outside the project in a `@dev` natspec comment.
- `T13` - Malicious actors can use the Right-To-Left-Override unicode character to force RTL text rendering and confuse users as to the real intent of a contract.
- `T14` - Try to take into account the c3 linearization when inheriting from two contracts that contain same function with different implementations
          (diamond problem)
- `T15` - The callable functions in a contract are not only the ones visible in the contract code but also the ones which are inherited but are not mentioned in the code itself.
- `T16` - Using `solmate safeTransferLib`, one should also make a function to check whether the token contract exist or not, because this is not included in that library
- `T17` - its a good practice to include the [headers](https://github.com/transmissions11/headers)
- `T18` - The functions should be grouped in the following order as given in the solidity style guide for the auditing process should be smooth <br>
          { constructor, receive function (if exists), fallback function (if exists), external, public, internal, private, view and pure functions last }
- `T19` - Always look for making an extra function(claim) if there is possibility of the funds to be stuck in the contract. This can be seen in the case of airdrops that are generally landed on the protocol contract and a claim function should be made to retrieve them.
- `T20` - In the beginning after deployment of the contract, the state variables are easy to manipulate(especially in defi) since there is not much of the funds locked in the contract, and hence not very much of the funds are required to manipulate the state of the contract, this can lead to the contract being more vulnerable in start

## Project

- `P1` - Use the right license (you must use GPL if you depend on GPL code, etc).
- `P2` - Unit test everything.
- `P3` - Fuzz test as much as possible.
- `P4` - Use symbolic execution where possible.
- `P5` - Run Slither/Solhint and review all findings.
- `P5` - The coverage for the tests should be 100%

## DeFi
`defi has many vulnerabilities outside solidity, so familiarize yourself with the crypto space and its trends`
includes : structuring to avoid AML/CTF, token inflation, fake trends, smurfing, Interlocking Directorate, 
- `D1` - Check your assumptions about what other contracts do and return.
- `D2` - Don't mix internal accounting with actual balances.
- `D3` - Don't use spot price from an AMM as an oracle.
- `D4` - Do not trade on AMMs without receiving a price target off-chain or via an oracle.
- `D5` - Use sanity checks to prevent oracle/price manipulation.
- `D6` - Watch out for rebasing tokens. If they are unsupported, ensure that property is documented.
- `D7` - Watch out for ERC-777 tokens. Even a token you trust could preform reentrancy if it's an ERC-777.
- `D8` - Watch out for fee-on-transfer tokens. If they are unsupported, ensure that property is documented.
- `D9` - Watch out for tokens that use too many or too few decimals. Ensure the max and min supported values are documented.
- `D10` - Be careful of relying on the raw token balance of a contract to determine earnings. Contracts which provide a way to recover assets sent directly to them can mess up share price functions that rely on the raw Ether or token balances of an address.
- `D11` - If your contract is a target for token approvals, do not make arbitrary calls from user input.
- `D12` - Always set a minimum deposit balance to revoke the privilege given to people depositing zero amount
- `D13` - One of the best optimisations can be decreasing the impermenant loss(maybe divide the loss among more people since the overall loss can not be decreased as this will affect the price impact on the AMM)
- `D14` - `Check out for whether governance given to an EOA has infinite minting or approval power(to avoid rug pull, exit scams, circulating price impact)
- `D15` - Look out for slippage tolerance in Defi Dex protocol, this saves from unexpected results and even protects from front running
- `D16` - There is slippage cap in the functions in AMMs but there should also be the cap on time as the slippage cap gives the person assets in a specified range but the real value of the asset can be changed with time, so even if getting the same amount of token, but not at proper time can lead to bad trade.
- `D17` - Try not to approve the token contracts which have onlyOwner functions which have the power to move the funds.
- `D18` - Watch out what if someone with very much money can do(in cases of auction), in these cases a flashloan attack is likely to happen
- `D19` - Functions without any protection(like onlyOwner) are vulnerable to frontrunning so consider what will happen if they are frontrunned.
- `D20` - Fees is a part of many protocols, watch out for the msg.sender, fee payer, funds receiver as different users.
- `D21` - In case of protocols having subscriptions, unregistered, de-registered, expired entries are also different, these should be acting according to the documentation. 

## After Transaction
1. The transaction data can be seen buy the miner, so don't use things like password in the transactions.

## NFT
1. Any smart contract using NFT contracts as input should also include a function to blacklist NFTs so that anyone can not use NFT contracts as inputs that are theft in the past
