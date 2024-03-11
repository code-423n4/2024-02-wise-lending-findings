## Overview 
Wiselending is a defi protocol allowing users to lend and borrow assets against collateralized assets where the borrowing rates are variable and depend on the utilization of the pool. It uses an automated scaling algorithm (LASA) to adjust borrow rates based on pool utilization.

The platform supports various modes of operation, including private deposits and withdrawals, integration with Aave for capital efficiency, special curve pools, and NFT-based position management. Before executing certain operations, the contract checks the health state of the user's position to ensure it's not undercollateralized.



# Challenges

## Complexity Risks
The whole codebase is designed in a modular way, every little detail has been abstracted in different modules which makes single Contract dependent on lot of other contracts.
This is one of the Best effecient engineering practices but in Your case, WiseLending codebase is Overengineered. 

One function depends on lot of other functions  which in itself depends on other functions and on and on which makes developer to easily loose track of complete context and miss critical bugs. Even Auditors get lost in complexity which puts protocol at risk. Since Blachats don't face any deadlines, projects like this automatically becomes their honeypots.



To Demonstrate the complexity of your codebase, examine this syncpool() modifier call graph

///// callgraph

///// callgraph


/// callgraph

And This complexity gets even worse in other functions. My feedback is refactor codebase to make it more simpler and easier.
`What would be the benefits of it?`
- Easier for auditors to understand and find hidden bugs quickly
- Easier for developers to fix bugs.
- Easier to fuzz test.
- Easier to formally verify properties of the codebase.
- Less Gas costs

## State Changes in modifiers
Taking the same example of syncpool(), lot of state changes happen inside this modifier, which violates the principles of object oriented programming.

// CODE EXAMPLES OF STATE CHANGES BY SYNCPOOL

Doing State changes in modifiers is considered bad software engineering practice  for several reasons, primarily revolving around the principles of encapsulation, maintainability, and the potential for introducing bugs or security vulnerabilities.

`Encapsulation Violation:` 
Modifiers are intended to provide a way to check conditions before executing a function. Changing the state within a modifier violates the fundamental principle of OOP i.e, encapsulation, which is a fundamental concept in object-oriented programming.

`Maintainability Issues:` 
When state changes are made within modifiers, it can lead to maintainability issues. If the logic within a modifier is complex or if the state changes are numerous, it becomes difficult to track how the state changes affect the overall behavior of the object. This can lead to bugs that are hard to diagnose and fix. It also makes the code harder to read and understand, as the state changes are not immediately visible in the function body

`potential for Bugs and Security Vulnerabilities:` 
Changing the state within a modifier can introduce bugs and security vulnerabilities. For example, if a modifier changes the state in a way that is not expected by the function it is guarding, it could lead to unexpected behavior. Additionally, if the state changes are not properly validated, it could open up security vulnerabilities, such as allowing unauthorized state changes 1.

`Recommendation:` 
Instead of changing the state within a modifier, it's often better to use separate functions or methods to handle state changes. This approach keeps the modifier focused on its primary role of checking conditions and makes the state changes more explicit and easier to manage. It also allows for better separation of concerns, where each part of the code has a single responsibility 1.

## Gas Costs
The codebase in general especially WiseLending contract includes several complex operations, such as calculating share prices and updating pool data. These operations can be gas-intensive, which could affect the platform's usability and cost-effectiveness.

Refactor codebase and use assembly tricks to optimize your codebase for gas costs. Apart from that,
I recommend to have atleast One comprehensive Gas Audit to make WiseLending cost-effective for users which in turn will attract mass attention.

## External Risks
The WiseLending contract makes external calls to other contracts, such as WISE_SECURITY and POSITION_NFT. It's important to ensure that these contracts are secure and trustworthy. 
Ensure they handle user inputs correctly and doesn't introduce any flaws that attacker can maliciously rig and exploit wiseLending funds.

## Multiple Modes of Depositing and withdrawing assets
- inconsistencies...


There are multiple modes of depositing and withdrawing assets, which in itself isn't the issue but any inconsistency in any one of the functions can be devastation to the protocol and cause unfair situations.
Ensure All of the deposit and withdraw functions work as expected and no devastating scenario can unfold.
- `Deposit`
    - depositExactAmountETH()
    - depositExactAmountETHMint()
    - depositExactAmountMint()
    - depositExactAmou

- `Withdraw`
    - withdrawExactAmountETH()
    - withdrawExactAmount()
    - withdrawOnBehalfExactAmount()
    - withdrawExactShares()
    - withdrawOnBehalfExactAmount()

## Testing
Wiselending is lacking comprehensive test suite. Ensure thorough unit, fuzz, and integration testing before deploying to the mainnet. OracleHub and positionNFT had two compilation errors which weren't caught by your test/ suite. for example in POSITION_NFT _exists causes compiler issue because, openzeppelin had depreciated this function since 2019.

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L161

```c
        if (_exists(tokenId) == false) {
            return ZERO_ADDRESS;
        }
```
But unfortunately Your test suite was ineffective to catch this

`Recommendations`
- Do comprehensive unit test and target to acheive > 85% coverage
- For the complex logic , use fuzz testing

__`Conduct Formal Verification`__
Apart from that one thing i feel wiseLending is in critical need of Formal Verification. Use Formal verification to prove the critical invariants and properties of the Lending and borrowing operations. But Before that, i think you need to refactor your codebase to make it simpler and easier to model as the formal verification tools can timeout due to complexity.

In my opinion, Certora is no doubt the best in industry for formal verification. Conduct Formal Verification audit through certora company.




## Centralization Risks
`onlyMaster()` modifier, highlight a critical aspect of smart contract security and governance. This modifier, serving as a single point of failure, poses a significant risk to the protocol's decentralization and overall security.

The centralization of control in a single entity or role within a smart contract can lead to several adverse outcomes, including:

- Loss of User trust in protocol
- Single Point of failure risks of Master account
- Limited Decision-Making Process tied to single Entity

To mitigate these centralization risks and promote true decentralization, I recommended to implement governance mechanisms within the WiseLending protocol. Governance mechanisms allow for decentralized decision-making, where proposals can be voted on by the community, and changes to critical functions or parameters can be made through a consensus process. This approach ensures that the protocol is more resilient to attacks, reduces the risk of centralization, and aligns the interests of all stakeholders.

Implementing governance can involve various strategies, such as:

- `Token-Based Voting:` Allowing token holders to vote on proposals can democratize decision-making and ensure that the interests of all stakeholders are considered.

- `Quadratic Voting:` This method, which gives more weight to those who hold fewer tokens, can help to balance the interests of large and small stakeholders.
  
- `Delegation:` Allowing token holders to delegate their voting power to others can facilitate more informed decision-making and encourage participation from those who may not have the time or resources to vote themselves.

By adopting governance mechanisms, WiseLending can enhance its security, trustworthiness, and alignment with the principles of decentralization. This approach not only mitigates the centralization risks associated with the onlyMaster() modifier but also fosters a more robust and resilient ecosystem for its users and stakeholders

## Summary Of Recommendations
Here are the final list of recommendations from my analysis report:
- Reduce code complexity
- Avoid state changes in modifiers
- Optimize gas-costs, conduct thorough gas audit
- Ensure robustness of external dependencies like positionnft and wisesecurity contracts
- Conduct thorough unit and fuzz test
- Conduct Formal verfication audit with certora
- Implement governance mechanisms to ensure proper decentralization and avoid single point of failure.
  

### Time spent:
17 hours