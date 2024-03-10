# Analysis report

From February 22 to March 10, 2024, I engaged in a security review of Wise Lending via the Code4rena platform, dedicating approximately 24 hours to this task.

Over two days, I immersed myself in the project's documentation, followed by a four-day deep dive into the codebase. The subsequent week was spent analyzing potential security vulnerabilities within the code, leading to the discovery of three medium-severity findings and four non-critical findings.

Medium Security Findings：

- Lack of Reentracy Guard in `_safeTransfer`, `safeTransferFrom`, and `safeApprove` with Low-level Call
- Rapid Growth of _keyId Potentially Leads to DoS Risks
- Front-Running Risk in `enterFarm` and `enterFarmETH` Functions

Beyond these identified issues, I offer the following recommendations for the project's codebase:

1. The prevalent use of `>` for validation checks in the code. While this approach is technically accurate, employing `!=` for such comparisons enhances code clarity and readability.
   
   For example,
   
   https://github.com/code-423n4/2024-02-wise-lending/blob/1240a22a3bbffc13d5f8ae6300ef45de5edc7c19/contracts/WrapperHub/Declarations.sol#L104-L118

   ```solidity

       function setWiseSecurity(
           address _securityAddress
       )
           external
           onlyMaster
       {
           if (address(WISE_SECURITY) > ZERO_ADDRESS) {
               revert AlreadySet();
           }

           WISE_SECURITY = IWiseSecurity(
               _securityAddress
           );
       }
   ```

2. Lack of Two-step procedure for critical operations leaves them error-prone. Several critical operations are done in one function call. This schema is error-prone and can lead to irrevocable mistakes.

   For example,

   https://github.com/code-423n4/2024-02-wise-lending/blob/1240a22a3bbffc13d5f8ae6300ef45de5edc7c19/contracts/WrapperHub/Declarations.sol#L105-L118

    ```solidity
        function setWiseSecurity(
            address _securityAddress
        )
            external
            onlyMaster
        {
            if (address(WISE_SECURITY) > ZERO_ADDRESS) {
                revert AlreadySet();
            } 

            WISE_SECURITY = IWiseSecurity(
                _securityAddress
            );
        }
    ```
3. The project features two `isOwner` functions located in the `MinterReserver` and `PositionNFTs` contracts for ownership verification. The design utilizes two or three equivalent conditions for determining ownership. 

    Taking `MinterReserver/isOwner` as an example, the function verifies ownership through two distinct checks:
      1. Checking reserved key ownership via `reservedKeys`
      2. Checking NFT ownership through `FARMS_NFTS.ownerOf`

    These checks are considered equivalent for the project, based on the following assumptions:

      1. Temporary vs. Permanent Ownership: For certain contract functionalities, both reserved status and NFT ownership suffice to prove a user's control over a key.
      2. Functional Overlap: In the contract's design, reserving keys through `reservedKeys` might serve to grant some form of ownership or rights to users before they obtain the actual NFT, making either condition sufficient for "ownership" due to potential functional overlaps.
      3. Simplified Logic

    However, this design could introduce potential risks such as:
      
      1. Excessive Permissions: If reserved keys (via the `reservedKeys` mapping) imply only temporary or conditional access rights, and NFT ownership represents permanent and irrevocable control, treating both equally might lead to unintended permission expansion. Users could perform operations that should strictly require NFT ownership by merely meeting the lower threshold of reservation.
      2. Inconsistent State: Reserved keys may represent a temporary state expected to be cleared or updated after certain actions (like NFT minting). Failure to properly handle such state transitions elsewhere in the contract could lead to inconsistencies, such as a user holding both a reserved key and its corresponding NFT, potentially violating the contract's business logic.
      3. Confused Logic: Equating two fundamentally different concepts of ownership could muddle the contract's logic, increasing the risk of vulnerabilities and making the contract's security more challenging to audit and verify.
      4. Access Control: Malicious users circumventing the normal NFT minting process while somehow reserving a key might bypass access control checks that require definite NFT ownership.

    To mitigate these risks, consider:
      1. Clear State and Permissions: Define and distinguish the permissions and states represented by reserved keys and NFT ownership clearly, and differentiate the operations allowed by these two states within the contract's logic.
      2. Access Control: Implement fine-grained access control checks to ensure each function of the contract requires the correct ownership state according to the actual business logic.
      3. Transparency and Documentation: Clearly document the meanings of different states and permissions and their roles in the contract's logic to aid in understanding and auditing the contract.
   
      https://github.com/code-423n4/2024-02-wise-lending/blob/1240a22a3bbffc13d5f8ae6300ef45de5edc7c19/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L95-L114

      ```solidity

        function isOwner(
            uint256 _keyId,
            address _owner
        )
            public
            view
            returns (bool)
        {
            if (reservedKeys[_owner] == _keyId) {
                return true;
            }

            if (FARMS_NFTS.ownerOf(_keyId) == _owner) {
                return true;
            }

            return false;
        }

      ```

      https://github.com/code-423n4/2024-02-wise-lending/blob/1240a22a3bbffc13d5f8ae6300ef45de5edc7c19/contracts/PositionNFTs.sol#L222-L243

      ```solidity
       function isOwner(
           uint256 _nftId,
           address _owner
       )
           external
           view
           returns (bool)
       {
           if (_nftId == FEE_MANAGER_NFT) {
               return feeManager == _owner;
           }

           if (reserved[_owner] == _nftId) {
               return true;
           }

           if (ownerOf(_nftId) == _owner) {
               return true;
           }

           return false;
       }
     ```

4. In the code, the project utilizes two different methods to delete `reservedKeys`.

   In `PendlePowerManager`, `reservedKeys[msg.sender] = 0;` is used to delete `reservedKeys`.
   
   https://github.com/code-423n4/2024-02-wise-lending/blob/1240a22a3bbffc13d5f8ae6300ef45de5edc7c19/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L227-L233

   ```solidity
           if (reservedKeys[msg.sender] == _keyId) {
               reservedKeys[msg.sender] = 0;
           } else {
               FARMS_NFTS.burnKey(
                   _keyId
               );
           }
   ```
   
   In `MinterReserver`, the `delete` keyword is used for deletion.

   https://github.com/code-423n4/2024-02-wise-lending/blob/1240a22a3bbffc13d5f8ae6300ef45de5edc7c19/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L122-L133

   ```solidity
           if (_keyId == 0) {
               revert InvalidKey();
           }

           delete reservedKeys[
               _userAddress
           ];

           FARMS_NFTS.mintKey(
               _userAddress,
               _keyId
           );
   ```
   Although the outcomes of these two methods are the same, it might be beneficial to unify these approaches unless there is a specific contextual logic requiring different methods. If there is special contextual logic, it should be clearly documented.


Furthermore, it is recommended that the project consider using established code libraries, such as OpenZeppelin. Many functional features in the project are implemented by the project team, which, while increasing transparency and flexibility, also raises the security risk and the burden of security auditing.

### Time spent:
24 hours