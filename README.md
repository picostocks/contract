# contract

Default ethereum contract for picostocks assets

## Main features

-	place and cancel bids and asks as on regular exchanges with 0.2% fee collected by the contract + 0.2% fee collected by custodian
-	spending funds collected in contract by the management and dividend payment requires a budget proposal and approval
-	optional investor approval process (KYC) during the first funding round
-	new funding rounds can be conducted if approved within the budget proposal
-	contract manager can be changed with > 50% of the votes (each token represents 1 vote)

## Contract deployment costs

The (optimized with solidity 0.4.25) contract deployment requires ca. 7M gas.
