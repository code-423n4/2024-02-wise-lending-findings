# Goal of Analysis 

The WiseLending Protocol was analysed for common design flaws, such as access control, arithmetic issues, front-running, denial of service, race conditions, and protocol manipulations. 
 
## Protocol Overview
The WiseLending Protocol is basically a platform that allow users to borrow provided liquidity by other users or investors looking to make profits or gains for their deposits. Now there is a nice functionality that was implemented in this protocol that won't be found in most lending protocols, which is the ability to deposit in a seperate pool that won't be given out to users that want to borrow so in case of too many bad debts, the users would still be able to get their funds with an added APY. 
In summary the WiseLending Protocol enables users to :

### 1) Supply pools with multiple possible stablecoins and earn APY
### 2) Earn passive funds with the use of PowerFarms
### 3) Borrow funds deposited by Lenders
### 4) Take Flashloans  

## Comments on code 
The codebase is well-written, but not sufficiently tested. The following comments point out possible areas for improvement:

### 1.) No presence of Whitelisted Borrowers and Lenders
 In most common lending protocols, it is common for there to be Whitelisted borrowers and Lenders, This specifically helps to grow a connection with the users as they would feel recognized as well as would assist the protocol generate more funds as Whitelisting almost always would need users to accumulate some shares in the contract and this also would help the protocol thrive some more in the Ecosystem

### 2.) Increase Tests for crucial functions and contracts 
 Some crucial contracts in the protocol have little or no tests, This is not really good practice because it would really help if tests are written, It is one thing to write a test and it is another to make it work the way you presume, It would have been really nice to test and see that code functionality is working as planned

### 3.) Create a good documentation for Devs and or Smart Contract Auditors 
 There really wasn't any actual documentation that showed what the codebase did and how to go about the codebase, There was only a link that pointed to the documentation on the WiseLending site, however this was little help as it was created for users and obviously Security Researchers don't have the same thought process as the target audience, in case of another audit I suggest a documentation be prepared for Developers or Security Researchers 

### 4.) Increase comments for essential components.
 There were some contracts and functions that had excellent comments that added to code readability just like this
![WiseLending.sol](C:\Users\USER\Pictures\wise3.jpg)
and 
![setAaveTokenAddressBulk](C:\Users\USER\Pictures\wise2.jpg)
 
but at the same time there were other contracts and functions that weren't properly commented on and this would take a long time trying to decipher just like in
![PendlePowerFarmControllerBase](C:\Users\USER\Pictures\wise4.jpg)
and 
![_syncSupply](C:\Users\USER\Pictures\wise5.jpg)




## Centralization risks
WiseLending is a partially permissioned protocol, which means that it is controlled by a central authority (the Master). This gives the master a great deal of power, including the management of fees, WiseSecurity, and WiseLending contracts. If the operator account is compromised, it could have serious consequences for the protocol, such as the compromise of fees distribution and the security of WiseLending as a whole.

To minimize this risk, it's advisable for the team to employ a multi-signature wallet for overseeing the Master account. A multi-signature wallet mandates multiple signatures from authorized users to validate transactions. This significantly heightens the difficulty for an attacker to breach the account, even if they manage to obtain one of the signatures.

## Highlights: Innovations and best practices

### 1.) Innovative Architecture

I particularly like the way they had a Declarations contract, A helper contract and the actual contract it jut made everything so much easier to locate, if you were looking for functionality you go to the helper contract, Declaration?, visit the declarations contract, Overall logic?, Visit the main contract. It was enjoyable navigating around the codebase with this in mind



### Time spent:
35 hours