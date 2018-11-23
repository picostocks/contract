# PicoStocks Asset Contract

Default ethereum contract for picostocks assets [ERCpico]

## Main features

-	trading tokens by placing and canceling bids and asks within the contract as on regular exchanges 
-	trading tokens with 0.2% fee collected by the contract + 0.2% fee collected by custodian
-	spending funds collected in contract by the management and dividend payment requires a budget proposal and approval
-	optional investor approval process (KYC) during the first funding round
-	new funding rounds can be conducted if approved within the budget proposal
-	contract manager can be changed with > 50% of the votes (each token represents 1 vote)
-	contract is compatible with the ERC20 standard

## Contract deployment and initial funding round

The (optimized with solidity 0.4.25) contract deployment requires ca. 7M gas. The constructor requires 3 parameters that can not be changed later:
-	"tokens": number of tokens assigned initially to the owner of the contract
-	"picoid": asset id on [picostocks.com](https://picostocks.com)
-	"symbol": asset code on [picostocks.com](https://picostocks.com)
The last 2 parameters can be ignored if the contract is not used by [picostocks.com](https://picostocks.com). In addition 6 parameters can be provided to automatically initiate the first funding round (ICO):
-	"price": price of one token (in wei)
-	"from": funding round starting block number
-	"to": funding round ending block number
-	"min": required minimum number of tokens to sell (tokens given initially to the owner are included in the calculation)
-	"max": maximum number of tokens to sell (tokens given initially to the owner are included in the calculation)
-	"kyc": if not zero then an address whitelisting is required for each investor (address)
if the "price" is not zero the funding round will be programmed. If price equals zero the funding round parameters will be ignored. In this case the first founding round can be started with the "setFirstInvestPeriod" function by providing the 6 arguments.
If the "kyc" parameter is set to a non-zero value then the owner has to call the "acceptKYC" function providing the address of the future investor before the investor can buy tokens.
To buy tokens the investor calls the "invest" function or just sends ether to the contract. The number of sold tokens will be calculated and transferred to the investor and the remaining funds will be returned in the same call.
In case the minimum number of tokens are not sold in the first funding round then the contract can not initiate any additional funding rounds and anybody is allowed to call the "disinvest" function. This function will split the funds collected by the contract across the investors proportionally to the number of owned tokens (by calling the "payDividend" function).

## Budget proposal and approval and additional funding rounds

The contract owner has to propose a budget to withdraw funds from the contract. The proposal (initiated by calling the "propose" function) includes the dividends per token to be paid ("dividendpershare"), the amount of ether to be approved for the owner ("budget") and optionally the number of new tokens to issue ("tokens") and their new price ("price"). If the number of new tokens is above zero a new funding round will be initiated if the proposal is approved. The owner can not send more than 1 proposal per 4 weeks (4 weeks are defined as 4*60*24*7 blocks).
Within 4 weeks from submission of the proposal all token owners can cast votes to support or oppose the proposal by calling the "voteYes" or "voteNo" functions respectively. Both functions can be called with the proposal id ("id") as parameter to improve security (to prevent a new proposal to arrive before the vote is recorded on the blockchain).
The proposal passes if the number of collected "Yes" votes is bigger than the number of "No" votes at the end of the voting period (4 weeks after the proposal submission), unless the proposal includes a new funding round with a token price that is lower than the token price of the last funding round. To approve a funding round with a lower token price the proposal has to collect an absolute majority of votes (more than 50% "Yes" votes of available votes).
If the proposal is accepted anybody can call the "executeProposal" function to execute it. The execution will approve the spending of the requested funds by the owner, issue a dividend round if the proposed dividend is bigger than 0 and start a new funding round with the provided parameters (number of new tokens and their price). The new investment rounds have fixed timing, the round starts 2 weeks after the proposal execution and lasts 2 weeks (4 weeks after the proposal execution). After the investment round is finished anybody can call the "closeInvestmentPeriod" function to record the new shareholder structure for dividend payments and to record the new voting power distribution.

## Retrieving dividends and funds management

A token owner can retrieve dividends and other collected funds by calling the "withdraw" function. If the optional "amount" parameter equals zero then all funds of the token owner are withdrawn. The token owner can also deposit funds with the "deposit" function and wire funds to a new address within the contract with the "wire" function by providing the "amount" and the destination address ("who"). Deposited funds are accessible only by the depositing address. The "pay" function can be used to send funds to the contract that can be accessed through budget proposals.
Funds collected with the "pay" function, collected during investment rounds and collected as trading fees can be used for dividend payments and to finance the management (owner) of the contract.
The management (owner) can only retrieve funds owned by the contract through budget proposal rounds.

## Changing the owner by token holders

Every token holder can propose a new owner of the contract by calling the "voteOwner" function with the new owner address as parameter. If any address collected more than 50% of votes then the owner is immediately changed to the new address.

## Administrative functions

The owner can set a new owner address by calling the "changeOwner" function.
The owner can also change the "www" string parameter of the contract by calling the "changeWww" function.
The owner can call the "spend" to send part ("amount") of approved funds (proposed and accepted budget) to a new address ("who").
The custodian can set a new custodian address by calling the "changeCustodian" function.
The owner can also execute a call with the "exec" function.

## Trading tokens

Token owner can buy and sell tokens by calling the "buy" and "sell" function respectively. Both functions take the amount of tokens ("amount") and price ("price") as argument. The buy function (is payable) requires funds to be send with the transaction to buy tokens and to place the bid order. The submitted funds must be sufficient to include the 0.4% fee. If the submitted funds are to low to buy requested amount of tokens a smaller amount will be purchased or ordered (bid order placed). Excess funds will be returned immediately in the same transaction. Executing the "sell" function will also result in immediate withdrawal of funds obtained for sold tokens.
Buy and sell orders can be canceled with the "cancelBuy" and "cancelSell" functions by providing the order id as argument. The order ids can be found with the "findBuy" and "findSell" functions. Both functions require as argument at least the address used to place the order. Optionally the minimum and maximum price of the order can be provided. If they are not provided the order if of the order closest to the market price will be returned.
Several functions can be used to analyze the order book. "ordersSell" and "ordersBuy" return the first 64 orders ("id","price","amount","address") and can optionally filter the orders by order address. Functions "whoSell", "whoBuy", "amountSell", "amountBuy", "priceSell", "priceBuy" take the order id as argument and return the "address", the "amount" and the "price". If no order id is provided then the order closest to the market is used.

