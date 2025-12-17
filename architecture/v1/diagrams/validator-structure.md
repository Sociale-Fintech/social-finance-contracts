# v1.0 Validator Structure

## ROSCA Validator Components

```
┌──────────────────────────────────────────────────┐
│                                                  │
│              ROSCA VALIDATOR                     │
│                                                  │
│  ┌────────────────────────────────────────────┐ │
│  │           STATE (Datum)                    │ │
│  │                                            │ │
│  │  rosca_id: ByteArray                       │ │
│  │  admin: PubKeyHash                         │ │
│  │  members: List<PubKeyHash> (10)            │ │
│  │  current_round: Int (0-9)                  │ │
│  │  total_rounds: Int (10)                    │ │
│  │  amount_per_round: Int                     │ │
│  │  balance: Int                              │ │
│  │  status: Active | Paused | Complete        │ │
│  │                                            │ │
│  └────────────────────────────────────────────┘ │
│                                                  │
│  ┌────────────────────────────────────────────┐ │
│  │         ACTIONS (Redeemers)                │ │
│  │                                            │ │
│  │  Deposit           → Add funds             │ │
│  │  ExecutePayout     → Pay current recipient │ │
│  │  AdvanceRound      → Move to next round    │ │
│  │  Pause             → Emergency stop        │ │
│  │  Complete          → Close after all rounds│ │
│  │                                            │ │
│  └────────────────────────────────────────────┘ │
│                                                  │
│  ┌────────────────────────────────────────────┐ │
│  │         VALIDATION LOGIC                   │ │
│  │                                            │ │
│  │  • Check signatures                        │ │
│  │  • Verify balances                         │ │
│  │  • Validate state transitions              │ │
│  │  • Enforce round order                     │ │
│  │  • Ensure correct recipients               │ │
│  │                                            │ │
│  └────────────────────────────────────────────┘ │
│                                                  │
└──────────────────────────────────────────────────┘
```

## State Transitions

```
┌─────────┐
│ Create  │  Initial state: round 0, balance 0
└────┬────┘
     │
     ▼
┌─────────┐
│ Active  │ ◄──┐
└────┬────┘    │
     │         │
     ├────────────► Deposit       → balance += amount
     │         │
     ├────────────► ExecutePayout → balance -= amount, pay recipient
     │         │
     ├────────────► AdvanceRound  → current_round += 1
     │         │
     │         └─── (back to Active)
     │
     ├────────────► Pause         → status = Paused
     │
     ▼
┌─────────┐
│Complete │  Final state: all rounds done
└─────────┘
```

## Authorization Model

```
┌────────────┐
│   Admin    │ (Single PubKeyHash)
└─────┬──────┘
      │
      │ Can execute:
      ├─── ExecutePayout
      ├─── AdvanceRound
      ├─── Pause
      └─── Complete

┌────────────┐
│  Members   │ (10 PubKeyHashes)
└─────┬──────┘
      │
      │ Can execute:
      └─── Deposit (anyone can)
```

