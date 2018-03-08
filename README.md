# CharityChain

Charitychain is a non-profit open source project.
Charitychain is the first fundraising platform on which fundraisers must give first
- The campaign author deposits 50% of the campaign goal.
- Contributors try to unblock the initial donation by collecting 50% of the goal.
- If they succeed, 100% of the funds go to the NGO, otherwise, everyone is refunded

[![N|Solid](https://charitychain.io/images/diag-readme.png)](https://charitychain.io/)

## Current status of smart contract
The smartcontract is in preparation (work in progress) for a third-party code audit.

## Overview
- This contract manages a single campaign. It's not a Factory
- All the code is in this class, there is no dependency on third-party libraries
- There are less than 200 lines of code.

## Constructor
The constructor can receive funds with the `payable` modifier.
To create a campaign, 3 arguments and an initial payment in Eth are required.

- The name of the campaign
- The end of the campaign (block number)
- The recipient's Ethereum wallet address
- An initial contribution

The campaign is created with a goal equal to twice the first contribution.

## State Machine
 - `CampaignInprogress` 
 - `CampaignFailure`
 - `CampaignSuccess`
 
As long as the end date is not exceeded, the campaign continues, even if the goal is already reached.
 
## Restricting Access
 - Beneficiary
 - Owner

## Contribute 
The `contribute()` transaction is only available in the `CampaignInprogress` state. A `contributionID` is created for each contribution. This ID is associated with the contributor's address.
## Possible scenarios
### The campaign success
If the end date of the campaign is exceeded and the amount collected is greater than the goal, then the current campaign stage becomes `CampaignSuccess`. In this state, the only possible action on the contract is withdrawal of money by the beneficiary.

```sh
    function payoutToBeneficiary() public atStage(Stages.CampaignSuccess) onlybeneficiary() {
        uint256 payoutAmount = this.balance;
        msg.sender.transfer(payoutAmount);
        ...
    }
```
### The campaign Failure
If the campaign is complete (current block above expiry) and the amount collected is less than the goal, then the current campaign stage becomes `CampaignFailure`. Contributors must recover their contributions with the function `withdrawRefundContribution()`
```sh
 function withdrawRefundContribution(uint256 _contributionID) public atStage(Stages.CampaignFailure) validRefund(_contributionID) {
 ...
 }
 ```
### In case of unexpected behavior
There is an emergency mechanism that force expiry of campaign an allows everyone to claim their money depending on the stage of the campaign. Only the Owner can execute this transaction.
```sh
    function emergencyCampaignExpiry() external onlyOwner {
        expiry = 0;
    }
```

## Informations
### events
the smart contracts can fire 3 events: 
```sh
event LogContributionMade (address _contributor);
event LogContributionRefunded(address _payoutDestination, uint256 _payoutAmount);
event LogBeneficiaryPayoutMade (address _payoutDestination, uint256 _amountRaised);
```
### Funding Cap
A `fundingCap` system is implemented (arbitrarily set at 10x fundingGoal), if achieved, he prevents any new contribution.

### Static Analysis
No issues found with solium

### Work In Progress
- Optimizations (especially on the size of int)
- Improved validation rules (especially on the end date when the contract was created)
- Bug Bounty Program
- Runtime Analysis (Unit testing)
- Code Review
- Code Audit

> Credit: This contract has been rewritten several times a year ago, it is now mainly inspired by the Weifund project because in my opinion, it is the simplest and most elegant implementation of crowdfunding smart contract on Ethereum. 
