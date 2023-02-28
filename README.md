# GoodEntry Markets

The project is a fork of Aave V2 Lending Markets.

The Aave V2 Lending Market protocol is a battle-tested lending market protocol, with over 10B locked over a few years.

We will follow Aave V2 at a snapshot of their development ([https://github.com/aave/protocol-v2/releases/tag/Deployment%23001](https://github.com/aave/protocol-v2/releases/tag/Deployment%23001)  - Aave Avax Deployment).

If any critical bugs are raised in the Aave Deployment, the protocol will strive to immediately pause and upgrade in accordance to the Aave V2 bugfix.

# Main changes
Only the LendingPool file itself has been changed.

We can break down the main changes into the following: 
1. Adding of the role of a PositionManager, which must be a contract: L61, L426-L493
2. Making the PositionManager not enable collateral usage by default on transfer to save gas: L132

# Code Safety Considerations
## For depositors
GoodEntry Markets primarily aim to help liquidity pool depositors earn extra supply yield with minimal additional technical risk. This is achieved by ensuring minimal changes to battle-tested code, and to ensure the new code added cannot be wielded to remove depositor's money.

PositionManagers have the flexibility to transfer ATokens to themselves; this is needed to help the user manage their positions. 

The added code has checks that protect users assets, as long as a user does not:
1) Initiate a call to any PositionManager, and only interact through the Lending Pool Proxy (listed below in Deployment)
2) Reduce their health factor to near liquidation (< 1.01) by borrowing,

GoodEntry Markets borrowers will always need to be sufficiently overcollateralised to keep their positions. However there is a possibility that in adverse market conditions, the lending protocol may take on bad debt when the value of the borrowers' assets do not sufficiently cover the debt. This may lead to a haircut to all depositors.

## For borrowers ...

### Using PositionManagers
The code base of PositionManagers will be separately audited, to ensure that the code flow doesn't compromise the borrowers' funds unexpectedly.

### Using LendingPool native borrowing
Please ensure that your health factor remains above 1.01, if you do not trust the PositionManager's ability to gracefully reduce your leverage. Health factor below 1 will also be subject to the standard Aave liquidation process.

## For all users
As the logic of the Lending Market can be upgraded, it is imperative that users verify every update to the logic, as what was previously safe could be updated to include code that may be unsafe for the users' assets.

Users may read more details here (https://www.certik.com/resources/blog/Timelock) about timelocks, and also how to monitor changes to timelocks so that any unexpected changes can raise alerts to react upon.

Major code safety issues raised may necessitate a protocol pause by the emergencyAdmin - this disables new borrows and deposits, but doesn't stop withdrawals and repayment. This pause will go on until the bug can be patched through the timelock delay of 2 days.

