# v1.0 ROSCA Lifecycle

## Full Lifecycle

```
START
  │
  │ 1. Create ROSCA
  ▼
┌─────────────────────────┐
│  Initial State          │
│  • 10 members defined   │
│  • Admin assigned       │
│  • Amount set           │
│  • Round 0              │
│  • Balance 0            │
│  • Status: Active       │
└───────────┬─────────────┘
            │
            │ 2. Members deposit
            ▼
┌─────────────────────────┐
│  Round 1                │
│  • Balance: 1000 ADA    │
│  • Ready for payout     │
└───────────┬─────────────┘
            │
            │ 3. Admin pays member[0]
            ▼
┌─────────────────────────┐
│  After Payout           │
│  • Balance: 900 ADA     │
│  • Member[0] received   │
└───────────┬─────────────┘
            │
            │ 4. Admin advances round
            ▼
┌─────────────────────────┐
│  Round 2                │
│  • current_round: 1     │
└───────────┬─────────────┘
            │
            │ 5. Repeat steps 2-4
            │    for rounds 2-9
            ▼
            ...
            │
            │ 10. Final round
            ▼
┌─────────────────────────┐
│  Round 10               │
│  • Last deposits        │
│  • Last payout          │
└───────────┬─────────────┘
            │
            │ 11. Admin completes
            ▼
┌─────────────────────────┐
│  Complete               │
│  • Status: Complete     │
│  • All members paid     │
│  • ROSCA finished       │
└─────────────────────────┘
  │
  ▼
END
```

## State Diagram

```
        ┌────────┐
        │ Create │
        └───┬────┘
            │
            ▼
        ┌────────┐
    ┌──▶│ Active │◄──┐
    │   └───┬────┘   │
    │       │        │
    │       │ Deposit/Payout/AdvanceRound
    │       │        │
    │       └────────┘
    │
    │ Pause
    ▼
┌────────┐    Resume
│ Paused │────────────┐
└────────┘            │
                      ▼
    ┌──────────────────┐
    │                  │
    │   Complete       │
    │   (terminal)     │
    │                  │
    └──────────────────┘
```

## Timeline Example

```
Week 1:   Create ROSCA
            └─► Deposits from all 10 members
            └─► Payout to Member 1
            └─► Advance to Round 2

Week 2:   Deposits from all 10 members
            └─► Payout to Member 2
            └─► Advance to Round 3

Week 3-9: ... (same pattern)

Week 10:  Deposits from all 10 members
            └─► Payout to Member 10
            └─► Complete ROSCA

Status:   ✓ All members contributed 10 times
          ✓ All members received 1 payout
          ✓ ROSCA successfully completed
```

## Error Handling

```
┌───────────────────────┐
│  Normal Flow          │
│  (Happy Path)         │
└──────────┬────────────┘
           │
           │ Something goes wrong
           ▼
┌───────────────────────┐
│  Admin calls Pause    │
│  status → Paused      │
└──────────┬────────────┘
           │
           │ Issue resolved
           ▼
┌───────────────────────┐
│  Admin calls Resume   │
│  status → Active      │
└──────────┬────────────┘
           │
           │ Continue
           ▼
┌───────────────────────┐
│  Normal Flow          │
│  (Happy Path)         │
└───────────────────────┘
```

## Member Participation

```
Member Perspective (Member 1):

Week 1:  Deposit 100 ADA → Receive 1000 ADA ✓
Week 2:  Deposit 100 ADA
Week 3:  Deposit 100 ADA
Week 4:  Deposit 100 ADA
Week 5:  Deposit 100 ADA
Week 6:  Deposit 100 ADA
Week 7:  Deposit 100 ADA
Week 8:  Deposit 100 ADA
Week 9:  Deposit 100 ADA
Week 10: Deposit 100 ADA

Total: Contributed 1000 ADA, Received 1000 ADA
Net: 0 (but got large sum early!)
```

