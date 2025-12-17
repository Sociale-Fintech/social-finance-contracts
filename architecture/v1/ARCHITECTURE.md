# v1.0 Architecture - Single ROSCA Validator

**Version:** 1.0  
**Date:** December 17, 2024

---

## Goal

Build the **simplest possible ROSCA on-chain** to prove the concept works.

---

## What is v1.0?

A single validator that handles:
- Holding funds (treasury)
- Tracking rotation order
- Executing payouts
- Managing rounds

**One validator. One admin. Ten members. That's it.**

---

## Core Concept

A ROSCA (Rotating Savings and Credit Association) is:
1. Group of 10 people
2. Each contributes $100/month
3. One person receives $1,000 that month
4. Rotate through all members
5. After 10 months, everyone got their turn

---

## Architecture

### System Diagram

```
┌─────────────────────────────────────────────┐
│                                             │
│           ROSCA VALIDATOR                   │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │  DATUM (State)                      │   │
│  │  • rosca_id                         │   │
│  │  • admin (single PubKeyHash)        │   │
│  │  • members (10 PubKeyHashes)        │   │
│  │  • current_round (0-9)              │   │
│  │  • amount_per_round (100 ADA)       │   │
│  │  • balance                          │   │
│  │  • status (Active/Paused/Complete)  │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │  REDEEMERS (Actions)                │   │
│  │  • Deposit                          │   │
│  │  • ExecutePayout                    │   │
│  │  • AdvanceRound                     │   │
│  │  • Pause                            │   │
│  │  • Complete                         │   │
│  └─────────────────────────────────────┘   │
│                                             │
└─────────────────────────────────────────────┘
         ▲                          │
         │ deposit                  │ payout
         │                          ▼
    ┌─────────┐              ┌──────────┐
    │ Members │              │ Recipient│
    └─────────┘              └──────────┘
```

### Data Flow

```
Round 1: Member 1-10 contribute → Balance = 1000 ADA → Payout to Member 1
Round 2: Member 1-10 contribute → Balance = 1000 ADA → Payout to Member 2
...
Round 10: Member 1-10 contribute → Balance = 1000 ADA → Payout to Member 10
→ Complete
```

---

## Technical Specification

### Datum

```aiken
type ROSCADatum {
  rosca_id: ByteArray,              // Unique identifier
  admin: PubKeyHash,                // Single admin (no multi-sig in v1)
  members: List<PubKeyHash>,        // 10 members (fixed list)
  current_round: Int,               // 0 to 9
  total_rounds: Int,                // Always 10 for v1
  amount_per_round: Int,            // Expected payout per round (e.g., 100 ADA)
  balance: Int,                     // Current balance in validator
  status: Status,                   // Active, Paused, or Complete
}

type Status {
  Active      // Normal operations
  Paused      // Emergency stop
  Complete    // All rounds finished
}
```

### Redeemers

```aiken
type ROSCARedeemer {
  // Anyone can deposit
  Deposit {
    amount: Int,
  }
  
  // Admin executes payout to current recipient
  ExecutePayout {
    recipient_index: Int,           // 0-9 (which member receives)
  }
  
  // Admin advances to next round
  AdvanceRound
  
  // Admin pauses (emergency)
  Pause
  
  // Admin completes after all rounds
  Complete
}
```

---

## Validation Rules

### Deposit

**Anyone can deposit funds**

Checks:
- ✓ Status is Active
- ✓ Amount > 0
- ✓ Balance increases correctly

Updates:
- Balance += amount

### ExecutePayout

**Admin pays current round recipient**

Checks:
- ✓ Signed by admin
- ✓ Status is Active
- ✓ recipient_index == current_round
- ✓ Recipient is members[recipient_index]
- ✓ Balance >= amount_per_round

Updates:
- Balance -= amount_per_round
- Transfer amount_per_round to recipient

### AdvanceRound

**Admin moves to next round**

Checks:
- ✓ Signed by admin
- ✓ Status is Active
- ✓ Previous payout completed
- ✓ current_round < total_rounds - 1

Updates:
- current_round += 1

### Pause

**Admin emergency stop**

Checks:
- ✓ Signed by admin
- ✓ Status is Active

Updates:
- status = Paused

### Complete

**Admin closes after all rounds**

Checks:
- ✓ Signed by admin
- ✓ current_round == total_rounds - 1
- ✓ Final payout executed

Updates:
- status = Complete

---

## Example Flow

### Setup

```aiken
Initial ROSCA Datum:
{
  rosca_id: "rosca_001",
  admin: <admin_pkh>,
  members: [<pkh1>, <pkh2>, ..., <pkh10>],
  current_round: 0,
  total_rounds: 10,
  amount_per_round: 100_000_000,  // 100 ADA
  balance: 0,
  status: Active
}
```

### Round 1

1. **Members deposit**: 10 x 100 ADA = 1000 ADA
   - Balance: 0 → 1000 ADA

2. **Admin executes payout**: 
   - ExecutePayout { recipient_index: 0 }
   - Transfer 100 ADA to members[0]
   - Balance: 1000 → 900 ADA (keeping extra as buffer)

3. **Admin advances round**:
   - AdvanceRound
   - current_round: 0 → 1

### Round 2-9

Same pattern...

### Round 10 (Final)

1. Members deposit
2. Admin executes payout to members[9]
3. Admin calls Complete
4. status: Active → Complete

---

## Limitations

These are **intentional** for v1.0 simplicity:

### What v1.0 Doesn't Have

❌ **Multi-signature** - Single admin has full control  
❌ **Governance** - No voting, admin decides everything  
❌ **Dynamic members** - List is fixed at creation  
❌ **Contribution tracking** - Relies on off-chain records  
❌ **Late fees** - No penalties for late contributions  
❌ **Variable amounts** - Fixed amount per round  
❌ **Partial payouts** - All-or-nothing per round  

### Workarounds

| Limitation | v1.0 Workaround |
|------------|----------------|
| No contribution tracking | Off-chain database tracks who paid |
| No member changes | Create new ROSCA if membership changes |
| Single admin | Trust-based; choose admin carefully |
| No multi-sig | Admin can be a multi-sig wallet off-chain |

---

## What v1.0 Proves

### Technical Validation

✅ **Cardano UTXO model** works for ROSCAs  
✅ **State transitions** function correctly  
✅ **Fund security** - Validator holds real money safely  
✅ **Gas costs** are acceptable (< $0.20 per tx)  
✅ **Wallet integration** works with Nami, Eternl, etc.  

### Product Validation

✅ **Users** can interact with validators  
✅ **Real money** flows correctly  
✅ **Rotation logic** is sound  
✅ **On-chain transparency** is valuable  
✅ **Concept** has product-market fit  

---

## Security Considerations

### Trust Model

v1.0 requires trusting:
1. **The admin** - Has full control
2. **The validator code** - Must be bug-free
3. **Cardano network** - Must stay secure

### Attack Vectors

**Admin abuse**
- Risk: Admin steals funds
- Mitigation: Social trust, small amounts for v1

**Smart contract bugs**
- Risk: Logic errors allow fund loss
- Mitigation: Thorough testing, security review

**Validator key loss**
- Risk: Admin loses private key
- Mitigation: Backup procedures

### Security Measures

✅ Extensive testing (100+ test cases)  
✅ Testnet deployment first (2+ weeks)  
✅ Start with small amounts ($100-1000)  
✅ Code review by Aiken experts  
✅ Emergency pause function  

---

## Testing Strategy

### Unit Tests

Test each redeemer:
- ✓ Deposit updates balance
- ✓ ExecutePayout validates recipient
- ✓ AdvanceRound increments counter
- ✓ Pause changes status
- ✓ Complete requires all rounds done

### Integration Tests

Test full ROSCA lifecycle:
- ✓ Create ROSCA with 10 members
- ✓ Execute 10 complete rounds
- ✓ All payouts succeed
- ✓ Complete after round 10

### Edge Cases

- ✓ Insufficient balance for payout
- ✓ Wrong recipient index
- ✓ Advance round before payout
- ✓ Complete before all rounds
- ✓ Actions when paused

---

## Deployment

### Testnet

1. Deploy validator to Cardano Preprod
2. Create test ROSCA with 10 test wallets
3. Run through 10 complete rounds
4. Monitor for issues
5. Test for 2+ weeks

### Mainnet

**Requirements before mainnet:**
- ✅ 50+ successful test rounds
- ✅ Zero critical bugs found
- ✅ Security review completed
- ✅ Gas costs confirmed acceptable
- ✅ Documentation complete

**Initial Deployment:**
- Start with 1-3 pilot entities
- Small amounts ($100-500 total)
- Close monitoring
- Quick response plan

---

## Next Steps

### After v1.0 is Live

**Learn from production:**
- What works well?
- What's painful?
- What features are most requested?
- What security issues arise?

**Plan v1.1:**
- Extract treasury pattern
- Split into 2 validators
- Maintain all v1.0 functionality
- Zero downtime migration

---

## Success Criteria

v1.0 is successful when:

✅ **Usage**: 10+ entities using it  
✅ **Volume**: $10,000+ under management  
✅ **Reliability**: 100+ successful payouts  
✅ **Security**: Zero critical incidents  
✅ **Performance**: Gas < $0.20 per tx  
✅ **Satisfaction**: Users want to continue using it  

---

## File Structure

```
validators/v1/
└── rosca.ak              # The validator implementation

tests/v1/
├── unit/
│   ├── deposit.ak
│   ├── payout.ak
│   └── rounds.ak
└── integration/
    └── full_cycle.ak

architecture/v1/
├── ARCHITECTURE.md       # This file
└── diagrams/
    ├── validator-structure.md  # Validator structure
    ├── transaction-flow.md     # Transaction flow
    └── lifecycle.md            # ROSCA lifecycle
```

---

## Summary

v1.0 is:
- **Simple** - One validator, one admin, ten members
- **Focused** - Prove ROSCA works on-chain
- **Complete** - Fully functional for basic use case
- **Foundation** - Basis for future enhancements

**Not trying to solve everything**, just prove the core concept works.

Once validated, we extract patterns and scale up in v1.1+.

---

**Questions?** Open an issue or check the [main docs](../DOCS.md).

---

**Note**: This is design documentation in `architecture/`. Aiken's auto-generated docs will be in `docs/`.

