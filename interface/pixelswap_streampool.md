## *Documentation for [contracts/pixelswap_streampool.tact](../contracts/pixelswap_streampool.tact)*

Pixelswap StreamPool Execution Engine  
===

Execution Engine
---

The Execution Engine is responsible for keeping an internal state of all tokens being traded (either as orderbook or as AMM).
It should add all incoming balances via `PlaceOrder` to its own balance, and deduct all outgoing balances
via `OrderExecutionResult` from its own balance.

From the system's perspective an Execution Engine may arbitrarily decide to withhold any input for an infinite amount of time,
or give any output at any time regardless of whether related to an input or not.
However an Execution Engine must keep the internal balance of all of its tokens positive at any point in time.
Since Settlement does not keep any state about token balances, all Execution Engines must guarantee this condition by themselves.

Multiple Execution Engines supporting different token pairs and with multiple configurations are expected to be added at launch
and more are expected to be added later. Execution Engines also may not be removed and should always remain available.
There is however no rule on the condition under which an Execution Engine must accept orders, and it may refuse to serve new requests
and only allow removing of funds.

StreamPool
---

StreamPool is the default Execution Engine at launch.
It is an AMM type engine. When users add liquidity into the AMM pool, the balance of StreamPool increases.
StreamPool issues a token (the LP token) to reflect ownership of this liquidity by users.
This functionality is implemented using the `JettonFactory` trait.

#### [Streampool Data Structures](#documentation-for-contractspixelswap_streampool_datatact)

#### [Streampool Messages](#documentation-for-contractspixelswap_streampool_messagestact)

## Contract [PixelswapStreamPool](../contracts/pixelswap_streampool.tact#L69)

#### Swap contract for Pixelswap TON/Jetton and Jetton/Jetton StreamPool.

When adding a token pair, a sequential pair id is assigned.
This pair id is used to identify the token in swaps.
This contract is the minter for semi-fungible LP tokens.
3 fee tiers are available: 0.1%, 0.3% and 1%.

##### Roles

**Owner**: Can set the configuration for the unipool. Can set the admin.
Owner should be a multisig wallet.  
**Validators**: Can validate pairs. Can create token and pairs disregarding fees and limitations.
Can collect protocol fee to the protocol fee recipient.  

**Fields**  

* [owner](../contracts/pixelswap_streampool.tact#L77): Address  
The minimum amount of liquidity that must be in the pool.
* [pending_owner](../contracts/pixelswap_streampool.tact#L78): Address  
* [user_role](../contracts/pixelswap_streampool.tact#L79): map<Address, Bool>  
* [jetton_index](../contracts/pixelswap_streampool.tact#L82): Int as uint32 = 0  
* [gas](../contracts/pixelswap_streampool.tact#L89): PoolGasConfig  
* [config](../contracts/pixelswap_streampool.tact#L90): PoolConfig  
* [settlement](../contracts/pixelswap_streampool.tact#L92): Address  
* [exec_id](../contracts/pixelswap_streampool.tact#L93): Int as uint32  
* [tokens_config](../contracts/pixelswap_streampool.tact#L96): map<Address, TokenConfig>  
map (token_id => TokenConfig)
* [tokens_count](../contracts/pixelswap_streampool.tact#L98): Int as uint128 = 0  
total number of tokens
* [pairs_config](../contracts/pixelswap_streampool.tact#L101): map<Int as uint32, PairConfig>  
`pair_id` is the hash
map (pair_id => pair_config)
* [pairs_status](../contracts/pixelswap_streampool.tact#L103): map<Int as uint32, PairStatus>  
map (pair_id => pair_status)
* [pairs_count](../contracts/pixelswap_streampool.tact#L105): Int as uint32 = 0  
total number of pairs
* [creation_fee_recipient](../contracts/pixelswap_streampool.tact#L107): Address  
* [protocol_fee_recipient](../contracts/pixelswap_streampool.tact#L108): Address  

### Initializer [init(owner: Address, settlement: Address, exec_id: Int)](../contracts/pixelswap_streampool.tact#L110)

Internal Messages
---

### Receive [PlaceOrder](../contracts/pixelswap_streampool.tact#L180)

Handle [`PlaceOrder`](./pixelswap_messages.md#message-placeorder) message.  
*Settlement -> **StreamPool** (-> LP::Mint) -> Settlement::OrderExecutionResult*  

#### Requirements

- The sender must be the settlement contract.  

#### Panics

If the sender is not the settlement contract.  
If the exec_id does not match.  
Otherwise AFR.

### Receive [WithdrawDustFromExec](../contracts/pixelswap_streampool.tact#L198)

Settlement withdraws extra gas.  
*Anyone EOAs -> Settlement -> **Streampool***  

#### Panics

When there is no dust to claim.

### Receive [RemoveLiquidityJettonNotification](../contracts/pixelswap_streampool.tact#L223)

Remove liquidity via Jetton Transfer Notification.  
If the sender is not the jetton wallet address of the specified pair, the jetton is sent back.  
Data for `forward_payload`:  
- expiry (u32): Unix epoch div 4  
- pair_id (u32): fund_id to route funds back to.  
- ref (ref Cell):  
    - amount0_min_out: u128;  
    - amount1_min_out: u128;  
    - token0_gas: Int? as uint32;  
    - token0_forward_milliton: Int? as uint32;  
    - token1_gas: Int? as uint32;  
    - token1_forward_milliton: Int? as uint32;  

#### Panics

If insufficient gas is supplied.  
No AFR error path.

### Receive [MSGTOOLS__JettonExcesses](../contracts/pixelswap_streampool.tact#L327)

Silently accept JettonExcesses

User Messages
---

### Receive [CreateToken](../contracts/pixelswap_streampool.tact#L335)

Create a new token.

### Receive [CreateTradingPair](../contracts/pixelswap_streampool.tact#L358)

Create a new trading pair.  
Developers can monitor the event to get `pair_id`, then use `pair_id` to get `pair_config`.  
`token0_address` `token1_address `fee_bps` `weight0` and can be used to query for `pair_id`.  

Getter Functions
---

### Function [config(): PoolConfig](../contracts/pixelswap_streampool.tact#L906)

### Function [number_pairs(): Int](../contracts/pixelswap_streampool.tact#L910)

### Function [pair_id(token0_id: Address, token1_id: Address, fee_bps: Int, weight0: Int): Int](../contracts/pixelswap_streampool.tact#L914)

### Function [pair_config(index: Int): PairConfig?](../contracts/pixelswap_streampool.tact#L924)

### Function [pair_status(index: Int): PairStatus?](../contracts/pixelswap_streampool.tact#L928)

### Function [gas_config(): PoolGasConfig](../contracts/pixelswap_streampool.tact#L932)

### Function [amount_out_for_exact_amount_in(amount_in: Int, pair_id: Int, side: Bool): SwapResult](../contracts/pixelswap_streampool.tact#L940)

Calculate the amount of token output for an exact amount of tokens input.  
`amount_in`: amount of tokens sent.  
`pair_id`: pair id.  
`side`: true if token0 is the input token.

### Function [amount_in_for_exact_amount_out(amount_out: Int, pair_id: Int, side: Bool): SwapResult](../contracts/pixelswap_streampool.tact#L954)

Calculate the amount of token input for an exact amount of tokens output.
`amount_out`: amount of tokens received.
`pair_id`: pair id.
`side`: true if token0 is the input token.

### Function [lp_amount_for_amounts_in(amount0: Int, amount1: Int, pair_id: Int, lp_balanced: Bool): LiquidityResult](../contracts/pixelswap_streampool.tact#L970)

Calculate the amount of LP tokens to mint for adding liquidity.  
`amount0`: amount of token0 to add.  
`amount1`: amount of token1 to add.  
`pair_id`: pair id.  
`lp_balanced`: true if the LP tokens should be minted in a balanced way.  
Returns a `LiquidityResult`.

### Function [token_amounts_for_lp_in(lp_amount: Int, pair_id: Int): LiquidityResult](../contracts/pixelswap_streampool.tact#L980)

Calculate the amount of token0 and token1 to receive for removing liquidity.

### Function [tokens_config(token_id: Address): TokenConfig?](../contracts/pixelswap_streampool.tact#L985)

### Function [tokens_count(): Int](../contracts/pixelswap_streampool.tact#L989)

### Function [exec_id(): Int](../contracts/pixelswap_streampool.tact#L993)

## *Documentation for [contracts/pixelswap_streampool_data.tact](../contracts/pixelswap_streampool_data.tact)*

### Struct [PoolConfig](../contracts/pixelswap_streampool_data.tact#L50)

Config settings for the entire pool

**Fields**  

* [enable_token_creation](../contracts/pixelswap_streampool_data.tact#L52): Bool  
Whether anyone can create tokens for the pool.
* [enable_pair_creation](../contracts/pixelswap_streampool_data.tact#L54): Bool  
Whether anyone can create pairs for the pool.
* [enable_atomic_routing](../contracts/pixelswap_streampool_data.tact#L56): Bool  
Whether atomic routing is enabled for the pool.
* [enable_trading](../contracts/pixelswap_streampool_data.tact#L58): Bool  
Whether trading is enabled for the pool.
* [enable_adding_liquidity](../contracts/pixelswap_streampool_data.tact#L60): Bool  
Whether liquidity adding is enabled for the pool.
* [token_creation_fee](../contracts/pixelswap_streampool_data.tact#L62): Int as uint32  
Token creation fee in mTON.
* [pair_creation_fee](../contracts/pixelswap_streampool_data.tact#L64): Int as uint32  
Pair creation fee in mTON.
* [fee_collector_address](../contracts/pixelswap_streampool_data.tact#L66): Address  
Fee collector address for token and pair creation fees.
* [default_protocol_fee_bps](../contracts/pixelswap_streampool_data.tact#L68): Int as uint16  
Default protocol fee as a percentage of the trading fee, in basis points.

### Struct [TokenConfig](../contracts/pixelswap_streampool_data.tact#L72)

Settings for a single token

**Fields**  

* [is_active](../contracts/pixelswap_streampool_data.tact#L76): Bool  
Is the token validated? Can only be set by admin. If false, then the jetton wallet address can't be trusted.
Is the token active? Can only be set by admin. If false, then no new pools involving this token can be created. Default is true.

### Struct [PairConfig](../contracts/pixelswap_streampool_data.tact#L90)

A pair is uniquely identified by `(token0_id, token1_id, fee_bps, weight0_t0)`.  
Note that for LBP, multiple schedules can be created as long as the initial `weight0_t0` is different.  
`token0` is always TON.

**Fields**  

* [is_active](../contracts/pixelswap_streampool_data.tact#L92): Bool  
Is the pair active? Can only be set by admin. If false, then the only operation allowed is to remove liquidity. Default is false.
* [is_locked](../contracts/pixelswap_streampool_data.tact#L94): Bool  
Is the pair locked? Can only be set by admin. If true, then no operation is allowed. Default is false.
* [enable_adding_liquidity](../contracts/pixelswap_streampool_data.tact#L96): Bool  
Whether anyone can add liquidity to the pair. If set to false, then only the pair creator and admin can add liquidity. Default is true.
* [enable_onchain_oracle](../contracts/pixelswap_streampool_data.tact#L98): Bool  
Whether onchain oracle is enabled for the pair. Default is false.
* [token0_id](../contracts/pixelswap_streampool_data.tact#L100): Address  
TokenId of token1.
* [token1_id](../contracts/pixelswap_streampool_data.tact#L102): Address  
TokenId of token0.
* [fee_bps](../contracts/pixelswap_streampool_data.tact#L104): Int as uint16  
Fee percentage for the pair. 0.1% = 10, 0.3% = 30, 1% = 100. Not limited in contract but only those 3 values are supported by the frontend.
* [weight0](../contracts/pixelswap_streampool_data.tact#L106): Int as uint16  
Weight of token0 in the pair, in percentage [1,65535]. 65536 = 100%.
* [oracle_address](../contracts/pixelswap_streampool_data.tact#L108): Address  
Onchain oracle address.
* [protocol_fee_bps](../contracts/pixelswap_streampool_data.tact#L112): Int as uint16  
Pair creator address.
Protocol fee for the pair, in basis points.

### Struct [PairStatus](../contracts/pixelswap_streampool_data.tact#L115)

**Fields**  

* [reserve0](../contracts/pixelswap_streampool_data.tact#L116): Int as coins  
* [reserve1](../contracts/pixelswap_streampool_data.tact#L117): Int as coins  
* [lp_supply](../contracts/pixelswap_streampool_data.tact#L118): Int as coins  
* [token0_fees_payable](../contracts/pixelswap_streampool_data.tact#L119): Int as coins  
* [token1_fees_payable](../contracts/pixelswap_streampool_data.tact#L120): Int as coins  

### Struct [SwapResult](../contracts/pixelswap_streampool_data.tact#L123)

**Fields**  

* [amount_in](../contracts/pixelswap_streampool_data.tact#L125): Int as coins  
The amount in, not including feesi
* [amount_out](../contracts/pixelswap_streampool_data.tact#L127): Int as coins  
The amount out.
* [fee](../contracts/pixelswap_streampool_data.tact#L129): Int as coins  
The fee, charged on the input token.

### Struct [LiquidityResult](../contracts/pixelswap_streampool_data.tact#L134)

Result of adding/removing liquidity.  
All tokens have the same sign (increase pool amounts for adding liquidity and decrease pool amounts for removing liquidity).

**Fields**  

* [amount0](../contracts/pixelswap_streampool_data.tact#L136): Int as coins  
Delta of token0.
* [amount1](../contracts/pixelswap_streampool_data.tact#L138): Int as coins  
Delta of token1.
* [liquidity](../contracts/pixelswap_streampool_data.tact#L140): Int as coins  
Delta of LP token balance.
* [fee0](../contracts/pixelswap_streampool_data.tact#L142): Int as coins  
Fee charged on top of amount 0.
* [fee1](../contracts/pixelswap_streampool_data.tact#L144): Int as coins  
Fee charged on top of amount 1.

### Struct [ProcessOrderResult](../contracts/pixelswap_streampool_data.tact#L148)

Result of process_order.  

**Fields**  

* [rsvd](../contracts/pixelswap_streampool_data.tact#L150): Cell?  
Empty cell.
* [exec_res](../contracts/pixelswap_streampool_data.tact#L152): Cell?  
The OrderExecutionResult cell.

## *Documentation for [contracts/pixelswap_streampool_messages.tact](../contracts/pixelswap_streampool_messages.tact)*

### Message [CreateToken](../contracts/pixelswap_streampool_messages.tact#L5)

**Fields**  

* [token_id](../contracts/pixelswap_streampool_messages.tact#L6): Address  

### Message [CreateTradingPair](../contracts/pixelswap_streampool_messages.tact#L9)

**Fields**  

* [token0_id](../contracts/pixelswap_streampool_messages.tact#L10): Address  
* [token1_id](../contracts/pixelswap_streampool_messages.tact#L11): Address  
* [enable_adding_liquidity](../contracts/pixelswap_streampool_messages.tact#L12): Bool  
* [fee_bps](../contracts/pixelswap_streampool_messages.tact#L13): Int as uint16  
* [weight0](../contracts/pixelswap_streampool_messages.tact#L14): Int as uint16  

### Message [RemoveLiquidityJettonNotification](../contracts/pixelswap_streampool_messages.tact#L33)

Remove liquidity via Jetton Transfer Notification.  
`query_id`: 32-MSB: subaccount of the recipient 32-LSB: fund_id of the target jetton.  
`amount`: amount of jetton transferred.  
`sender`: sender of the jetton.  
`forward_payload`: payload to forward to the Execution contract.  
Data for `forward_payload`:  
**Remove Liquidity**:  
- expiry (u32): Unix epoch div 4  
- pair_id (u32): fund_id to route funds back to.  
- ref (ref Cell):  
    - amount0_min_out: u128;  
    - amount1_min_out: u128;  
    - token0_gas: Int? as uint32;  
    - token0_forward_milliton: Int? as uint32;  
    - token1_gas: Int? as uint32;  
    - token1_forward_milliton: Int? as uint32;  

**Fields**  

* [fund_id](../contracts/pixelswap_streampool_messages.tact#L34): Int as uint32  
* [subaccount](../contracts/pixelswap_streampool_messages.tact#L35): Int as uint32  
* [amount](../contracts/pixelswap_streampool_messages.tact#L36): Int as coins  
* [user](../contracts/pixelswap_streampool_messages.tact#L37): Address  
* [forward_payload](../contracts/pixelswap_streampool_messages.tact#L38): Slice as remaining  

### Message [TokenCreatedEvent](../contracts/pixelswap_streampool_messages.tact#L91)

**Fields**  

* [token_id](../contracts/pixelswap_streampool_messages.tact#L92): Address  

### Message [PairCreatedEvent](../contracts/pixelswap_streampool_messages.tact#L95)

**Fields**  

* [pair_id](../contracts/pixelswap_streampool_messages.tact#L96): Int as uint32  
* [token0_id](../contracts/pixelswap_streampool_messages.tact#L97): Address  
* [token1_id](../contracts/pixelswap_streampool_messages.tact#L98): Address  
* [fee_bps](../contracts/pixelswap_streampool_messages.tact#L99): Int as uint16  
* [weight0](../contracts/pixelswap_streampool_messages.tact#L100): Int as uint16  

### Message [SwapEvent](../contracts/pixelswap_streampool_messages.tact#L103)

**Fields**  

* [user](../contracts/pixelswap_streampool_messages.tact#L104): Address  
* [subaccount](../contracts/pixelswap_streampool_messages.tact#L105): Int as uint32  
* [pair_id](../contracts/pixelswap_streampool_messages.tact#L106): Int as uint32  
* [side](../contracts/pixelswap_streampool_messages.tact#L107): Bool  
* [amount_in](../contracts/pixelswap_streampool_messages.tact#L108): Int as coins  
* [amount_out](../contracts/pixelswap_streampool_messages.tact#L109): Int as coins  
* [fee](../contracts/pixelswap_streampool_messages.tact#L110): Int as coins  

### Message [AddLiquidityEvent](../contracts/pixelswap_streampool_messages.tact#L114)

Fees are only charged when adding unbalanced liquidity.

**Fields**  

* [user](../contracts/pixelswap_streampool_messages.tact#L115): Address  
* [subaccount](../contracts/pixelswap_streampool_messages.tact#L116): Int as uint32  
* [pair_id](../contracts/pixelswap_streampool_messages.tact#L117): Int as uint32  
* [amount0](../contracts/pixelswap_streampool_messages.tact#L118): Int as coins  
* [amount1](../contracts/pixelswap_streampool_messages.tact#L119): Int as coins  
* [lp_amount](../contracts/pixelswap_streampool_messages.tact#L120): Int as coins  
* [fee0](../contracts/pixelswap_streampool_messages.tact#L121): Int as coins  
* [fee1](../contracts/pixelswap_streampool_messages.tact#L122): Int as coins  

### Message [RemoveLiquidityEvent](../contracts/pixelswap_streampool_messages.tact#L125)

**Fields**  

* [user](../contracts/pixelswap_streampool_messages.tact#L126): Address  
* [subaccount](../contracts/pixelswap_streampool_messages.tact#L127): Int as uint32  
* [pair_id](../contracts/pixelswap_streampool_messages.tact#L128): Int as uint32  
* [amount0](../contracts/pixelswap_streampool_messages.tact#L129): Int as coins  
* [amount1](../contracts/pixelswap_streampool_messages.tact#L130): Int as coins  
* [lp_amount](../contracts/pixelswap_streampool_messages.tact#L131): Int as coins  

*Documentation generated by [Tactdoc](https://github.com/microcosm-labs/tact-tools/tree/main/tactdoc) v0.1.3.*
