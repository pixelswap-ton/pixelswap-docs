## *Documentation for [contracts/pixelswap_funding.tact](../contracts/pixelswap_funding.tact)*

Pixelswap Funding Contract  
===

The Funding contract records all user balances in the Pixelswap system.
The default Funding contract uses a storage pattern similar to that of Jetton contracts,
but all TON/Jetton tokens are stored in one Funding Wallet per user.

A user places orders and withdraw requests to his Funding Wallet.
Those requests are validated and accounted for in the Funding Wallet, and are then routed to Settlement or Execution.

New funding contracts may be added to the system in the future, but existing funding contracts will always remain available.

Structs
---

### Struct [SpendOrderResult](../contracts/pixelswap_funding.tact#L88)

**Fields**  

* [ref_po](../contracts/pixelswap_funding.tact#L89): Cell?  
* [gas_spent](../contracts/pixelswap_funding.tact#L90): Int as coins  

User Message Definitions
---

### Message [ProvideWalletInfo](../contracts/pixelswap_funding.tact#L108)

On-chain getter for wallet info.

**Fields**  

* [payload](../contracts/pixelswap_funding.tact#L110): Slice as remaining  
Custom payload to be attached with the return message.

### Message [TakeWalletInfo](../contracts/pixelswap_funding.tact#L114)

Response for `ProvideWalletInfo`.

**Fields**  

* [master](../contracts/pixelswap_funding.tact#L115): Address  
* [fund_id](../contracts/pixelswap_funding.tact#L116): Int as uint32  
* [owner](../contracts/pixelswap_funding.tact#L117): Address  
* [subaccount](../contracts/pixelswap_funding.tact#L118): Int as uint32  
* [caller_is_authorized](../contracts/pixelswap_funding.tact#L119): Bool  
* [payload](../contracts/pixelswap_funding.tact#L120): Slice as remaining  

### Message [ProvideTokenBalance](../contracts/pixelswap_funding.tact#L124)

On-chain getter for token balance.

**Fields**  

* [token_id](../contracts/pixelswap_funding.tact#L125): Address  
* [payload](../contracts/pixelswap_funding.tact#L126): Slice as remaining  

### Message [TakeTokenBalance](../contracts/pixelswap_funding.tact#L130)

Response for `ProvideTokenBalance`.

**Fields**  

* [token_id](../contracts/pixelswap_funding.tact#L131): Address  
* [balance](../contracts/pixelswap_funding.tact#L132): Int as coins  
* [caller_is_authorized](../contracts/pixelswap_funding.tact#L133): Bool  
* [payload](../contracts/pixelswap_funding.tact#L134): Slice as remaining  

Events
---

### Message [TokenBalanceChangeEvent](../contracts/pixelswap_funding.tact#L146)

Event emitted every time the token balance of a user wallet changes.  
The invariant $\sum{\text{balance changes}} = \text{current amount held}$ holds true for all token pairs.  
The user can verify this invariant by summing up all TokenBalanceChangeEvent events,
and comparing with [`token_balance`](#function-token_balancetoken_id-int-int).

**Fields**  

* [token_id](../contracts/pixelswap_funding.tact#L147): Address  
* [diff](../contracts/pixelswap_funding.tact#L148): Int as int128  

## Contract [PixelswapFunding](../contracts/pixelswap_funding.tact#L156)

Default funding contract for Pixelswap.  

This contract is responsible for managing the funding of the Pixelswap protocol.  
All events are emitted by the Funding Wallets. and not by this contract for gas optimization.  
To construct the token balances of the Funding contract, sum up the balances of all Funding Wallets.  

**Fields**  

* [settlement](../contracts/pixelswap_funding.tact#L162): Address  
Settlement contract address, cannot be modified once deployed.
* [fund_id](../contracts/pixelswap_funding.tact#L164): Int as uint32  
Unique ID for each funding contract, cannot be modified once deployed.
* [gas_config](../contracts/pixelswap_funding.tact#L166): FundingGasConfig  
Gas configuration for the contract.

### Initializer [init(settlement: Address, fund_id: Int)](../contracts/pixelswap_funding.tact#L168)

Internal Messages
---

### Receive [OrderExecutionResult_Partial_4](../contracts/pixelswap_funding.tact#L192)

Forward [`OrderExecutionResult`](./pixelswap_messages.md#message-orderexecutionresult) to Funding Wallet.  
This message would never AFR since execution state is irreversible.  
*Execution -> Settlement -> **Funding** -> Funding Wallet*  

### Receive [FundWallet_Partial_2](../contracts/pixelswap_funding.tact#L200)

Forward [`FundWallet`](./pixelswap_messages.md#message-fundwallet) to Funding Wallet.  
*Jetton Wallet --TransferNotification--> Settlement -> **Funding** -> Funding Wallet*  

### Receive [PlaceOrder](../contracts/pixelswap_funding.tact#L211)

Forward [`PlaceOrder`](./pixelswap_messages.md#message-placeorder) to Settlement.  
*Funding Wallet -> **Funding** -> Settlement -> Execution*  

#### Panics

If the sender if not the Funding Wallet for a normal message.  
Otherwise AFR.  

### Receive [WithdrawFunds_Partial_3](../contracts/pixelswap_funding.tact#L230)

Foaward [`WithdrawFunds`](./pixelswap_messages.md#message-withdrawfunds) to Settlement.  
*Funding Wallet -> **Funding** -> Settlement -> Jetton Wallet*  

#### Panics

If the sender if not the Funding Wallet for a normal message.  
Otherwise never.  

### Receive [InternalTransfer](../contracts/pixelswap_funding.tact#L243)

Forward [`InternalTransfer`](./pixelswap_messages.md#message-internaltransfer) to Settlement,
or forward InternalTransfer to Funding Wallet.  
*Funding Wallet -> **Funding** -> Settlement -> (...)*  
*(...) -> Settlement -> **Funding** -> Funding Wallet*  

#### Panics

If the sender if not the Funding Wallet nor Settlement.
Otherwise never.

### Receive [BulkInternalTransfer](../contracts/pixelswap_funding.tact#L275)

Bulk transfer of Jetton or TON to multiple recipients within the same funding system.
*Funding Wallet -> **Funding** --fanout--> Funding Wallets*

### Receive [WithdrawDustFromFunding](../contracts/pixelswap_funding.tact#L293)

Settlement withdraws extra gas.  
*Anyone EOAs -> Settlement -> **Funding***

#### Panics

When there is no dust to claim.

Getter Functions
---

### Function [get_wallet_address(user: Address, subaccount: Int): Address](../contracts/pixelswap_funding.tact#L323)

Returns the Address of the Funding Wallet for a given user.  

### Function [gas_config(): FundingGasConfig](../contracts/pixelswap_funding.tact#L328)

### Function [fund_id(): Int](../contracts/pixelswap_funding.tact#L332)

Pixelswap Funding Wallet Contract  
===
The `AccessControlSingleRoleNonTransferrable` trait gives the contract basic account abstraction capabilities.

## Contract [PixelswapFundingWallet](../contracts/pixelswap_funding.tact#L362)

**Fields**  

* [gas_config](../contracts/pixelswap_funding.tact#L367): FundingWalletGasConfig  
Gas configuration for the wallet.
* [master](../contracts/pixelswap_funding.tact#L369): Address  
Master contract address (Funding).
* [fund_id](../contracts/pixelswap_funding.tact#L371): Int as uint32  
Fund ID of the master.
* [owner](../contracts/pixelswap_funding.tact#L373): Address  
Owner of the wallet.
* [subaccount](../contracts/pixelswap_funding.tact#L375): Int as uint32  
Subaccount of the wallet.
* [user_role](../contracts/pixelswap_funding.tact#L377): map<Address, Bool>  
Addresses who can control funds in the wallet.
* [token_balances](../contracts/pixelswap_funding.tact#L379): map<Address, Int as uint128>  
Token balances of the wallet.

### Initializer [init(master: Address, fund_id: Int, owner: Address, subaccount: Int)](../contracts/pixelswap_funding.tact#L381)

User Messages
---

### Receive [PlaceOrder](../contracts/pixelswap_funding.tact#L415)

Initiate a [`PlaceOrder`](./pixelswap_messages.md#message-placeorder) to Execution.  
***Funding Wallet** -> Funding -> Settlement -> Execution*  

#### AFR

Never.  

#### Panics

If input gas is insufficient.  

### Receive [WithdrawFunds](../contracts/pixelswap_funding.tact#L451)

Withdraw funds from the wallet.
**Funding Wallet** -> Funding -> Settlement -> Jetton Wallet.  

#### Panics

If the sender is not the user of the wallet.  
If the token ID is invalid.  
If the fund ID is invalid.  
If the user has insufficient funds.  

### Receive [InternalTransfer](../contracts/pixelswap_funding.tact#L470)

Transfer funds internally between wallets.  
***Funding Wallet** -> Funding A (-> Settlement -> Funding B) -> **Funding Wallet***  

### Receive [BulkInternalTransfer](../contracts/pixelswap_funding.tact#L492)

Bulk transfer of Jetton or TON to multiple recipients within the same funding system.  
***Funding Wallet** -> Funding -> Funding Wallets*

### Receive [RescueJetton](../contracts/pixelswap_funding.tact#L516)

User rescues Jetton mistakenly sent to the Funding Wallet.

### Receive ["withdraw dust"](../contracts/pixelswap_funding.tact#L522)

User recovers extra TON.

### Receive [ProvideWalletInfo](../contracts/pixelswap_funding.tact#L528)

### Receive [ProvideTokenBalance](../contracts/pixelswap_funding.tact#L540)

Internal Messages
---

### Receive [OrderExecutionResult](../contracts/pixelswap_funding.tact#L559)

Fund the wallet with tokens as a result of [`OrderExecutionResult`](./pixelswap_messages.md#message-orderexecutionresult).  
*Execution -> Settlement -> Funding -> **Funding Wallet***  

#### Panics

Never panics.  

### Receive [FundWallet](../contracts/pixelswap_funding.tact#L569)

Fund the wallet with tokens as a result of [`FundWallet`](./pixelswap_messages.md#message-fundwallet).  
*Jetton Wallet --TransferNotification--> Settlement -> Funding -> **Funding Wallet***  

#### Panics

Never panics.  

Getter Functions
---

### Function [gas_config(): FundingWalletGasConfig](../contracts/pixelswap_funding.tact#L775)

### Function [token_balance(token_id: Address): Int](../contracts/pixelswap_funding.tact#L779)

### Function [token_balances(): map<Address, Int as uint128>](../contracts/pixelswap_funding.tact#L783)

### Function [fund_id(): Int](../contracts/pixelswap_funding.tact#L787)

*Documentation generated by [Tactdoc](https://github.com/microcosm-labs/tact-tools/tree/main/tactdoc) v0.1.3.*
