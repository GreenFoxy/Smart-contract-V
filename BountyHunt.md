BountyHunt
=======
https://etherscan.io/address/0xb5766f61911f8b520b0e938aae100834aa3048c6#code

Vulnerability type
------
Integer overflow

Reentry

Abstract
------
A smart contract implement for BountyHunt, allows users to call `grantBounty(address beneficiary, uint amount)` to save ethers as bounty in this contract and appoint the beneficiary who will be able to claim this bounty by calling `claimBounty()`.

