## Title: Malicious sales of positions

## **Severity:**

Low Risk

## **Relevant GitHub Links:**

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L38-L40

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L460

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L714

## **Vulnerability Details:**

The protocol utilizes NFT’s for users to save their collateral and borrows inside, making it possible to trade their whole positions or use them in second-layer contracts.

- [Reference](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L38-L40)

In the current setup, the lack of restrictions on the timing and frequency of modifying a position could lead to issues. Specifically, it opens the door for malicious actors to list profitable positions at inflated prices, then use front running to sell these positions and withdraw all the collateral, potentially selling an empty position.

Although this issue seems beyond the protocol's control and challenging to safeguard against directly, raising awareness among users about this potential attack could diminish its likelihood of occurring.

## **Impact:**

This could result in users buying positions that are essentially empty, at steep prices, leading to financial losses.

## **Tools Used:**

Manual analysis

## **Recommendation:**

The protocol could introduce additional features to temporarily lock a position from modifications, enhancing safety for purchasers. However, this solution requires careful consideration to ensure it does not negatively impact liquidations and loan repayments, which could make implementation challenging. At a minimum, educating users about this potential attack could help mitigate its occurrence.