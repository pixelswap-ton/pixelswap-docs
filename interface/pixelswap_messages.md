## *Documentation for [contracts/pixelswap_messages.tact](../contracts/pixelswap_messages.tact)*

Message definitions for Pixelswap contracts  
===

### Struct [TokenInfo](../contracts/pixelswap_messages.tact#L21)

TokenInfo to be used in the `PlaceOrder` message.  
If the gas fields are set for an input token, this means rejected orders will be refunded to the external wallet.
In this case, the gas will be deducted from the input token amount first,
and then if orders succeed, the gas will be added to the output token's TON amount.  
If gas fields are set for the TON token, they will be spent from the user's TON balance and may never be refunded.  
Note that output to funding/external can be set separately for each token, regardless whether it's input or output.  

**Fields**  

* [token_is_input](../contracts/pixelswap_messages.tact#L23): Bool  
Whether token is input.
* [token_id](../contracts/pixelswap_messages.tact#L25): Address  
Token ID.
* [token_amt](../contracts/pixelswap_messages.tact#L27): Int as coins  
Token amount.
* [token_gas](../contracts/pixelswap_messages.tact#L30): Int? as uint32  
The gas for the transfer in mTON. If this value is too low, the transfer will fail.
This field being non-null implies that the output should be sent to the external wallet.
* [token_forward_milliton](../contracts/pixelswap_messages.tact#L33): Int? as uint32  
The forward TON amount in mTON for Jetton transfer. If not set, the forward value defaults to 0.
If this field is set while the `token1_gas` field is null, the value is spent from the user's TON balance and may never be refunded.

### Function [load_token_info(self: Slice): TokenInfo](../contracts/pixelswap_messages.tact#L39)

Tact implementation of FunC `$TokenInfo$_load(slice sc_0)`.

#### Panics

If payload does not contain valid TokenInfo.

### Struct [ExtraTokens](../contracts/pixelswap_messages.tact#L60)

TokenInfo to be used in the `PlaceOrder` message.  
If the gas fields are set for an input token, this means rejected orders will be refunded to the external wallet.
In this case, the gas will be deducted from the input token amount first,
and then if orders succeed, the gas will be added to the output token's TON amount.  
If gas fields are set for the TON token, they will be spent from the user's TON balance and may never be refunded.  
Note that output to funding/external can be set separately for each token, regardless whether it's input or output.  

**Fields**  

* [token_gas](../contracts/pixelswap_messages.tact#L63): Int? as uint32  
The gas for the transfer of token in `PlaceOrder` message in mTON. If this value is too low, the transfer will fail.  
This field being non-null implies that the output should be sent to the external wallet.  
* [token_forward_milliton](../contracts/pixelswap_messages.tact#L66): Int? as uint32  
The forward TON amount in mTON for Jetton transfer of token in `PlaceOrder` message. If not set, the forward value defaults to 0.
If this field is set while the `token1_gas` field is null, the value is spent from the user's TON balance and may never be refunded.
* [num_tokens](../contracts/pixelswap_messages.tact#L68): Int as uint8  
Number of extra tokens in `rem`. Value in [0,2].
* [rem](../contracts/pixelswap_messages.tact#L70): Slice as remaining  
Slice containing extra tokens `TokenInfo`. May contain maximum of 2 extra tokens.

### Function [load_extra_tokens(c: Cell): ExtraTokens](../contracts/pixelswap_messages.tact#L76)

Tact implementation of FunC `$ExtraTokens$_load(slice sc_0)`.

#### Panics

If payload does not contain valid ExtraTokens.

### Message [PlaceOrder](../contracts/pixelswap_messages.tact#L133)

Place an order to swap tokens or add liquidity.  
Funding Wallet -> Funding -> Settlement -> Execution.  
As far as the Settlement is concerned, the only relevant fields are `fund_id` and `exec_id`.
All other fields are unverified and passed along transparently to the Execution contract.  

All amounts marked as `input` are validated and spent by the Funding Wallet contract. `token_info` may contain additional input amouts.
The spent amount is added to the "virtual balance" of the message, and passed along to Settlement and Execution,
who ultimately spends it by consuming the `PlaceOrder` message, and replying with `OrderExecutionResult`.  
The TON amount attached to this message belongs to the user, and is used to pay for gas fees along the execution chain.  

For the simplest case of a swap from tokenA to tokenB, only a single cell is needed.  

Example payload for Jetton1/Jetton2 swap (swap Jetton1 for Jetton2):
```
PlaceOrder {
    exec_id, fund_id, user, subaccount,
    mode: 1,
    expiry,
    token_is_input: true,
    token_id: {Jetton1 wallet address of Settlement},
    token_amt: {amount of Jetton1},
    out_amount: {amount of Jetton2, taken into account slippage},
    extra_tokens: null,
    rsvd: null,
    routing_code: {pair_id of Jetton1/Jetton2 pair}
}
```
Example payload for Jetton1/Jetton2 add liquidity:
```
PlaceOrder {
    exec_id, fund_id, user, subaccount,
    mode: 128,
    expiry,
    token_is_input: true,
    token_id: {Jetton1 wallet address of Settlement},
    token_amt: {amount of Jetton1},
    out_amount: {amount of Jetton2, taken into account slippage},
    extra_tokens: ExtraTokens { token1_is_input: true, ... },
    rsvd: null,
    routing_code: {pair_id of Jetton1/Jetton2 pair}
}
```

**Fields**  

* [exec_id](../contracts/pixelswap_messages.tact#L135): Int as uint32  
Execution contract ID to which the order should be routed.
* [fund_id](../contracts/pixelswap_messages.tact#L137): Int as uint32  
Funding contract ID from which the order is initiated.
*all fields below are implementation-specific, agreed upon between Funding and Execution*
* [user](../contracts/pixelswap_messages.tact#L140): Address  
User address.
* [subaccount](../contracts/pixelswap_messages.tact#L142): Int as uint32  
Subaccount ID.
* [mode](../contracts/pixelswap_messages.tact#L148): Int as uint8  
Mode of the order. For Streampool:  
Swap: mode = 1: exact_in, output to funding, mode = 2: exact_out, output to funding  
Swap: mode = 3: exact_in, output to extern, mode = 4: exact_out, output to extern  
Multi-hop Swap: ???  
Add liquidity: mode = 128: add LP balanced, mode = 129: add LP unbalanced  
* [expiry](../contracts/pixelswap_messages.tact#L150): Int as uint32  
The expiry timestamp in UNIX epoch divided by 4.
* [token_is_input](../contracts/pixelswap_messages.tact#L152): Bool  
Whether the token is input. Normally, the first token should always be input.
* [token_id](../contracts/pixelswap_messages.tact#L154): Address  
Token ID.
* [token_amt](../contracts/pixelswap_messages.tact#L156): Int as coins  
Token amount.
* [out_amount](../contracts/pixelswap_messages.tact#L159): Int as coins  
For normal swap this is the output token amount. For add liquidity this is the output LP amount.  
The output token ID can be read from the pair config.  
* [extra_tokens](../contracts/pixelswap_messages.tact#L161): Cell?  
May contain another Cell of additional `ExtraTokens`.  
* [rsvd](../contracts/pixelswap_messages.tact#L163): Cell?  
Empty cell.
* [routing_code](../contracts/pixelswap_messages.tact#L174): Slice as remaining  
Routing code. May contain 1 further ref.  
For Streampool, the following routing code is used:  
- pair_id_0: u32  
(for swap)  
- routing_mode: u8(optional)  
- pair_id_1: u32(optional)  
- ref_0: Cell (used for advanced routing)  
(for add liquidity)  
- ref_0: Cell  
    - lp_amount_out_min: u128

### Message [PlaceOrder_Partial_4](../contracts/pixelswap_messages.tact#L181)

Partial message to save gas.

**Fields**  

* [exec_id](../contracts/pixelswap_messages.tact#L182): Int as uint32  
* [fund_id](../contracts/pixelswap_messages.tact#L183): Int as uint32  
* [user](../contracts/pixelswap_messages.tact#L184): Address  
* [subaccount](../contracts/pixelswap_messages.tact#L185): Int as uint32  
* [payload](../contracts/pixelswap_messages.tact#L186): Slice as remaining  

### Message [PlaceOrder_Partial_2](../contracts/pixelswap_messages.tact#L190)

Partial message to save gas.

**Fields**  

* [exec_id](../contracts/pixelswap_messages.tact#L191): Int as uint32  
* [fund_id](../contracts/pixelswap_messages.tact#L192): Int as uint32  
* [rem](../contracts/pixelswap_messages.tact#L193): Slice as remaining  

### Function [load_place_order(self: Slice): PlaceOrder](../contracts/pixelswap_messages.tact#L197)

Tact implementation of FunC `$PlaceOrder$_load(slice sc_0)`.

### Message [OrderExecutionResult](../contracts/pixelswap_messages.tact#L249)

The Execution Engine is responsible for keeping an orderbook and executing trades.  
Execution -> Settlement -> Funding -> Funding Wallet.  
For AMM Execution Engines, the orderbook is a heap of incoming orders, either fulfilled or rejected immediately.  
For Orderbook Execution Engines, the Execution is responsible for generating/broadcasting orderIds.
It may identify subsequent modifications to an order by requiring the orderId to be passed in the routing code.
After some or an infinite amount of time, the Execution contract will execute/cancel/reject the order with 1 to N `OrderExecutionResult` messages.  
Any amount specified in the output is deducted from the "virtual balance" of Execution and credited to the user's account.  
Trading fees are managed by the Execution contract, and are usually not part of the `OrderExecutionResult` message.  
`OrderExecutionResult`s are final and shall never be rejected downstream.  
An `OrderExecutionResult` may contain another `OrderExecutionResult` message in the `ref` field.  
An `OrderExecutionResult` without a ref may be used to send only TON by specifying `token_id` as TON,
or send TON and Jetton by adding any additional TON to the user specified value of the `forward_milliton` field.  
If an external `OrderExecutionResult` sends tokens to a funding wallet, then all nested `OrderExecutionResult`s also send tokens to a funding wallet,
regardless of their token_gas settings.
If an external `OrderExecutionResult` sends tokens to an external wallet, then the nested `OrderExecutionResult` may send tokens to an external wallet or to a funding wallet,

**Fields**  

* [exec_id](../contracts/pixelswap_messages.tact#L251): Int as uint32  
Execution contract ID from which the order is initiated.
* [fund_id](../contracts/pixelswap_messages.tact#L253): Int as uint32  
Funding contract ID to which the order should be routed.
* [user](../contracts/pixelswap_messages.tact#L255): Address  
User to transfer the funds to.
* [subaccount](../contracts/pixelswap_messages.tact#L257): Int as uint32  
Subaccount to transfer the funds to.
* [pair_id](../contracts/pixelswap_messages.tact#L259): Int as uint32  
Pair Id.
* [token_id](../contracts/pixelswap_messages.tact#L261): Address?  
Token ID. If null then the `token_*` fields are ignored. If ID is 0, then the token is TON and `token_amt` means TON.
* [token_amt](../contracts/pixelswap_messages.tact#L263): Int as coins  
Token amount.
* [token_gas](../contracts/pixelswap_messages.tact#L266): Int? as uint32  
Gas to pay for the transfer. To be added to the user's TON value if token is TON.
This field being non-null implies that the output should be sent to the external wallet.
* [token_forward_milliton](../contracts/pixelswap_messages.tact#L268): Int? as uint32  
Forward milliton. To be added to the user's TON balance/value if output to funding or if token is TON.
* [ton_amt](../contracts/pixelswap_messages.tact#L270): Int as coins  
Additional TON amount to send.
* [ref](../contracts/pixelswap_messages.tact#L273): Cell?  
The ref may contain another `OrderExecutionResult` message.  
Execution must make sure inputs to those `OrderExecutionResult` messages are valid (no token is created out of thin air).  

### Message [OrderExecutionResult_Partial_4](../contracts/pixelswap_messages.tact#L277)

Partial message to save gas.

**Fields**  

* [exec_id](../contracts/pixelswap_messages.tact#L278): Int as uint32  
* [fund_id](../contracts/pixelswap_messages.tact#L279): Int as uint32  
* [user](../contracts/pixelswap_messages.tact#L280): Address  
* [subaccount](../contracts/pixelswap_messages.tact#L281): Int as uint32  
* [payload](../contracts/pixelswap_messages.tact#L282): Slice as remaining  

### Function [load_order_execution_result(self: Slice): OrderExecutionResult](../contracts/pixelswap_messages.tact#L286)

Tact implementation of FunC `$OrderExecutionResult$_load(slice sc_0)`.

### Message [FundWallet](../contracts/pixelswap_messages.tact#L318)

Add funds to a user's Funding Wallet.
Jetton Wallet --TransferNotification--> Settlement -> Funding -> Funding Wallet.
Settlement sends this message after receiving a `JettonTransferNotification`,
if the routing destination is set to funding.

**Fields**  

* [source_user](../contracts/pixelswap_messages.tact#L321): Address  
transaction originator
This parameter is set in BulkInternalTransfer to define the user to whom gas will be returned.
* [user](../contracts/pixelswap_messages.tact#L323): Address  
User destination.
* [subaccount](../contracts/pixelswap_messages.tact#L325): Int as uint32  
Subaccount destination.
* [token_id](../contracts/pixelswap_messages.tact#L326): Address?  
* [token_amt](../contracts/pixelswap_messages.tact#L327): Int as coins  
* [ton_amt](../contracts/pixelswap_messages.tact#L328): Int as coins  
* [routing_code](../contracts/pixelswap_messages.tact#L330): Slice as remaining  
Custom routing code, to be used by the receiving Funding contract.

### Message [FundWallet_Partial_2](../contracts/pixelswap_messages.tact#L334)

Partial message to save gas.

**Fields**  

* [source_user](../contracts/pixelswap_messages.tact#L335): Address  
* [user](../contracts/pixelswap_messages.tact#L336): Address  
* [subaccount](../contracts/pixelswap_messages.tact#L337): Int as uint32  
* [payload](../contracts/pixelswap_messages.tact#L338): Slice as remaining  

### Message [WithdrawFunds](../contracts/pixelswap_messages.tact#L347)

Withdraw funds from a user's Funding Wallet.  
Funding Wallet -> Funding -> Settlement -> Jetton Wallet.  
This method can be used to withdraw both TON and Jetton at the same time.  
The `forward_milliton` and `gas_transfer` fields are used to specify the amount of TON to transfer and the gas to pay for the transfer.
Those amounts are deducted from the user's Funding Wallet's TON balance and not from `contxet().value`.
If the user specifies too little gas, withdraw might fail definately without recourse.

**Fields**  

* [fund_id](../contracts/pixelswap_messages.tact#L349): Int as uint32  
Funding contract ID from which the order is initiated.
* [user](../contracts/pixelswap_messages.tact#L351): Address  
User who initiated the withdrawal.
* [subaccount](../contracts/pixelswap_messages.tact#L353): Int as uint32  
Subaccount who initiated the transfer.
* [token_id](../contracts/pixelswap_messages.tact#L355): Address  
Token ID of the Jetton to withdraw.
* [token_amt](../contracts/pixelswap_messages.tact#L357): Int as coins  
Amount of the Jetton to withdraw.
* [gas_transfer](../contracts/pixelswap_messages.tact#L359): Int as uint32  
Gas to pay for the transfer. If token_id is TON, this field is ignored.
* [forward_milliton](../contracts/pixelswap_messages.tact#L361): Int as uint32  
Amount of TON to withdraw in addition to Jetton. If token_id is TON, this field is ignored.

### Message [WithdrawFunds_Partial_3](../contracts/pixelswap_messages.tact#L365)

Partial message to save gas.

**Fields**  

* [fund_id](../contracts/pixelswap_messages.tact#L366): Int as uint32  
* [user](../contracts/pixelswap_messages.tact#L367): Address  
* [subaccount](../contracts/pixelswap_messages.tact#L368): Int as uint32  
* [payload](../contracts/pixelswap_messages.tact#L369): Slice as remaining  

### Message [InternalTransfer](../contracts/pixelswap_messages.tact#L374)

Transfer funds internally between Funding Wallets, potentially across multiple Funding contracts.
Funding Wallet -> Funding A (-> Settlement -> Funding B) -> Funding Wallet.

**Fields**  

* [token_id](../contracts/pixelswap_messages.tact#L376): Address  
Token ID of the Jetton or TON to transfer.
* [token_amt](../contracts/pixelswap_messages.tact#L378): Int as coins  
Amount of the Jetton or TON to transfer.
* [from_fund_id](../contracts/pixelswap_messages.tact#L380): Int as uint32  
Funding contract ID from which the transfer is initiated.
* [user](../contracts/pixelswap_messages.tact#L382): Address  
User who initiated the transfer.
* [from_subaccount](../contracts/pixelswap_messages.tact#L384): Int as uint32  
Subaccount who initiated the transfer.
* [to_fund_id](../contracts/pixelswap_messages.tact#L386): Int as uint32  
Funding contract ID to which the funds should be transferred.
* [recipient](../contracts/pixelswap_messages.tact#L388): Address  
Recipient of the funds.
* [to_subaccount](../contracts/pixelswap_messages.tact#L390): Int as uint32  
Subaccount of the recipient.
* [routing_code](../contracts/pixelswap_messages.tact#L392): Slice as remaining  
Custom routing code, to be used by the receiving Funding contract.

### Struct [SingleTransfer](../contracts/pixelswap_messages.tact#L396)

A single transfer in [`BulkInternalTransfer`](#message-bulkinternaltransfer).

**Fields**  

* [token_amt](../contracts/pixelswap_messages.tact#L398): Int as coins  
Amount of Jetton to transfer.
* [ton_amt](../contracts/pixelswap_messages.tact#L400): Int as coins  
Amount of TON to transfer.
* [recipient](../contracts/pixelswap_messages.tact#L402): Address  
Recipient of the transfer.
* [subaccount](../contracts/pixelswap_messages.tact#L404): Int as uint32  
Subaccount of the recipient.

### Message [InternalBulkInternalTransfer](../contracts/pixelswap_messages.tact#L407)

**Fields**  

* [fund_id](../contracts/pixelswap_messages.tact#L409): Int as uint32  
Funding contract id
* [user](../contracts/pixelswap_messages.tact#L411): Address  
User who initiated the transfer.
* [from_subaccount](../contracts/pixelswap_messages.tact#L413): Int as uint32  
Subaccount who initiated the transfer.
* [token_id](../contracts/pixelswap_messages.tact#L415): Address?  
Token ID of the Jetton or TON to transfer.
* [num_entries](../contracts/pixelswap_messages.tact#L417): Int as uint16  
Number of entries.
* [entries](../contracts/pixelswap_messages.tact#L419): map<Int as uint16, SingleTransfer>  
Entries of the transfer.

### Message [BulkInternalTransfer](../contracts/pixelswap_messages.tact#L426)

Make an internal transfer to multiple funding wallets.  
Can only target other funding wallets within the same funding contract.  
Can transfer Jetton and TON at the same time,
but can only transfer one Jetton token type at a time.  

**Fields**  

* [user](../contracts/pixelswap_messages.tact#L428): Address  
User who initiated the transfer.
* [from_subaccount](../contracts/pixelswap_messages.tact#L430): Int as uint32  
Subaccount who initiated the transfer.
* [token_id](../contracts/pixelswap_messages.tact#L432): Address?  
Token ID of the Jetton or TON to transfer.
* [num_entries](../contracts/pixelswap_messages.tact#L434): Int as uint16  
Number of entries.
* [entries](../contracts/pixelswap_messages.tact#L436): map<Int as uint16, SingleTransfer>  
Entries of the transfer.

### Message [WithdrawDustFromFunding](../contracts/pixelswap_messages.tact#L441)

Withdraw extra gas from Funding.
Anyone (EOA) -> Settlement -> Funding.

**Fields**  

* [fund_id](../contracts/pixelswap_messages.tact#L443): Int as uint32  
Funding contract ID to withdraw gas

### Message [WithdrawDustFromExec](../contracts/pixelswap_messages.tact#L448)

Withdraw extra gas from Exec.
Anyone (EOA) -> Settlement -> Exec.

**Fields**  

* [exec_id](../contracts/pixelswap_messages.tact#L450): Int as uint32  
Exec contract ID to withdraw gas

*Documentation generated by [Tactdoc](https://github.com/microcosm-labs/tact-tools/tree/main/tactdoc) v0.1.3.*
