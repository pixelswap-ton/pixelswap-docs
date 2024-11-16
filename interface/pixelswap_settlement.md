## *Documentation for [contracts/pixelswap_settlement.tact](../contracts/pixelswap_settlement.tact)*

Pixelswap Settlement Contract  
===

The Settlement contract is the centerpiece of the Pixelswap system. The Settlement contract is designed to be simple and robust,
and is meant to be never upgraded or replaced, even as new versions for other parts of the system are rolled out.
To keep its storage strictly bounded, it does not store any state of any tokens or user accounts.
The only potentially unbounded storages are maps of allowed Funding contracts and Execution Engines.
In reality the number of those contracts is relatively small, and we mitigate the risk of misuse by requiring TON payments to store those values.

The Settlement contract holds all TON and Jetton tokens of the Pixelswap system, including all tokens of all AMM pairs,
as well as all deposits from users. Users always deposit directly into the Settlement contract, and Settlement credits
those deposits into the corresponding Funding Wallet.

The following invariant should hold for each token:

$\text{total deposits} - \text{total withdrawals} = \text{total balance of funding contracts} + \text{total balance of execution engines}$

Structs
---

### Struct [ProcessOrderExecutionResult](../contracts/pixelswap_settlement.tact#L122)

**Fields**  

* [gas_spent](../contracts/pixelswap_settlement.tact#L123): Int as coins  
* [res](../contracts/pixelswap_settlement.tact#L124): Cell?  
* [inner](../contracts/pixelswap_settlement.tact#L125): Cell?  

### Struct [SpendOrderResult](../contracts/pixelswap_settlement.tact#L128)

**Fields**  

* [ref_po](../contracts/pixelswap_settlement.tact#L129): Cell?  
* [gas_spent](../contracts/pixelswap_settlement.tact#L130): Int as coins  

User Message Definitions
---

### Message [Deposit](../contracts/pixelswap_settlement.tact#L174)

User depositing TON to the contract.  
The amount deposited is the message value minus gas used.  

**Fields**  

* [fund_id](../contracts/pixelswap_settlement.tact#L176): Int? as uint32  
Fund ID to deposit to.
* [subaccount](../contracts/pixelswap_settlement.tact#L178): Int as uint32  
Subaccount ID to deposit to.
* [routing_code](../contracts/pixelswap_settlement.tact#L180): Slice as remaining  
Custom routing code, to be used by the receiving Funding contract.

### Message [Swap](../contracts/pixelswap_settlement.tact#L186)

User swap TON for Jetton.  
The amount_in is the message value minus gas used.  
The output maybe routed to funding or directly to the user.  

**Fields**  

* [exec_id](../contracts/pixelswap_settlement.tact#L188): Int? as uint32  
Exec ID to use.
* [fund_id](../contracts/pixelswap_settlement.tact#L190): Int? as uint32  
Fund ID to use.
* [subaccount](../contracts/pixelswap_settlement.tact#L192): Int as uint32  
Subaccount ID.
* [expiry](../contracts/pixelswap_settlement.tact#L194): Int as uint32  
Expiry timestamp. In Unix epoch / 4.
* [input_is_exact](../contracts/pixelswap_settlement.tact#L196): Bool  
Whether the amount_in is exact or the amount_out is exact.
* [amount_out](../contracts/pixelswap_settlement.tact#L200): Int as coins  
Amount of Jetton to receive.  
If amount_in_exact is true, this is min_amount_out.  
If amount_in_exact is false, this is exact_amount_out.  
* [extra_tokens](../contracts/pixelswap_settlement.tact#L202): Cell?  
May contain another Cell of additional `ExtraTokens`.  
* [routing_code](../contracts/pixelswap_settlement.tact#L204): Slice as remaining  
Routing code for the swap.

### Message [OrderJettonNotification](../contracts/pixelswap_settlement.tact#L208)

The same as `JettonTransferNotification` but formatted a bit differently.

**Fields**  

* [fund_id](../contracts/pixelswap_settlement.tact#L209): Int as uint32  
* [subaccount](../contracts/pixelswap_settlement.tact#L210): Int as uint32  
* [amount](../contracts/pixelswap_settlement.tact#L211): Int as coins  
* [sender](../contracts/pixelswap_settlement.tact#L212): Address  
* [forward_payload](../contracts/pixelswap_settlement.tact#L213): Slice as remaining  

Event Definitions
---

### Message [TokenBalanceChangeEvent](../contracts/pixelswap_settlement.tact#L226)

Event emitted when the token balance of Settlement changes.  
"Token balance" refers to the amount of TON/Jettons held by the Settlement contract.
The invariant $\sum{\text{balance changes}} = \text{current amount held}$ holds true for all token pairs.  
$\text{current amount held} = \sum{\text{balances of funds}} + \sum{\text{balances of execs}}$  
The Settlement contract itself NEVER holds any balance on its own books (except transiently when processing a message).

**Fields**  

* [is_fund](../contracts/pixelswap_settlement.tact#L227): Bool  
* [contract_id](../contracts/pixelswap_settlement.tact#L228): Int as uint32  
* [subaccount](../contracts/pixelswap_settlement.tact#L229): Int as uint32  
* [token_id](../contracts/pixelswap_settlement.tact#L230): Address  
* [diff](../contracts/pixelswap_settlement.tact#L231): Int as int128  

## Contract [PixelswapSettlement](../contracts/pixelswap_settlement.tact#L239)

Settlement contract for Pixelswap.  
When paused, users can no longer make new deposits or trades, but can always withdraw their funds.  

User Messages
---

### Receive [OrderJettonNotification](../contracts/pixelswap_settlement.tact#L338)

Deposit/add liquidity TON-Jetton/swap via Jetton Transfer Notification.  
*User EOA -> **Settlement** --FundWallet--> Funding: Deposit action*  
*User EOA -> **Settlement** --PlaceOrder--> Execution: Add liquidity or swap*  
Only works when the contract is not paused.  
`query_id`: 32 bits (from/to fund_id) + 32 bits (from/to subaccount)  
`amount`: amount of jetton transferred.  
`sender`: sender of the jetton.  
`forward_payload`: payload to forward to the Execution contract.  

Data for `forward_payload` (if number of bits =0, route to funding in deposit mode):  
`route_to_exec`: bool, if true, route to exec, otherwise route to funding.
**Deposit**: none.  
**Swap** or **Add Liquidity**:  
- mode (u8): 1 to 4 (swap mode), 128 or 129 (add liquidity mode)  
- expiry (u32): Unix epoch div 4  
- exec_id (u32): exec_id to route to  
- ref (ref Cell):
    - out_amount (coins)  
    - extra_tokens (Cell?)  
    - routing_code (Slice as remaining): pair_id (u32), other routing code  

#### Refunds tokens to user

- If contract is paused.  
- If payload is invalid.  
- If any other error occurs.  

#### Panics

- If gas check fails.  

### Receive [Swap](../contracts/pixelswap_settlement.tact#L459)

User swap TON for Jetton.  
*User EOA -> **Settlement** -> Execution (-> ...)*  
Only works when the contract is not paused.  
The amount_in is the message value minus gas used.  
The output maybe routed to funding or directly sent to the user depending on `output_to_ext`.  

#### Panics

- If gas check fails.  

#### AFR

- If the contract is paused.  

### Receive [Deposit](../contracts/pixelswap_settlement.tact#L539)

Deposit TON funds to Funding Wallet.  
*User EOA -> **Settlement** -> Funding -> Funding Wallet*  
Only works when the contract is not paused.  
Deposits to the specified fund contract.  

#### Panics

- If gas check fails.  

#### AFR

- If the contract is paused.  
- If the routing code is too long.  
- If the deposit amount is invalid.  
- If the fund contract ID is invalid.  

### Receive [MSGTOOLS__JettonExcesses](../contracts/pixelswap_settlement.tact#L600)

Admin function to withdraw dust from the contract.
Receive Jetton Excess

Internal Messages
---

### Receive [PlaceOrder](../contracts/pixelswap_settlement.tact#L615)

Forward [`PlaceOrder`](./pixelswap_messages.md#message-placeorder) to Execution.  
*Funding Wallet -> Funding -> **Settlement** -> Execution*  

#### Panics

- If the sender is neither the Funding contract (for a normal message) nor Execution (for AFR).  
- If gas check fails.  

#### AFR

- If the exec or fund contract ID is invalid.  
- If the contract is paused (for a normal message).  

### Receive [OrderExecutionResult](../contracts/pixelswap_settlement.tact#L655)

Forward [`OrderExecutionResult`](./pixelswap_messages.md#message-orderexecutionresult) to Funding Wallet.  
This message would never AFR since execution state is irreversible.  
*Execution -> **Settlement** -> Funding -> Funding Wallet*  

#### Panics

If the sender is not the Execution contract.  
If some logic check fails (impossible to reach such paths).  
If the payload data staucture is corrupted.  

### Receive [InternalTransfer](../contracts/pixelswap_settlement.tact#L681)

Forward `InternalTransfer` to another funding contract.  
*Funding A Wallet -> Funding A -> **Settlement** -> Funding B -> Funding B Wallet*  

#### AFR

When the target funding contract is non-existant.  
When the contract is paused.  

#### Panics

Never

### Receive [InternalBulkInternalTransfer](../contracts/pixelswap_settlement.tact#L691)

### Receive [WithdrawFunds](../contracts/pixelswap_settlement.tact#L739)

Withdraw funds.  
*Funding Wallet -> Funding -> **Settlement** -> Jetton Wallet*  

#### Panics

If the sender is not the Funding contract.  
Otherwise never.  

Getter Functions
---

### Function [gas_config(): SettlementGasConfig](../contracts/pixelswap_settlement.tact#L996)

### Function [exec_contracts_count(): Int](../contracts/pixelswap_settlement.tact#L1000)

### Function [fund_contracts_count(): Int](../contracts/pixelswap_settlement.tact#L1004)

### Function [exec_address(exec_id: Int): Address](../contracts/pixelswap_settlement.tact#L1008)

### Function [fund_address(fund_id: Int): Address](../contracts/pixelswap_settlement.tact#L1012)

### Function [default_exec(): Int](../contracts/pixelswap_settlement.tact#L1016)

### Function [default_fund(): Int](../contracts/pixelswap_settlement.tact#L1020)

### Function [default_info(): SettlementDefautInfo](../contracts/pixelswap_settlement.tact#L1024)

### Function [address_to_int(addr: Address): Int](../contracts/pixelswap_settlement.tact#L1035)

*Documentation generated by [Tactdoc](https://github.com/microcosm-labs/tact-tools/tree/main/tactdoc) v0.1.3.*
