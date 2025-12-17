# v1.0 Transaction Flow

## Deposit Flow

```
Member Wallet
    │
    │ 1. Build tx with funds
    ▼
┌─────────────────┐
│ Deposit Action  │
│ amount: 100 ADA │
└────────┬────────┘
         │
         │ 2. Submit to validator
         ▼
┌───────────────────────┐
│  ROSCA Validator      │
│  Validates:           │
│  • Status is Active   │
│  • Amount > 0         │
└────────┬──────────────┘
         │
         │ 3. Accept & update state
         ▼
┌───────────────────────┐
│  New Datum:           │
│  balance: old + 100   │
└───────────────────────┘
```

## Payout Flow

```
Admin Wallet
    │
    │ 1. Build ExecutePayout tx
    ▼
┌────────────────────────┐
│ ExecutePayout Action   │
│ recipient_index: 0     │
└──────────┬─────────────┘
           │
           │ 2. Submit to validator
           ▼
┌────────────────────────────┐
│  ROSCA Validator           │
│  Validates:                │
│  • Admin signature         │
│  • Status is Active        │
│  • Recipient index correct │
│  • Sufficient balance      │
└──────────┬─────────────────┘
           │
           │ 3. Execute payout
           ▼
┌────────────────────────────┐
│  Transfer 100 ADA          │
│  To: members[0]            │
└──────────┬─────────────────┘
           │
           │ 4. Update state
           ▼
┌────────────────────────────┐
│  New Datum:                │
│  balance: old - 100        │
└────────────────────────────┘
           │
           ▼
    Recipient Wallet
    (receives 100 ADA)
```

## Advance Round Flow

```
Admin Wallet
    │
    │ 1. Build AdvanceRound tx
    ▼
┌────────────────────────┐
│ AdvanceRound Action    │
└──────────┬─────────────┘
           │
           │ 2. Submit to validator
           ▼
┌────────────────────────────┐
│  ROSCA Validator           │
│  Validates:                │
│  • Admin signature         │
│  • Status is Active        │
│  • Payout was executed     │
│  • Not at final round      │
└──────────┬─────────────────┘
           │
           │ 3. Increment round
           ▼
┌────────────────────────────┐
│  New Datum:                │
│  current_round: old + 1    │
└────────────────────────────┘
```

## Complete Cycle Flow

```
Round 1: Deposits → ExecutePayout(0) → AdvanceRound → Round 2
Round 2: Deposits → ExecutePayout(1) → AdvanceRound → Round 3
...
Round 10: Deposits → ExecutePayout(9) → Complete → DONE
```

