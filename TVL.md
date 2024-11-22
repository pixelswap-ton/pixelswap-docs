# Transaction Volume Calculation

*We do have funding layer & settlement layer, which means there do exist user funding wallet with AA feature, so the calculation may be a little bit different:

Total Volume = Deposit Event + Withdraw Event + Swap Events

## Deposit event

sample txid: 78482c51a4d4664ae0e3d0f9dbef3ffe18d840cf94f0a7e201eda55f07d44f2d
Deposit[Deposit]
TLB: deposit#307bf181 fund_id:Maybe uint32 subaccount:uint32 routing_code:remainder<slice> = Deposit

## Withdraw event

sample txid: 1ab8431ee9adbdd3dcbfd7fa37725d6c08d4556f8bd48c43f508c96b27fe5fd8

WithdrawFunds[Withdraw (withdraw funds from the internal funding wallet to the external EOA wallet)]
    TLB: withdraw_funds#e189c37f fund_id:uint32 user:address subaccount:uint32 token_id:address token_amt:coins gas_transfer:uint32 forward_milliton:uint32 = WithdrawFunds

## Swap Event (from outsource wallet to funding wallet)

txid: bce2a2d5257b7ca3c46877f6d13d0c364d5f68518ab18491c3aa946e8af12df5
  MSGTOOLS__JettonTransferNotification [Swap]
  TLB: msgtools_jetton_transfer_notification#7362d09c query_id:uint64 amount:coins sender:address forward_payload:remainder<slice> = MSGTOOLS__JettonTransferNotification

## Swap Event (from funding wallet to other outsource wallets)

txid: e58a24c91bb1c9a90e6e5842e2ac0cd7fa39f3f608b57fc2383b8894299503ff

  PlaceOrder [Swap]
  TLB: place_order#d10e49a1 exec_id:uint32 fund_id:uint32 user:address subaccount:uint32 mode:uint8 expiry:uint32 token_is_input:bool token_id:address token_amt:coins out_amount:coins extra_tokens:Maybe ^cell ref_po:Maybe ^cell routing_code:remainder<slice> = PlaceOrder

## Swap Event (from outsource wallet to outsource wallet )

sample txid:
34128d431c7c434d8cf1e16cbe33ab3de3a4c3e5bdbe2721fd90004b3b08ecce

TLB: msgtools_jetton_transfer_notification#7362d09c query_id:uint64 amount:coins sender:address forward_payload:remainder<slice> = MSGTOOLS__JettonTransferNotification

## Swap Event (from funding wallet to other funding wallets)

 txid:
7fb62679ec8dbb3b6a3933e23076a3e346f693d7ed5574e6abedfe9bf2d4f9a0

PlaceOrder [Swap]
  TLB: place_order#d10e49a1 exec_id:uint32 fund_id:uint32 user:address subaccount:uint32 mode:uint8 expiry:uint32 token_is_input:bool token_id:address token_amt:coins out_amount:coins extra_tokens:Maybe ^cell ref_po:Maybe ^cell routing_code:remainder<slice> = PlaceOrder

TVL Calculation
Total TVL = Deposit Event + Add Liquidity Event - Withdraw Event - Remove Liquidity Event + Swap Event (From the external EOA wallet to the internal funding wallet) - Swap Event (From the internal funding wallet to the external EOA wallet)

## deposit event

sample txid: 78482c51a4d4664ae0e3d0f9dbef3ffe18d840cf94f0a7e201eda55f07d44f2d
  Deposit[Deposit]
  TLB: deposit#307bf181 fund_id:Maybe uint32 subaccount:uint32 routing_code:remainder<slice> = Deposit

## add-Liquidity event

 sample txid: af492407deb004b15915683befe4ee6ce4d4394edda8599b5b49f1d859235e88
  MSGTOOLS__JettonTransferNotification [Add Liquidity]
  
TLB: msgtools_jetton_transfer_notification#7362d09c query_id:uint64 amount:coins sender:address forward_payload:remainder<slice> = MSGTOOLS__JettonTransferNotification

## withdraw event

sample txid: 1ab8431ee9adbdd3dcbfd7fa37725d6c08d4556f8bd48c43f508c96b27fe5fd8
  WithdrawFunds[Withdraw (withdraw funds from the internal funding wallet to the external EOA wallet)]
  TLB: withdraw_funds#e189c37f fund_id:uint32 user:address subaccount:uint32 token_id:address token_amt:coins gas_transfer:uint32 forward_milliton:uint32 = WithdrawFunds

## removeLiquidity event

txid: 9fe4297abc0c750d11ebf4e168413b24a5eab62a3406e004c8013ebbef46d176
  RemoveLiquidityJettonNotification[Remove Liquidity]
  TLB: remove_liquidity_jetton_notification#7362d09c fund_id:uint32 subaccount:uint32 amount:coins user:address forward_payload:remainder<slice> = RemoveLiquidityJettonNotification

## Swap Event (From the external EOA wallet to the internal funding wallet)

 txid: bce2a2d5257b7ca3c46877f6d13d0c364d5f68518ab18491c3aa946e8af12df5
  MSGTOOLS__JettonTransferNotification [Swap]
  TLB: msgtools_jetton_transfer_notification#7362d09c query_id:uint64 amount:coins sender:address forward_payload:remainder<slice> = MSGTOOLS__JettonTransferNotification

## Swap Event (From the internal funding wallet to the external EOA wallet)

 txid: e58a24c91bb1c9a90e6e5842e2ac0cd7fa39f3f608b57fc2383b8894299503ff
  PlaceOrder [Swap]
  TLB: place_order#d10e49a1 exec_id:uint32 fund_id:uint32 user:address subaccount:uint32 mode:uint8 expiry:uint32 token_is_input:bool token_id:address token_amt:coins out_amount:coins extra_tokens:Maybe ^cell ref_po:Maybe ^cell routing_code:remainder<slice> = PlaceOrder